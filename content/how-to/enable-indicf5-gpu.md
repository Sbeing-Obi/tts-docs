---
title: How to enable IndicF5 (GPU)
tags: [how-to, gpu, indic, models]
---

# How to enable IndicF5 (GPU)

IndicF5 makes your voice speak 11 Indian languages, but it needs a **GPU**. The service runs
on Kokoro (CPU/English) by default; enable IndicF5 on a GPU host (Colab, rented, or cloud).

## On a GPU machine

1. Accept the gated model + log in:
   - Accept access at https://huggingface.co/ai4bharat/IndicF5
   - `huggingface-cli login` (paste a Read token)
2. Install the pinned GPU deps (transformers **must** be < 4.50):
   ```bash
   pip install -e .
   pip install -r requirements-indic.txt
   ```
3. Enable + run:
   ```bash
   TTS_ENABLE_INDICF5=1 uvicorn app.main:app
   ```
   This registers engines `indicf5` and `indic` (the language router) next to `kokoro`.

## Use it

1. Enroll your voice → get a `voice_id` (see the [voices reference](../reference/v1-voices.md)).
2. Synthesize Hindi in your voice:
   ```bash
   curl http://localhost:8000/v1/audio/speech -H "Content-Type: application/json" \
     -d '{"model":"indic","language":"hi","voice":"<voice_id>","input":"नमस्ते"}' --output hi.wav
   ```

Routing: `model=indic` picks IndicF5 for its 11 languages (`hi, bn, te, mr, ta, gu, kn, ml, pa, or, as`) and falls back to Kokoro otherwise.

> [!NOTE]
> No GPU handy? Validate first on Colab with `notebooks/indicf5_zeroshot.ipynb`.

## Running locally on Windows CPU (demo only)

IndicF5 also runs on Windows CPU for a quick look — no GPU needed:

1. `pip install -r requirements-indic.txt`, then `huggingface-cli login` (paste a Read token).
2. `TTS_ENABLE_INDICF5=1 uvicorn app.main:app`

The engine auto-patches `torch.compile` (unsupported on Windows) with a thin eager wrapper
that preserves the checkpoint's `_orig_mod.` prefix, so weights load correctly.

> [!WARNING]
> On CPU this is **very slow — ~5 minutes per sentence** (RTF ~60). Use it only to *see*
> cloning work; the response cache makes repeats instant. For any real/real-time use, run on
> a GPU (Linux), where torch.compile works and it's orders of magnitude faster.

## Related

- [Voice enrollment reference](../reference/v1-voices.md)
- [ADR-0007: Indic multilingual stack](../adr/0007-indic-multilingual-stack.md)
