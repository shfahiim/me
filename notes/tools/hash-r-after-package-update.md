# Why a package update sometimes does not show up in the current terminal

## What it is

Sometimes I update a package on Linux, but the current terminal session still behaves like nothing changed. If I open a new terminal tab, the updated command works there.

## Why it matters

This can make it look like the package update failed, even when it actually succeeded. The problem is often just the current shell session, not the package manager.

## Key idea

The shell can cache command locations. After a package update, the current shell may still use the old cached command path instead of checking again.

A new terminal tab works because it starts a fresh shell session with a fresh command lookup state.

## Fix

The practical fix I used:

```bash
hash -r
```

This clears Bash's command hash table so the shell looks up commands again.

## When this happens

This usually shows up when:

- a package updates a binary
- the command path changes
- the current shell still remembers the old command location

## Example

I update a package in one terminal, try the command again, and the old behavior remains. Running `hash -r` in that same terminal fixes it without needing to open a new tab.

## Related

- Shell command lookup
- `$PATH`
- `type <command>`
