---
title: "ADR-0004: Support both cloning tiers (zero-shot first)"
tags: [adr, cloning, product]
status: accepted
date: 2026-07-01
---

# ADR-0004: Support both cloning tiers (zero-shot first)

## Status

Accepted

## Context

Voice cloning comes in two tiers:

- **Zero-shot (instant):** a speaker encoder turns a ~5–10 s sample into an embedding that
  conditions synthesis. No training; near-instant; lower fidelity.
- **Fine-tuned (professional):** the model/adapter is trained on 30 min–3 h of a speaker's
  audio. Higher fidelity and expressiveness; requires **paid GPU training hours**.

The product goal is to offer **both tiers**. The constraint is a **zero budget** at the
start, which collides with the paid compute that fine-tuning requires.

## Decision

We will support **both tiers in the architecture and code from day one**, but **ship the
zero-shot tier fully working first** and **defer the paid fine-tune training runs** until a
small GPU budget exists. Concretely:

- The voice lifecycle, database schema, and `/v1/voices` API model **both** a `zero_shot`
  and a `professional` voice type from the start.
- The fine-tune/adapter **pipeline is built** (and testable on tiny data / free GPU
  sessions), but production training is gated behind a feature flag until funded.
- Nothing in the design assumes only one tier; switching a voice from instant to
  professional is a data/training change, not a rewrite.

## Consequences

- Good: honors the "both tiers" product intent without blocking on money; no architectural rework when fine-tuning is funded.
- Good: zero-shot delivers a usable cloning product at $0 (local CPU + free GPUs).
- Cost: we carry some unused-at-first schema/pipeline complexity. Accepted, because retrofitting a second tier later is far more expensive than designing for it now.
