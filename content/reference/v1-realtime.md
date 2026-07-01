---
title: Reference — WS /v1/realtime (streaming)
type: reference
status: current
last-verified: 2026-07-01
tags: [reference, api, streaming]
---

# Reference — `WS /v1/realtime`

Low-latency streaming synthesis over WebSocket. The server splits text into sentences and
streams each sentence's audio as it's ready, so playback can start on the first sentence
(low time-to-first-audio). See ADR-0003.

## Protocol

Client sends one JSON message:
```json
{"model": "kokoro", "input": "text to speak", "voice": "af_heart", "language": "hi", "speed": 1.0}
```

Server responds:
```
{"type":"start","sample_rate":24000,"format":"pcm_s16le_mono","chunks":N}
<binary PCM frame>   x N      (16-bit little-endian, mono, @ sample_rate)
{"type":"done"}
```

On failure: `{"type":"error","message":"..."}`.

## Notes

- Works with **any** engine (`kokoro`, `indic`, …) — streaming is per-sentence `synthesize`.
- `chunks` tells the client how many binary frames to expect.
- Audio is raw PCM; play each frame as it arrives, or concatenate at `sample_rate`.

## Example (Python)

```python
import asyncio, json, websockets

async def main():
    async with websockets.connect("ws://localhost:8000/v1/realtime") as ws:
        await ws.send(json.dumps({"model": "kokoro", "input": "Hello. World.", "voice": "af_heart"}))
        start = json.loads(await ws.recv())
        pcm = b"".join([await ws.recv() for _ in range(start["chunks"])])
        # ... play `pcm` at start["sample_rate"] ...

asyncio.run(main())
```

## Related

- [POST /v1/audio/speech](v1-audio-speech.md) (one-shot)
- [ADR-0003: real-time first](../adr/0003-realtime-first-architecture.md)
- Source: `app/api/realtime.py`, `app/streaming/chunker.py`
