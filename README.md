# Between Learning and Unlearning

This repository is a personal knowledge base.

The idea is simple: I learn things, use them, forget parts of them, and then relearn them. Instead of letting that cycle stay invisible, this repo keeps a written record of what was learned, what mattered, and what is worth revisiting.

The motto is:

> Between learning and unlearning.

## Purpose

This repo is for notes that are:

- practical
- personal
- reusable
- easy to revisit later

It is not meant to be a perfect wiki. It is meant to be useful.

## What Goes Here

You can add:

- topic notes
- short explainers
- setup guides
- debugging lessons
- code snippets with context
- mistakes worth remembering
- things you had to relearn

Use either `.md` or `.mdx`, depending on whether you need embedded components later.

## Suggested Structure

Start simple and grow only when needed.

```text
.
├── README.md
├── WRITING_GUIDE.md
├── notes/
│   ├── README.md
│   ├── frontend/
│   ├── backend/
│   ├── devops/
│   ├── tools/
│   └── misc/
└── templates/
    └── note-template.md
```

You do not need to create every folder immediately. Add categories only when they become useful.

## How To Write Notes

Use this default flow:

1. Create a new file inside `notes/`.
2. Name it clearly, for example: `react-state-vs-ref.md` or `git-rebase-mistakes.md`.
3. Start from [`templates/note-template.md`](./templates/note-template.md).
4. Keep the note focused on one idea or one problem.
5. Add links to related notes when they exist.
6. Commit and push the note after writing it.

Detailed writing guidance lives in [`WRITING_GUIDE.md`](./WRITING_GUIDE.md).

## Update Habit

After adding or significantly updating a note:

1. Review the note quickly for clarity.
2. Commit it with a direct message.
3. Push it to the remote.

Example:

```bash
git add .
git commit -m "Add note on hash -r after package updates"
git push
```

## Principles

- Write for your future self.
- Prefer clarity over completeness.
- Keep notes atomic: one file, one idea.
- Record why something matters, not just what it is.
- Include examples when they reduce future confusion.
- If something was hard to understand once, write it down.

## Good Note Titles

- `how-http-caching-actually-works.md`
- `postgres-indexes-basics.md`
- `why-this-docker-build-was-slow.md`
- `tailwind-class-ordering.md`
- `things-i-keep-forgetting-about-promises.md`

Avoid vague names like:

- `notes.md`
- `random.md`
- `stuff-i-learned.md`

## Maintenance

Over time, you can:

- move notes into topic folders
- add cross-links between related ideas
- split large notes into smaller ones
- add summary index files per topic

The system should stay lightweight. If organization becomes heavier than writing, simplify it.

