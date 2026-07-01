---
title: Reference — POST /v1/audio/speech
type: reference
status: current
last-verified: 2026-07-01
tags: [reference, api]
---

# Reference — `POST /v1/audio/speech`

One-shot text-to-speech. OpenAI-compatible, so existing OpenAI client SDKs work by simply
pointing at this server's base URL.

## Request

`POST /v1/audio/speech` · `Content-Type: application/json`

| field | type | required | default | notes |
|---|---|---|---|---|
| `model` | string | yes | — | engine id: `kokoro` (alias `tts-1`) |
| `input` | string | yes | — | text to speak (≤ 4096 chars) |
| `voice` | string | no | `af_heart` | preset voice id (see below) |
| `response_format` | string | no | `wav` | `wav` or `pcm` (Phase 1) |
| `speed` | number | no | `1.0` | 0.25–4.0 |
| `language` | string | no | `en-us` | engine extension (not in OpenAI's spec) |
| `instructions` | string | no | — | accepted, ignored (Kokoro has no style control) |

## Response

Raw audio bytes (not JSON).

| `response_format` | Content-Type | details |
|---|---|---|
| `wav` | `audio/wav` | RIFF/WAVE, 16-bit PCM, mono, 24000 Hz |
| `pcm` | `audio/pcm` | headerless signed 16-bit little-endian, mono, 24000 Hz (rate is out-of-band) |

## Errors

| status | when |
|---|---|
| 404 | unknown `model` |
| 422 | invalid `response_format` (only `wav`/`pcm` in Phase 1) or request validation error |
| 400 | engine failure (e.g. empty text) |

## Example

```bash
curl http://127.0.0.1:8000/v1/audio/speech \
  -H "Content-Type: application/json" \
  -d '{"model":"kokoro","input":"Hello world","voice":"af_heart","response_format":"wav"}' \
  --output hello.wav
```

## Voices

54 preset voices (Kokoro). Ids look like `af_heart`, `af_bella`, `am_michael`, `bf_emma`, …
The prefix encodes locale + gender (`a`=American, `b`=British; `f`=female, `m`=male). Query
the loaded set programmatically via the engine's `list_voices()`.

## Related

- [How to run the service locally](../how-to/run-the-service-locally.md)
- [How codec-LM TTS works](../explanation/how-codec-lm-tts-works.md)
- Decisions: [ADR-0002](../adr/0002-self-host-open-weight-models.md) · [ADR-0003](../adr/0003-realtime-first-architecture.md)
- Source: `app/api/speech.py`, `app/engine/kokoro.py`, `app/engine/base.py`
