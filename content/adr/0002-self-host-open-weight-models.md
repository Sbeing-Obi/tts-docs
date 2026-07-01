---
title: "ADR-0002: Self-host commercially-licensed open-weight models"
tags: [adr, models, licensing]
status: accepted
date: 2026-07-01
---

# ADR-0002: Self-host commercially-licensed open-weight models

## Status

Accepted

## Context

The product is a TTS *service* others plug into, with voice cloning as the differentiator.
Two broad strategies exist:

1. **Wrap a commercial API** (ElevenLabs, Cartesia, …) — fast to ship, zero ML-ops, but thin margins, no moat, and voice cloning is gated by the provider.
2. **Self-host open-weight models** — own the stack and the cloning capability; best margins above a usage crossover (~10–50M characters/month); requires running GPU inference.

A critical constraint surfaced during research: many of the *best-sounding* open models ship
under **non-commercial** licenses (F5-TTS, XTTS-v2/Coqui, Fish self-host weights, MaskGCT,
Llasa, Spark-TTS, Higgs v3). Building on those would be a legal dead-end for a product.

## Decision

We will **self-host open-weight models**, and **only** ones under permissive,
commercial-safe licenses (MIT / Apache-2.0):

- **Chatterbox (MIT)** — primary engine: zero-shot cloning, multilingual, built-in PerTh watermark.
- **Kokoro-82M (Apache-2.0)** — CPU-capable narration fallback for zero-GPU development (no cloning).
- **CosyVoice 2 / Orpheus (Apache-2.0)** — evaluated for the low-latency streaming path.

We explicitly **avoid non-commercial-licensed models**, even for prototyping, to prevent a
rewrite later. The model sits behind a model-agnostic `TTSEngine` interface so it can be
swapped without breaking clients.

## Consequences

- Good: we own the cloning capability and the cost curve; no per-character API tax at scale; a real moat.
- Good: license safety verified up front; the `TTSEngine` abstraction keeps us free to switch models.
- Cost: we run GPU inference ourselves (mitigated early with local CPU + free Colab/Kaggle GPUs). Must re-verify each model's exact LICENSE at integration time, and respect underlying base-model licenses (e.g. Llama) for combined checkpoints.
