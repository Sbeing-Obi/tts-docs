---
title: "ADR-0007: Indic multilingual stack (IndicF5 + Indic Parler-TTS)"
tags: [adr, models, multilingual, indic]
status: accepted
date: 2026-07-01
---

# ADR-0007: Indic multilingual stack (IndicF5 + Indic Parler-TTS)

## Status

Accepted. Refines [ADR-0006](0006-finetune-open-base.md): fine-tuning is now **optional /
deferred**, not the v1 path — zero-shot works out of the box.

## Context

The product goal is a TTS covering **Indian languages** (target the 22 scheduled languages)
**with voice cloning**, under a **commercial (MIT/Apache)** license. Research (mid-2026)
found that **no single open model** satisfies coverage + cloning + commercial license at
once. The pragmatic answer is a two-model stack. Zero-shot validation of the user's own
voice on Chatterbox already sounded like them, but Chatterbox is English-only, so Hindi
words got an English accent — a *language-coverage* gap, not a voice-identity gap.

## Decision

Ship a **two-model Indic stack behind one language-routing composite engine** (model id
`indic`), with no API/route changes:

- **IndicF5 (MIT)** — primary. 11 languages (Assamese, Bengali, Gujarati, Hindi, Kannada,
  Malayalam, Marathi, Odia, Punjabi, Tamil, Telugu) with **zero-shot personal cloning**
  (reference clip + its transcript). Tier 1 — ship first.
- **Indic Parler-TTS (Apache-2.0)** — coverage backstop for the remaining scheduled
  languages (Urdu, Sanskrit, Konkani, …) as **preset/described voices** (no personal clone).
  Tier 2 — enable per-language after QA.
- **Kokoro** stays the CPU English fast-path.

A composite `IndicRouterEngine(TTSEngine)` dispatches on `SynthesisRequest.language`.
Cloning is offered **only for the 11 IndicF5 languages**; the voice-store `voice` id already
anticipated in `base.py` carries the enrolled clip. **No fine-tuning for v1.** Fine-tuning
(F5-TTS pipeline; CC-BY datasets: IndicVoices-R, Rasa, IndicTTS) is reserved for adding
Tier-2 cloning, a brand voice, or improving a weak language.

## Consequences

- Good: reaches all 22 scheduled languages (11 with cloning); **zero API/client churn** — just new engines; no training needed for v1.
- Good: realistic and free/cheap for a solo dev; models are MIT/Apache.
- Cost/limits: cloning limited to 11 languages (honest limitation to state to users); IndicF5 needs a GPU (Colab/rented) — Kokoro remains the CPU dev path; two backends + routing add some complexity; IndicF5 needs the reference **transcript** and same-language references clone best.
