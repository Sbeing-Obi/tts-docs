---
title: How to run as a multi-tenant platform
tags: [how-to, platform]
---

# How to run as a multi-tenant platform

Turn on API keys, rate limiting, and usage metering so others can sign up and use your API.

## 1. Enable it

```bash
TTS_REQUIRE_API_KEY=1 TTS_ADMIN_TOKEN=choose-a-secret uvicorn app.main:app
```

## 2. Create a key for a customer

```bash
curl -X POST http://localhost:8000/admin/keys \
  -H "X-Admin-Token: choose-a-secret" -H "Content-Type: application/json" \
  -d '{"tenant":"acme","name":"prod"}'
# -> {"api_key":"tts_...","tenant":"acme"}
```

## 3. They call the API with it

```bash
curl http://localhost:8000/v1/audio/speech \
  -H "Authorization: Bearer tts_..." -H "Content-Type: application/json" \
  -d '{"model":"kokoro","input":"Hello","voice":"af_heart"}' --output hello.wav
```

## 4. Usage & limits

- `GET /v1/usage` (with the key) → requests + characters for that tenant.
- Rate limit: `TTS_RATE_LIMIT_CPM` characters/min per key (429 when exceeded).

> [!NOTE]
> v1 uses SQLite + in-memory rate limiting (single process). For horizontal scale, move
> keys/usage to Postgres and rate limiting to Redis — no API changes needed.

## Related

- [Platform API reference](../reference/platform-api.md)
