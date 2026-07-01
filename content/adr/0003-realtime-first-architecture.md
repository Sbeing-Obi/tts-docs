---
title: "ADR-0003: Optimize for real-time streaming first"
tags: [adr, architecture, streaming]
status: accepted
date: 2026-07-01
---

# ADR-0003: Optimize for real-time streaming first

## Status

Accepted

## Context

TTS has two broad use cases with very different constraints:

- **Real-time voice agents** (phone bots, live assistants, game NPCs) need audio to *start*
  within ~200–300 ms or the conversation feels broken.
- **Batch / long-form** (audiobooks, content) cares about quality and cost-per-character,
  not latency.

Real-time is the harder constraint. An architecture that can stream the first audio chunk
in well under 300 ms can also serve batch by simply buffering to completion — but a
batch-first architecture cannot be retrofitted into low latency without rework.

## Decision

We will **optimize for the real-time streaming path first**. Concretely:

- Expose a streaming **`/v1/realtime` WebSocket** alongside the one-shot
  **`/v1/audio/speech`** REST endpoint (both OpenAI-compatible).
- Stream **24 kHz 16-bit PCM** chunks from a background inference thread via an
  `asyncio.Queue`; chunk text by clause so synthesis starts on the first phrase.
- Track **time-to-first-audio (TTFA)** as a first-class metric with a <300 ms target.
- Favor autoregressive / codec-LM models (low TTFA) over diffusion/flow-matching for the
  live path.

## Consequences

- Good: solving the hard constraint subsumes the easy one (batch falls out for free).
- Good: positions the product for the hot voice-agent market and telephony (Twilio/LiveKit) later.
- Cost: streaming inference, cancellation, and chunking are more complex than one-shot generation. We accept this complexity deliberately and document the pipeline in an explanation note.
