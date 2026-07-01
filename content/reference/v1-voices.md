---
title: Reference — voice enrollment (/v1/voices)
type: reference
status: current
last-verified: 2026-07-01
tags: [reference, api, cloning]
---

# Reference — voice enrollment

Cloning engines (IndicF5) need a reference clip **and** its transcript. Enroll them once to
get a `voice_id`, then pass that id as `voice` on `/v1/audio/speech`. See ADR-0007.

## POST /v1/voices

`multipart/form-data`

| field | type | required | notes |
|---|---|---|---|
| `audio` | file | yes | reference clip (WAV, <= 6s recommended) |
| `ref_text` | string | yes | exact transcript of the clip |
| `language` | string | no | ISO code, e.g. `hi` |
| `engine` | string | no | default `indicf5` |

Response: `{ "voice_id": "...", "engine": "indicf5", "language": "hi" }`

## GET /v1/voices

Lists enrolled voices: `{ "voices": [ { "id", "engine", "language" } ] }`

## Using a cloned voice end-to-end

```bash
# 1. enroll (returns a voice_id)
curl -X POST http://localhost:8000/v1/voices \
  -F audio=@me.wav -F ref_text="exactly what I said in me.wav" -F language=hi

# 2. speak Hindi in your voice (needs the IndicF5 engine enabled — see the how-to)
curl http://localhost:8000/v1/audio/speech -H "Content-Type: application/json" \
  -d '{"model":"indic","language":"hi","voice":"<voice_id>","input":"नमस्ते"}' --output hi.wav
```

## Related

- [How to enable IndicF5 (GPU)](../how-to/enable-indicf5-gpu.md)
- [ADR-0007: Indic multilingual stack](../adr/0007-indic-multilingual-stack.md)
- Source: `app/api/voices.py`, `app/voice/store.py`, `app/engine/indicf5.py`, `app/engine/router.py`
