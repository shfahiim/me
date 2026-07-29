# open Alias for xdg-open

## What it is

`xdg-open` is the default Linux CLI command to open files, folders, or web URLs in your desktop's default application. Creating a shell alias (`alias open='xdg-open'`) brings macOS-style `open` behavior to Linux terminal workflows.

## Why it matters

`xdg-open` is tedious to type repeatedly. Mapping `open` to `xdg-open` gives you a quick 4-letter command to open anything directly from your terminal.

## Key idea

Add `alias open='xdg-open'` to your `~/.bashrc` (or `~/.zshrc`) so you can use `open <file-or-url>` anywhere.

## Example

Add to `~/.bashrc`:

```bash
echo "alias open='xdg-open'" >> ~/.bashrc
source ~/.bashrc
```

Use `open` for files, folders, or URLs:

```bash
open image.png           # Opens image in default image viewer
open document.pdf        # Opens PDF in default viewer
open https://google.com  # Opens URL in default browser
open .                   # Opens current directory in file manager
```

## Pitfalls

- Built-in `open` commands in some Linux distros (e.g. symlinked to `openvt`) might conflict if not aliased, but an interactive shell alias takes priority cleanly.
- `xdg-open` relies on system MIME associations to decide which application opens a file.

## Related

- [`mimeopen-change-default-app.md`](./mimeopen-change-default-app.md) — Change default application for file types used by `xdg-open`
