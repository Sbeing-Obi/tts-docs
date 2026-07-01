---
title: How modern (codec-LM) TTS works
tags: [explanation, tts, concepts]
status: current
last-verified: 2026-07-01
---

# How modern (codec-LM) TTS works

This note explains *why* today's text-to-speech sounds human, and the building blocks our
service is made of. It is understanding-oriented — read it once to get the mental model that
the rest of the docs assume.

## The one-sentence idea

> [!IMPORTANT]
> A modern TTS engine is **structurally a language model** — it predicts *audio* tokens the
> same way an LLM predicts *text* tokens. That single shift is why prosody, emotion, breaths,
> and laughter finally sound natural.

## Why older approaches sounded robotic

- **Concatenative (unit selection):** stitch together tiny recorded snippets of real speech.
  Intelligible, but the joins are audible and it can't say anything outside its recordings.
- **Parametric / HMM:** generate speech from statistical parameters. Flexible, but buzzy and
  over-smoothed.
- **Tacotron-era (mel + neural vocoder):** a big leap in quality, but limited at long-range
  prosody and prone to attention failures (skipped/repeated words).

All three either lack expressive prosody or impose it with hand-written rules.

## The modern pipeline: audio as tokens

The current generation (VALL-E 2, Chatterbox, CosyVoice 2, Orpheus, Higgs) works in three
stages:

1. **Encode audio into discrete tokens.** A *neural audio codec* (EnCodec, DAC, SNAC, Mimi)
   compresses a waveform into a sequence of discrete "acoustic tokens" — like a vocabulary
   for sound. The codec's **frame rate** (tokens per second) sets how long the sequence is,
   which directly drives latency and cost.
2. **Predict tokens with a transformer.** An autoregressive language model — often a small
   Llama-family backbone — predicts the next audio token given the text and any voice
   conditioning. Because it is a language model over *sound*, it naturally learns rhythm,
   emphasis, emotion, and even the acoustic room of the prompt, instead of being told them.
3. **Decode tokens back to a waveform.** The codec's decoder (or a neural vocoder such as
   Vocos / HiFi-GAN) turns the predicted tokens back into audio you can play.

```
text ──▶ [ AR transformer ] ──▶ audio tokens ──▶ [ codec decoder / vocoder ] ──▶ waveform
                ▲
        voice conditioning (speaker embedding)
```

## Where voice cloning fits

Cloning is just **conditioning** in stage 2. A **speaker encoder** (often borrowed from
speaker-verification, e.g. ECAPA-TDNN) turns a reference clip into a *speaker embedding* — a
vector capturing timbre. Feed that embedding to the transformer and it generates *your text*
in *that voice*.

- **Zero-shot:** compute the embedding from a ~5–10 s sample at inference time. Instant, no
  training. Captures timbre well; flattens fine prosody.
- **Fine-tuned:** adapt the model/adapter on 30 min–3 h of a speaker. Recovers pacing and
  emotional range for flagship fidelity. (See [ADR-0004](../adr/0004-both-cloning-tiers.md).)

## What makes it sound *human*

The cues your ear reads as "a real person": prosodic variation, emotional range, **breaths,
hesitations/disfluencies, pacing and pauses**, and natural turn-taking. Codec-LM models
reproduce these from the prompt and from inline emotion/audio tags — not from rules. That is
the whole reason we chose this model family.

## Why this matters for *our* architecture

- **Streaming:** because tokens are produced one-by-one, we can decode and stream audio as it
  is generated — the basis of our <300 ms time-to-first-audio target
  (see [ADR-0003](../adr/0003-realtime-first-architecture.md)).
- **Model-agnostic engine:** stages 1–3 differ per model, so we hide them behind a single
  `TTSEngine` interface and swap models freely (see
  [ADR-0002](../adr/0002-self-host-open-weight-models.md)).
- **Watermarking:** since we synthesize the waveform, we can embed an inaudible watermark at
  stage 3 — required for safety and law.

## Further reading

- VALL-E 2 (human-parity zero-shot TTS): https://arxiv.org/html/2406.05370v1
- Chatterbox (our primary model): https://github.com/resemble-ai/chatterbox
- Neural codecs overview (Mimi / SNAC / DAC): see the codec links in the project research notes.
