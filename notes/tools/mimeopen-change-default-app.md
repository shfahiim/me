# mimeopen — Change Default App for a File Type

## What it is

`mimeopen` is a command-line tool that opens a file using its MIME type handler. The useful side effect: it lets you interactively pick and save a new default application for that file type.

## Why it matters

`xdg-open` respects whatever default is registered for a MIME type. If the wrong app keeps opening, you fix it through `mimeopen`, not `xdg-open`.

## Key idea

Run `mimeopen -d` on a file once to set the default. After that, `xdg-open` will use it automatically.

## Example

Change the default app for `.md` files:

```bash
mimeopen -d some-file.md
```

It lists available apps and prompts you to pick one. After you choose, that app becomes the default for all `.md` files.

From then on:

```bash
xdg-open some-file.md   # opens with the app you chose
```

## Pitfalls

- `xdg-open` alone cannot change the default — it only uses what is already set.
- The `-d` flag is what saves the choice. Without it, `mimeopen` just opens the file once.

## Related

- `xdg-mime query default text/markdown` — check what the current default is
- `xdg-mime default app.desktop text/markdown` — another way to set it if you already know the `.desktop` file name
