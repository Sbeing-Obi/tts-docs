---
title: Changelog
tags: [changelog]
---

# Changelog

All notable changes to this project are documented here.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and the project aims to follow [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added

- **Phase 0 — code:** `tts-service` scaffold — FastAPI app exposing `GET /health`, pytest suite, packaging via `pyproject.toml`.
- **Phase 0 — docs:** `tts-docs` Obsidian vault + Quartz project. Seeded decision log (ADR-0001…0005), the "How modern codec-LM TTS works" explanation note, four authoring templates, and the portable-authoring convention.
- **Phase 0 — publish:** Quartz v5 wired to `content/` with `quartz.config.yaml`, a GitHub Actions deploy workflow, and a how-to for publishing the site.
- **Phase 1 — voice:** OpenAI-compatible `POST /v1/audio/speech` backed by a CPU `KokoroEngine` (kokoro-onnx). Model-agnostic `TTSEngine` interface, WAV/PCM output, engine registry, and pytest coverage. First audio generated with zero GPU.
- **Direction:** decided to build our own model by fine-tuning Chatterbox (MIT) on the user's own recorded voice via LoRA (ADR-0006); added the fine-tuning playbook how-to.
- **Direction (multilingual):** validated Chatterbox zero-shot on the user's voice (sounds like them). Product goal set to **Indian-language TTS + cloning**; adopted a two-model Indic stack — **IndicF5 (MIT, 11 languages, cloning) + Indic Parler-TTS (Apache, coverage)** — behind a language-routing engine, no API changes, no fine-tuning for v1 (ADR-0007).
- **Validated:** IndicF5 zero-shot on the user's own voice (Colab) — their voice speaking Hindi with correct Indic pronunciation, ~8/10, no training. Confirms IndicF5 as the v1 Indic engine (needs a GPU + `transformers==4.49.0`).
- **Indic integration:** `IndicF5Engine` (GPU, cloning), `IndicRouterEngine` (routes by language, model `indic`), a SQLite voice store, and `POST /v1/voices` enrollment — all behind the unchanged `/v1/audio/speech` API, gated by `TTS_ENABLE_INDICF5`. 10 tests green (store/router/enrollment verified locally without a GPU).
- **Phase 2 — streaming:** `WS /v1/realtime` streams per-sentence audio as it's generated (Indic-aware sentence chunker incl. Devanagari danda), for low time-to-first-audio. Engine-agnostic; 12 tests green.
- **Platform (multi-tenant):** API keys (`/admin/keys`), Bearer auth on synthesis endpoints, cost-based (characters) rate limiting, and usage metering (`/v1/usage`) — behind `TTS_REQUIRE_API_KEY` (off in dev). SQLite + in-memory; 18 tests green.
- **Discovery + playground:** `GET /v1/models` (loaded engines, voices, capabilities) and a browser test UI at `/playground/` (voices, speed, format, streaming, voice enrollment).
- **Caching:** in-memory LRU audio cache (`X-Cache: HIT|MISS`, `TTS_CACHE_MAX`) keyed by request params — repeat requests skip synthesis. Measured on CPU: **5.0s → 9ms, byte-identical, $0 compute**. 21 tests green.
- **Model microservices (ProxyEngine):** models with conflicting deps run as separate services routed via a `ProxyEngine` (ADR-0008). Added a **Chatterbox** microservice (`services/chatterbox/`, its own venv: transformers 5.x/torch 2.6) registered as `model=chatterbox` via `TTS_CHATTERBOX_URL`. Proxy verified against a mock; 24 tests green.
