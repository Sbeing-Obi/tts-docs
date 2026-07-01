---
title: TTS Service — Documentation
tags: [home, moc]
---

# TTS Service Docs

The knowledge base for a **self-owned, human-like Text-to-Speech API with voice cloning**.
This vault is the home of record for every decision, explanation, and how-to as the
product is built. It is authored in plain, portable Markdown so the same files render in
Obsidian, on GitHub, and on the published Quartz site.

> [!NOTE]
> Documentation here follows the [Diátaxis](https://diataxis.fr/) model: four distinct
> kinds of docs, each kept separate so none drifts into another.

## Map of the docs

- **Explanation** — understanding-oriented: *why* the system is the way it is.
  - [How modern (codec-LM) TTS works](explanation/how-codec-lm-tts-works.md)
- **Decisions (ADRs)** — the record of architecturally significant choices.
  - [Decision log](adr/index.md)
- **How-to guides** — task-oriented recipes.
  - [Run the service locally](how-to/run-the-service-locally.md)
  - [Enable IndicF5 (GPU)](how-to/enable-indicf5-gpu.md)
  - [Fine-tune your own voice](how-to/fine-tune-your-voice.md)
  - [Publish the docs site](how-to/publish-the-docs-site.md)
- **Reference** — information-oriented API facts.
  - [POST /v1/audio/speech](reference/v1-audio-speech.md)
  - [WS /v1/realtime (streaming)](reference/v1-realtime.md)
  - [Voice enrollment (/v1/voices)](reference/v1-voices.md)
- **Dev-log** — per-session engineering journal (what I did / why / what I learned).
  - [2026-07-01 — Project kickoff](devlog/2026-07-01.md)
- **Changelog** — what changed and when → [changelog](changelog.md)

_Coming as the build progresses (Diátaxis says: create these when there's something to put in them):_
- **Tutorials** — learning-oriented guided lessons (Phase 2+)

## How this vault is maintained

A change to the product is not "done" until it is documented, in the **same commit**:

1. Commit message is a Conventional Commit (`feat:` / `fix:` / `docs:` …).
2. `changelog.md` `[Unreleased]` updated if the change is user-visible.
3. A hard-to-reverse decision → a new ADR in `adr/`.
4. Behavior changed → touch the relevant how-to / reference page.
5. Something surprised you → one line in the dev-log.

See [ADR-0005](adr/0005-separate-quartz-docs-vault.md) for why this vault is separate and how it publishes.
