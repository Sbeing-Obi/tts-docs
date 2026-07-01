---
title: "ADR-0006: Build our own model by fine-tuning an open base"
tags: [adr, models, fine-tuning]
status: accepted
date: 2026-07-01
---

# ADR-0006: Build our own model by fine-tuning an open base

## Status

Accepted. Refines [ADR-0002](0002-self-host-open-weight-models.md): its licensing
constraint still holds; this decision sets *how* we get to "our own" product model.

## Context

The goal is for the service to run **our own** model, not simply someone else's used
as-is. Three paths exist: (1) train from scratch, (2) fine-tune an open base, (3) self-host
open weights unchanged. Training from scratch needs 100s–1000s of hours of data, thousands
of dollars of GPU, ML research, and months — infeasible for a solo, no-budget start.
Self-hosting open weights (current: Kokoro) works but the weights aren't ours. **Fine-tuning
a commercially-licensed open base on our own data yields a model that is practically and
legally ours, at a cost of a few GPU dollars and days.**

## Decision

We will produce "our model" by **fine-tuning a commercially-licensed (Apache-2.0 / MIT)
open base on our own data/voice**, and ship the result behind the existing `TTSEngine`
interface. Kokoro stays the CPU dev placeholder; the fine-tuned model becomes the product
engine. The base choice (candidate: **Chatterbox**, MIT) is confirmed by research at
implementation time. Actual training runs use a GPU (free Colab / Kaggle to start),
consistent with the deferred-paid-compute stance of [ADR-0004](0004-both-cloning-tiers.md).

## Consequences

- Good: the product model is differentiated and ours, with **no architectural change** — it drops in as another `TTSEngine`.
- Good: realistic for a solo / low-budget builder (fine-tune, not from-scratch).
- Cost: we must collect/record training data and run GPU training; output quality depends on data quality. From-scratch training remains a possible funded future phase.
