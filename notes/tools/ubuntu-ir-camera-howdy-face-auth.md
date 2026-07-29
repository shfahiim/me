# Using a Windows Hello IR Camera on Ubuntu with Howdy

## What it is

Ubuntu can use some laptop IR cameras for face authentication through
[Howdy](https://github.com/boltgolt/howdy), a PAM-based face authentication tool.
It is not Windows Hello, but it gives similar behavior for Linux authentication
prompts such as `sudo`.

In this case, the laptop already exposed the IR camera to Linux. The problem was
not missing camera hardware support. The actual blockers were:

- Howdy was not pointed at the IR video device.
- Howdy's config file was duplicated and could not be parsed.
- Howdy's PAM module expected Python 2's `ConfigParser`, but Ubuntu 24.04 uses
  Python 3.12 for `pam_python`.

## Why it matters

The same IR sensor worked in Windows Hello, but Ubuntu did not use it out of the
box. This is easy to misdiagnose as an unsupported Linux driver problem.

The important lesson: first confirm whether Linux sees the IR stream. If it
does, the rest is usually configuration and PAM integration.

## System context

The working system was:

```txt
Ubuntu 24.04.2 LTS
Kernel: 6.8.0-52-generic
Camera USB ID: 13d3:56eb IMC Networks USB2.0 HD UVC WebCam
```

Linux exposed four video nodes:

```txt
/dev/video0  normal webcam video
/dev/video1  normal webcam metadata
/dev/video2  IR camera video
/dev/video3  IR camera metadata
```

The IR camera showed up as:

```txt
USB2.0 HD UVC WebCam: USB2.0 IR
Pixel format: GREY
Resolution: 640x360
Frame rate: 15/30 fps
```

## Key idea

Do not guess the camera path. Find the actual IR device and configure Howdy to
use that path.

For this machine, the correct path was:

```ini
device_path = /dev/video2
```

## Diagnosis

List USB devices:

```bash
lsusb
```

The camera appeared as:

```txt
Bus 003 Device 002: ID 13d3:56eb IMC Networks USB2.0 HD UVC WebCam
```

List video devices:

```bash
v4l2-ctl --list-devices
```

Inspect each video node:

```bash
for d in /dev/video*; do
  echo "### $d"
  v4l2-ctl --device="$d" --all | sed -n "1,80p"
  v4l2-ctl --device="$d" --list-formats-ext | sed -n "1,120p"
done
```

The useful result was that `/dev/video2` had:

```txt
Card type: USB2.0 HD UVC WebCam: USB2.0 IR
Pixel Format: 'GREY' (8-bit Greyscale)
Width/Height: 640/360
```

Confirm that the IR stream can produce frames:

```bash
v4l2-ctl \
  --device=/dev/video2 \
  --set-fmt-video=width=640,height=360,pixelformat=GREY \
  --stream-mmap \
  --stream-count=5 \
  --stream-to=/tmp/howdy-ir-test.raw
```

If this writes a raw file, the IR camera is producing frames.

## Install Howdy

Howdy was installed from the PPA:

```bash
sudo add-apt-repository ppa:boltgolt/howdy
sudo apt update
sudo apt install howdy
```

On this machine, `apt-cache policy howdy` showed:

```txt
Installed: 2.6.1
Candidate: 2.6.1
Source: ppa:boltgolt/howdy
```

The package files were installed under:

```txt
/lib/security/howdy
/usr/lib/security/howdy
```

But the `howdy` command was not available on `$PATH`, so a symlink was added:

```bash
sudo ln -sf /lib/security/howdy/cli.py /usr/local/bin/howdy
```

## Fix Howdy config

The config file was:

```txt
/lib/security/howdy/config.ini
```

It had duplicate sections such as repeated `[core]` and `[video]` blocks.
Python's strict `configparser` rejected it with:

```txt
DuplicateSectionError: While reading from '/lib/security/howdy/config.ini': section 'core' already exists
```

The fix was to back up the broken file and replace it with a clean config:

```bash
sudo cp -a /lib/security/howdy/config.ini \
  /lib/security/howdy/config.ini.bak.$(date +%Y%m%d-%H%M%S)
```

Minimal working config:

```ini
[core]
detection_notice = false
no_confirmation = false
suppress_unknown = false
ignore_ssh = true
ignore_closed_lid = true
abort_if_ssh = true
abort_if_lid_closed = true
disabled = false
use_cnn = false
workaround = off

[video]
certainty = 3.5
timeout = 4
device_path = /dev/video2
warn_no_device = true
max_height = 320
frame_width = -1
frame_height = -1
dark_threshold = 50
recording_plugin = opencv
device_format = v4l2
force_mjpeg = false
exposure = -1
device_fps = -1
rotate = 0

[snapshots]
capture_failed = false
capture_successful = false
save_failed = false
save_successful = false

[rubberstamps]
enabled = false
stamp_rules =

[debug]
end_report = false
verbose_stamps = false
gtk_stdout = false
```

Validate the config:

```bash
python3 - <<'PY'
import configparser

path = "/lib/security/howdy/config.ini"
config = configparser.ConfigParser()
config.read(path)
print("device_path =", config.get("video", "device_path"))
PY
```

Expected output:

```txt
device_path = /dev/video2
```

## Enroll a face model

After the config was fixed:

```bash
sudo howdy add
sudo howdy test
```

The enrollment succeeded:

```txt
Adding face model for the user fahim
Please look straight into the camera
Scan complete
Added a new model to fahim
```

The OpenCV/GStreamer warnings appeared but did not block enrollment:

```txt
OpenCV | GStreamer warning: Could not read from resource.
OpenCV | GStreamer warning: unable to start pipeline
```

In this case, those warnings were noisy but not fatal.

## Fix PAM on Ubuntu 24.04

After enrollment, `sudo` still asked for the password:

```bash
sudo -k
sudo -v
```

PAM was already wired correctly:

```txt
/etc/pam.d/sudo includes common-auth
/etc/pam.d/common-auth calls pam_python.so /lib/security/howdy/pam.py
```

The real error was in `/var/log/auth.log`:

```txt
/lib/security/howdy/pam.py: import ConfigParser
ModuleNotFoundError: No module named 'ConfigParser'
```

Cause:

```txt
/lib/security/pam_python.so is linked to libpython3.12.so.1.0
```

Old Howdy expected Python 2's module name:

```python
import ConfigParser
```

Ubuntu 24.04 runs the PAM Python module with Python 3, where the module is:

```python
import configparser
```

Back up the PAM file:

```bash
sudo cp -a /lib/security/howdy/pam.py \
  /lib/security/howdy/pam.py.bak.$(date +%Y%m%d-%H%M%S)
```

Patch `/lib/security/howdy/pam.py`:

```python
try:
    import configparser
except ImportError:
    import ConfigParser as configparser

config = configparser.ConfigParser()
```

This replaces the old Python 2-only code:

```python
import ConfigParser
config = ConfigParser.ConfigParser()
```

Syntax-check the patched file:

```bash
python3 - <<'PY'
import ast
from pathlib import Path

path = Path("/lib/security/howdy/pam.py")
ast.parse(path.read_text(), filename=str(path))
print("pam.py parses successfully")
PY
```

## Verify sudo face authentication

Reset the current sudo authentication timestamp:

```bash
sudo -k
```

Ask sudo to authenticate:

```bash
sudo -v
```

Working result:

```txt
Identified face as fahim
```

That means PAM called Howdy successfully and face authentication passed.

## Troubleshooting checklist

If it stops working:

- Check that `/dev/video2` is still the IR camera. Device numbers can change.
- Prefer stable paths under `/dev/v4l/by-path/` if available.
- Check that `/lib/security/howdy/config.ini` still has `device_path = /dev/video2`.
- Check that `/lib/security/howdy/config.ini` has no duplicate `[core]` or `[video]` sections.
- Check `/var/log/auth.log` for `howdy`, `pam_python`, or Python import errors.
- Check whether a package update reverted `/lib/security/howdy/pam.py`.
- Re-run `sudo howdy test` to confirm the camera and model still work.

Useful log command:

```bash
grep -R "howdy\|pam_python\|ConfigParser\|ModuleNotFound\|facial" \
  /var/log/auth.log /var/log/syslog 2>/dev/null | tail -80
```

## Security note

Howdy is convenient, not a replacement for strong authentication. The upstream
README explicitly warns not to use it as the sole authentication method.

Keep password authentication available as fallback.

## What to remember

The fix was not one thing. It was a chain:

1. Linux already saw the IR camera.
2. The IR video stream was `/dev/video2`.
3. Howdy needed a clean config with `device_path = /dev/video2`.
4. The old Howdy PAM file needed a Python 3 `configparser` patch on Ubuntu 24.04.
5. After that, `sudo -v` succeeded with `Identified face as fahim`.

## Related

- [Fixing Ubuntu Freezing with Swap and zswap](./fixing-ubuntu-freezing-with-zswap.md)
- [Why a package update sometimes does not show up in the current terminal](./hash-r-after-package-update.md)
