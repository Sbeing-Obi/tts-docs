---
title: "ADR-0008: Run conflicting models as separate services (ProxyEngine)"
tags: [adr, architecture, models]
status: accepted
date: 2026-07-02
---

# ADR-0008: Run conflicting models as separate services (ProxyEngine)

## Status

Accepted

## Context

Different ML models often have **mutually incompatible dependencies**. Concretely,
**Chatterbox** needs `transformers 5.x` + `torch 2.6`, while **IndicF5** needs
`transformers 4.49` (`<4.50`) + `torch 2.2`. Installing both in one venv is impossible —
and they run in the same Python process (the in-process engine registry). This generalises:
any two models with conflicting pins can't share a process.

## Decision

Run each such model as its **own microservice** (own venv / process) and route to it from
the main API via a **`ProxyEngine`** — a `TTSEngine` that forwards a request over HTTP and
returns the PCM it gets back. Cloned `voice` ids are resolved to a reference clip via the
shared voice store, so cloning still works across the boundary.

- Chatterbox lives in `services/chatterbox/` (its own venv + a tiny FastAPI server).
- The main service registers it as `model=chatterbox` when `TTS_CHATTERBOX_URL` is set.
- Clients and the `/v1/audio/speech` contract are unchanged — it's just another engine.

## Consequences

- Good: models with incompatible deps coexist; each can be scaled/deployed independently (e.g. its own container on its own GPU); no API/client changes.
- Good: the pattern is reusable for *any* future conflicting model.
- Cost: an extra process to run and monitor; cross-process latency; the proxy assumes a **shared filesystem** for reference clips (true locally / with a shared volume). For a fully remote model service, pass the audio bytes in the request instead of a path.
