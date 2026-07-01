---
title: "ADR-0005: Separate Quartz docs vault, portable authoring"
tags: [adr, documentation]
status: accepted
date: 2026-07-01
---

# ADR-0005: Separate Quartz docs vault, portable authoring

## Status

Accepted

## Context

A hard requirement: every piece of work is documented as it is built, in Obsidian, and the
docs must be **usable anywhere and in any case** — not locked to one tool. Two questions
followed: *where does the vault live*, and *how do we keep the files portable*.

Research surfaced a key fact: Obsidian's nicest features — `[[wikilinks]]`, `![[embeds]]`,
Dataview, `==highlights==` — are **non-standard** and render as broken text on GitHub or in
other renderers. Portability is therefore an **authoring convention**, not a tool choice.

## Decision

1. **Separate private repo** for docs (`tts-docs`), distinct from the code repo
   (`tts-service`). It is a **Quartz v5** project whose **`content/` folder is the Obsidian
   vault**. `npx quartz sync` commits, pushes, and rebuilds the published site (free GitHub
   Pages). This keeps the personal dev-log out of the code repo and makes the knowledge base
   reusable across future projects.
2. **Portable authoring convention** (enforced from file #1):
   - Wikilinks **off**; use standard relative links `[text](../adr/0001-….md)`.
   - Callouts limited to GitHub-compatible types: `NOTE, TIP, IMPORTANT, WARNING, CAUTION`.
   - Standard image syntax `![alt](./assets/x.png)`, relative paths.
   - Filenames lowercase-hyphenated; always include `.md` in links.
   - No Dataview/Templater syntax in published files (templates live in a Quartz-ignored folder).
3. **Definition of Done = documented** — the 5-point in-commit gate (see the home index).

## Consequences

- Good: the same files render correctly in Obsidian, on GitHub, and in Quartz — and Quartz still builds a graph/backlinks from standard links, so the "networked garden" feel is kept.
- Good: plain Markdown is future-proof; the publishing generator is swappable with no rewrite.
- Cost: we give up a few Obsidian conveniences (wikilink autocomplete, transclusion). Accepted as the price of true portability.
