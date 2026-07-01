---
title: Reference — platform (auth, keys, usage)
type: reference
status: current
last-verified: 2026-07-01
tags: [reference, api, platform]
---

# Reference — multi-tenant platform

API keys, authentication, rate limiting, and usage metering. Enforcement is **off by default**
(local dev runs as tenant `dev`); turn it on with `TTS_REQUIRE_API_KEY=1`.

## Authentication

Send your key as `Authorization: Bearer tts_...` (or `X-API-Key: tts_...`). Over the
`/v1/realtime` WebSocket, pass it in the first message as `api_key` (or the Bearer header).

## POST /admin/keys — create a key

Guarded by `X-Admin-Token: <TTS_ADMIN_TOKEN>`.

```
{"tenant": "acme", "name": "prod"}   ->   {"api_key": "tts_...", "tenant": "acme"}
```

## GET /v1/usage — your usage

Auth required. → `{"tenant": "acme", "requests": N, "characters": N}`

## Rate limiting

Cost-based on **characters** per key, per 60-second window (a 5-word and a 5,000-char request
differ wildly). Over budget → HTTP `429` (or an `error` message over the WebSocket).

## Environment

| var | default | meaning |
|---|---|---|
| `TTS_REQUIRE_API_KEY` | off | enforce API keys |
| `TTS_ADMIN_TOKEN` | — | token required by `/admin/keys` |
| `TTS_RATE_LIMIT_CPM` | 100000 | characters/min per key |
| `TTS_CACHE_MAX` | 512 | in-memory audio cache entries (0 disables) |

## Caching

Identical requests (same model, text, voice, format, speed, language) are served from an
in-memory LRU cache — a hit **skips synthesis** (seconds → ~1ms, $0 compute) and returns
byte-identical audio. Responses carry `X-Cache: HIT|MISS`. Swap for Redis/CDN at scale.

## Related

- [How to run as a platform](../how-to/run-as-a-platform.md)
- Source: `app/infra/` (keys, auth, ratelimit, usage), `app/api/admin.py`, `app/api/usage.py`
