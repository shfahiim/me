# Fixing Ubuntu Freezing with Swap and zswap

## What it is

A solution to prevent Ubuntu from freezing when memory usage gets high on systems with limited RAM (8GB or less). The fix combines traditional swap with zswap, a kernel feature that compresses memory pages before writing them to disk.

## Why it matters

On systems with 8GB RAM, high memory usage can cause complete system freezes where even the cursor stops moving. Regular swap alone often doesn't help because disk I/O is too slow, and the system locks up before swap becomes effective.

zswap solves this by compressing memory pages in RAM first, reducing the need for slow disk swaps and keeping the system responsive.

## Key idea

**zswap = compressed RAM cache for swap**

Instead of writing directly to slow disk swap, zswap compresses pages in a RAM pool first. This means:
- Less disk I/O (the main cause of freezing)
- Faster swap access
- More effective use of available RAM
- System stays responsive under memory pressure

## How to set it up

### 1. Create swap file (if you don't have one)

```bash
# Create 6-8GB swap file
sudo fallocate -l 6G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### 2. Enable swap in fstab

```bash
# Add to /etc/fstab
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### 3. Enable zswap in GRUB

```bash
# Edit grub config
sudo nano /etc/default/grub

# Add to GRUB_CMDLINE_LINUX_DEFAULT line:
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash zswap.enabled=1"

# Update grub and reboot
sudo update-grub
sudo reboot
```

### 4. Verify it's working

```bash
# Check if zswap is enabled (should show 'Y')
cat /sys/module/zswap/parameters/enabled

# Check swap usage
swapon --show
```

## What you'll see

With zswap active:
- Swap will show as being used (good sign)
- System won't freeze even when swap usage is high
- Memory-heavy tasks stay responsive

Without zswap:
- System freezes before swap can help
- Disk thrashing causes complete lockup

## Pitfalls

- **Forgetting to update grub**: Adding `zswap.enabled=1` to `/etc/default/grub` isn't enough - you must run `sudo update-grub` and reboot
- **Not having swap at all**: zswap needs actual swap space to work - it's a compression layer, not a replacement
- **Too little swap**: With 8GB RAM, use at least 4-6GB swap

## Related

- `hash-r-after-package-update.md` - Another system-level fix worth remembering

---

## From the Community (Reddit discussion)

Source: [r/Ubuntu - Ubuntu 24.04 filling up RAM and when it is full the system freezing](https://www.reddit.com/r/Ubuntu/comments/1qh4k3s/ubuntu_2404_filling_up_ram_and_when_it_is_full/)

The thread confirms the zswap approach above, but also surfaces other solutions and edge cases worth knowing.

### zram vs zswap

Don't use zram and swap together. Pick one:

- **zswap + swap** (what this note covers): compressed cache layer in front of disk swap. Recommended.
- **zram** alone: creates a compressed RAM block device as swap. No disk swap needed, but lower ceiling.

Some users suggested `sudo apt install zram-tools` or `sudo apt install zram-config` as simpler alternatives, but zswap with a proper swapfile gives better control.

### Alternative: early OOM killers

If you don't want to deal with swap at all, install something that kills memory-hungry processes *before* the system freezes:

```bash
sudo apt install earlyoom
sudo systemctl enable --now earlyoom
```

Other options mentioned: `nohang` (version 0.3+), `systemd-oomd`.

### The real cause might be a memory leak

In the original thread, the poster found that **gjs** (GNOME JavaScript runtime) was consuming 1.8GB in 10 minutes and filling all RAM and swap overnight. Uninstalling gjs fixed it entirely.

If your system keeps filling memory even with no apps open, check for a leak first:

```bash
# Sort by memory usage, watch over time
htop  # press F6, sort by MEM%
```

Common culprits:
- **gjs**: GNOME shell extensions or bad GNOME config. Try clearing: `~/.config/gnome-shell`, `~/.local/share/gnome-shell`, `~/.cache/gjs`
- **Browsers**: many tabs, especially with video (YouTube)
- **Brave / Chrome**: known to balloon memory over time

### Diagnosis tools mentioned

```bash
sudo apt install htop btop
dmesg -Tw  # kernel messages, useful for OOM events
```

### Swap file vs swap partition

There was debate in the thread. Short version:
- **Swap file** (what most people use): fine on modern Linux, easy to create
- **Swap partition**: no fragmentation risk, can be shared across distros if dual booting, slightly cleaner
- For most single-OS setups, a swap file is perfectly sufficient
