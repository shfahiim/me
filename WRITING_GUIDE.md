# Writing Guide

This repo should make it easy to capture knowledge quickly and find it again later.

## Core Rule

Write notes so that future-you can understand them in a few minutes.

## Recommended Note Shape

Use this structure when it helps:

```md
# Title

## What it is
Short explanation in plain language.

## Why it matters
Why this is useful or when it becomes relevant.

## Key idea
The main concept to remember.

## Example
A short example, command, or snippet.

## Pitfalls
What is easy to misunderstand or forget.

## Related
Links to other notes in this repo.
```

Not every note needs every section. Keep what helps and remove what does not.

## Writing Rules

- Prefer short files over long, mixed notes.
- One note should usually cover one concept, issue, or lesson.
- Explain things in your own words.
- Use examples from real usage when possible.
- If you copy a command or snippet, add one line explaining why it matters.
- If something was confusing, document the confusing part explicitly.
- If a note becomes too large, split it.
- After writing a note, commit and push it so the repo stays current.

## File Naming

Use lowercase kebab-case filenames:

- `react-rendering-basics.md`
- `nginx-reverse-proxy-notes.md`
- `debugging-node-memory-issues.mdx`

Avoid:

- spaces in filenames
- dates at the start unless the note is journal-like
- generic names like `note1.md`

## When To Use Markdown vs MDX

Use `.md` when the note is plain text and code blocks.

Use `.mdx` when you specifically need:

- interactive components
- embedded demos
- richer presentation later

Default to `.md` unless there is a reason not to.

## What Makes A Note Useful

A useful note usually answers at least one of these:

- What is this?
- Why does it matter?
- When should I use it?
- What do I keep forgetting?
- What mistake did I make?
- What is the shortest correct example?

## Categories

Use folders only when they help retrieval.

Suggested categories:

- `notes/frontend/`
- `notes/backend/`
- `notes/devops/`
- `notes/tools/`
- `notes/misc/`

If a category is too broad later, split it then. Do not over-design early.

## Editing Existing Notes

When revisiting a note:

- fix unclear wording
- add the missing insight you forgot last time
- remove stale or repeated content
- link related notes
- keep the best example near the top

The goal is not to preserve every draft. The goal is to preserve useful understanding.

## Git Habit

Treat each note as a small saved unit of learning.

Suggested flow:

```bash
git add .
git commit -m "Add note on <topic>"
git push
```

Push after each note or meaningful note update.

