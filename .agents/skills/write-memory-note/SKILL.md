---
name: write-memory-note
description: Drafts new knowledge-base notes for this repository. Use when the user wants to add a memory note, capture a fact, or write markdown under memory/ following the repo's note conventions.
---

# Write memory notes

Notes live in `memory/` as flat markdown files. Each file is one atomic idea.

## Front matter

Every note must start with YAML front matter using the repo pattern:

```markdown
---
title: Short human-readable title
tags:
- topic-one
- topic-two
---

Body starts here.
```

**Tags**: Required. Use lowercase, hyphenated or single-word labels that help browse and discover related notes (e.g. `css`, `react`, `cli`). Prefer a few precise tags over many vague ones.

## Body rules

1. **Be concise.** Prefer short paragraphs and bullets.
2. **Link, don't re-explain.** If a concept already exists in another note, cite it with a relative markdown link: `[scroll container](scroll-containers.md)`. Only state what is *new* in this note; assume linked notes are the canonical explanation.
3. **One idea per file.** If you need two big ideas, split into two files and cross-link them.

## Cross-references

- Link to sibling files under `memory/` with paths relative to the current file (e.g. `[overflow: clip](overflow-clip.md)`).

## Checklist before finishing

- [ ] `title` and at least one `tags` entry in front matter
- [ ] Body is tight; no duplicated explanations available in other notes
- [ ] Other notes are linked where relevant
- [ ] Filename is lowercase, words separated by hyphens, ending in `.md` (match existing `memory/*.md` style)
