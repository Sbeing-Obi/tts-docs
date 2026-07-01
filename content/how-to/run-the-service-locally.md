---
title: How to run the service locally
tags: [how-to, dev]
---

# How to run the service locally

Get the TTS API running on your machine (CPU, no GPU) and generate your first audio.

## Prerequisites

- **Python 3.11** (the project uses a 3.11 venv for model/library compatibility)

## Steps

1. Create + activate a venv and install (app + dev + models):
   ```powershell
   cd tts-service
   py -3.11 -m venv .venv
   .venv\Scripts\Activate.ps1
   pip install -e ".[dev,models]"
   ```

2. Download the Kokoro model files into `models/` (once, ~115 MB):
   ```powershell
   mkdir models -Force
   $rel = "https://github.com/thewh1teagle/kokoro-onnx/releases/download/model-files-v1.0"
   curl.exe -L "$rel/kokoro-v1.0.int8.onnx" -o models/kokoro-v1.0.int8.onnx
   curl.exe -L "$rel/voices-v1.0.bin"        -o models/voices-v1.0.bin
   ```
   (Override locations with `KOKORO_MODEL_PATH` / `KOKORO_VOICES_PATH`.)

3. Run the API:
   ```powershell
   uvicorn app.main:app --reload
   ```

4. Generate audio:
   ```powershell
   curl http://127.0.0.1:8000/v1/audio/speech -H "Content-Type: application/json" -d '{\"model\":\"kokoro\",\"input\":\"Hello world\",\"voice\":\"af_heart\"}' --output hello.wav
   ```
   Open `hello.wav` to listen. Interactive API docs live at http://127.0.0.1:8000/docs.

5. Run the tests:
   ```powershell
   pytest
   ```

> [!NOTE]
> First synthesis on CPU is slow (int8 model, ~2× real-time) and includes a one-time
> warmup. Fine for one-shot use; low-latency streaming is a later phase.

## Related

- [API reference: POST /v1/audio/speech](../reference/v1-audio-speech.md)
