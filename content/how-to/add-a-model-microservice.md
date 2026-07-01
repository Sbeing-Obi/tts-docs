---
title: How to add a model as a separate microservice
tags: [how-to, models, architecture]
---

# How to add a model as a separate microservice

Use this when a model's dependencies conflict with the main service's (e.g. Chatterbox needs
`transformers 5.x`/`torch 2.6`, which clash with IndicF5's pins). See [ADR-0008](../adr/0008-model-microservices-proxy.md).

## The pattern

1. The model runs in its **own venv + process**, exposing `POST /synthesize` that returns
   raw 16-bit PCM (with an `X-Sample-Rate` header).
2. The main service registers a **`ProxyEngine`** (`app/engine/proxy.py`) that forwards to it
   and resolves cloned voice ids via the shared voice store.

## Chatterbox (worked example)

Code lives in `services/chatterbox/`. To run it:

```powershell
cd services\chatterbox
py -3.11 -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt        # torch 2.6, transformers 5.x, ~2.5 GB
uvicorn server:app --port 8100         # loads Chatterbox on startup
```

Then point the main service at it and restart:

```powershell
$env:TTS_CHATTERBOX_URL = "http://127.0.0.1:8100"
uvicorn app.main:app
```

`model=chatterbox` now shows up in `/v1/models` and the playground. Enrolled voices work
(the reference clip's path is passed through; both processes share the filesystem).

> [!NOTE]
> Chatterbox is ~0.5B — GPU recommended; slow on CPU and may not fit a 4 GB card. It's best
> for English/multilingual cloning; IndicF5 remains the choice for Indian languages.

## Adding another conflicting model

Copy `services/chatterbox/` as a template, swap the model loading + `/synthesize` body, give
it a config URL (like `TTS_CHATTERBOX_URL`), and register another `ProxyEngine` in
`app/main.py`. No API or client changes.

## Related

- [ADR-0008: model microservices + ProxyEngine](../adr/0008-model-microservices-proxy.md)
- Source: `app/engine/proxy.py`, `services/chatterbox/`
