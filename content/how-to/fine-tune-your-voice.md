---
title: How to fine-tune your own voice (Chatterbox + LoRA)
tags: [how-to, fine-tuning, models]
---

# How to fine-tune your own voice

Turn the open, MIT-licensed **Chatterbox** base into a model that speaks in *your* voice,
then run it self-hosted behind the service's `TTSEngine`. See [ADR-0006](../adr/0006-finetune-open-base.md).

Cost: **$0** on free Colab/Kaggle (or **< $1** on a rented GPU). GPU is required for
training — this does **not** run on the CPU dev machine; use Colab/Kaggle.

## Step 0 — Validate zero-shot first (0 GPU-hours)

Chatterbox can clone a voice from a ~10 s clip *without any training*. Do this before
recording hours of audio:

1. Record one clean **10 s** clip of your voice.
2. In a Chatterbox Colab, run zero-shot synthesis using that clip as the reference.
3. Listen. If it already sounds like you → you may not need to fine-tune at all. If timbre
   or prosody is off → proceed to a LoRA fine-tune below.

## Step 1 — Record your dataset

**Target: 1–2 hours of clean, consistent speech** (≈ 1,000–1,500 clips of 3–10 s).
Chatterbox trains from 30 min, but 1–2 h takes it from "recognizable" to "good."
**Cleanliness and prosodic variety matter more than raw hours.**

Recording spec:
- One mic, one quiet room, one session style; fixed distance + pop filter.
- Record **48 kHz**, then resample; **mono, 16-bit PCM WAV**; consistent volume + pace.
- What to read: **CMU ARCTIC** (~1,132 phonetically balanced sentences ≈ 1 h) + **Harvard
  Sentences**, then ~10–15 min of natural sentences with questions, numbers, and your
  product's vocabulary (so prosody isn't monotone).

> [!WARNING]
> Background noise/reverb is the #1 quality killer — the model learns it as "your voice."
> Record in the quietest space you have and keep every clip consistent.

## Step 2 — Prepare the dataset (record → clean → segment → transcribe → manifest)

1. Record long takes (10–20 min each).
2. **Clean the full take first** (before chunking): denoise/de-reverb (Audacity / Ultimate
   Vocal Remover), then loudness-normalize every file.
3. **Segment** into 3–10 s clips on natural pauses (pydub silence-split: `min_silence_len≈300ms`,
   `silence_thresh≈-45dBFS`, `keep_silence≈200ms`). Drop clips < 2 s or > 15 s.
4. **Transcribe** each clip with **WhisperX**; then clean text (expand numbers/acronyms to
   words, standardize punctuation, drop empties).
5. **Conform audio** to one consistent sample rate (24 kHz is safe), mono, 16-bit.
6. Write the **LJSpeech manifest** `metadata.csv` (pipe-delimited, UTF-8):
   ```
   wavs/clip_0001.wav|Hello and welcome.|Hello and welcome.
   ```
   Folder layout: `my_voice/wavs/*.wav` + `my_voice/metadata.csv`.
7. **Split** 95% train / 5% val — no clips from the same source take in both splits.

**Shortcut:** `gokhaneraslan/tts-dataset-generator` does steps 3–6 in one pass and pairs
with the Chatterbox trainer from the same author.

## Step 3 — Fine-tune (LoRA) on Colab/Kaggle

- **LoRA**, not full fine-tune (fits a free T4, resists overfitting on small data).
- Start config: **rank 16, alpha 32–64, dropout 0.05, AdamW lr 1e-4, 3–5 epochs**.
- Trainer: `gokhaneraslan/chatterbox-finetuning` (LJSpeech ingestion + LoRA + `merge_lora.py`).
- Prefer **Kaggle** (30 GPU-hrs/week, 12 h sessions) for the full run; Colab for a fast first pass.
- Expect **~1–3 h** wall-clock. Watch val loss + listen to sample outputs; stop when samples
  stop improving.

## Step 4 — Merge, download, and wrap as an engine

1. Run `merge_lora.py` → a standalone `voice_chatterbox.safetensors` (+ config/tokenizer).
   Keep one clean 6–10 s reference clip of your voice with it.
2. Download to `tts-service/models/chatterbox_myvoice/` (git-ignored).
3. Add `app/engine/chatterbox.py` — a `ChatterboxEngine(TTSEngine)` mirroring `KokoroEngine`
   (lazy import, load model once, emit 16-bit LE mono PCM, `supports_cloning = True`), and
   register it in the engine registry. No route or client changes needed.

## Related

- [Decision: fine-tune an open base](../adr/0006-finetune-open-base.md)
- [How codec-LM TTS works](../explanation/how-codec-lm-tts-works.md)
- [API reference](../reference/v1-audio-speech.md)
