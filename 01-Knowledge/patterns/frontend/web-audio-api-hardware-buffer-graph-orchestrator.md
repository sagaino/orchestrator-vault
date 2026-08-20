---
title: "Web Audio API Hardware Buffer Graph Orchestrator"
type: pattern
tags: [pattern, frontend, web-audio, audio-dsp, hardware-bridge, native-api]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127791371-cc78a949
sources: ["Harvest 1787127791371 Cc78a949.json"]
---

# Web Audio API Hardware Buffer Graph Orchestrator

Web Audio API Hardware Buffer Graph Orchestrator with Micro-Scrubbing Burst

## 1. Overview & Architecture

Manages the Web Audio API hardware audio graph lifecycle with in-memory decoded PCM buffer caching, user-gesture autoplay suspension recovery, and dynamic gain routing.

## 2. Implementation & Code Structure

Audio Context Singleton -> In-Memory Decoded Buffer Map -> Active Node Pool (Source + Gain) -> Routing to AudioContext.destination.

## 3. Key Implementation Points

- Autoplay policy resolution by auto-resuming suspended AudioContext on user action.
- Dynamic gain routing allowing instantaneous volume/mute toggles without resetting audio buffers.
- 300ms micro-burst scrubbing loop for low-latency interactive timeline preview.

## 4. Code Examples

### Web Audio API hardware graph manager with PCM decoding, gain routing, and micro-burst scrubbing

```typescript
let ctx: AudioContext | null = null;

function getContext(): AudioContext {
  if (!ctx) {
    ctx = new AudioContext();
  }
  if (ctx.state === "suspended") {
    ctx.resume();
  }
  return ctx;
}

const decodedBuffers = new Map<string, AudioBuffer>();
const activeSources = new Map<string, { source: AudioBufferSourceNode; gain: GainNode }>();

export async function decodeAudio(mediaId: string, file: File): Promise<void> {
  const c = getContext();
  const arrayBuffer = await file.arrayBuffer();
  const buffer = await c.decodeAudioData(arrayBuffer);
  decodedBuffers.set(mediaId, buffer);
}

export function playAudio({
  mediaId,
  offsetSec,
  muted,
  trimStartSec = 0,
}: {
  mediaId: string;
  offsetSec: number;
  muted: boolean;
  trimStartSec?: number;
}): void {
  const existing = activeSources.get(mediaId);
  if (existing) {
    existing.source.onended = null;
    try { existing.source.stop(); } catch { /* no-op */ }
  }

  const buffer = decodedBuffers.get(mediaId);
  if (!buffer) return;

  const c = getContext();
  const source = c.createBufferSource();
  source.buffer = buffer;

  const gain = c.createGain();
  gain.gain.value = muted ? 0 : 1;

  source.connect(gain);
  gain.connect(c.destination);

  const effectiveOffset = Math.max(0, offsetSec + trimStartSec);
  const clampedOffset = Math.max(0, Math.min(effectiveOffset, buffer.duration - 0.01));
  source.start(0, clampedOffset);

  source.onended = () => {
    activeSources.delete(mediaId);
  };
  activeSources.set(mediaId, { source, gain });
}

export function scrubAudio({ mediaId, offsetSec, muted, trimStartSec = 0 }: any): void {
  const buffer = decodedBuffers.get(mediaId);
  if (!buffer) return;

  const c = getContext();
  const source = c.createBufferSource();
  source.buffer = buffer;
  const gain = c.createGain();
  gain.gain.value = muted ? 0 : 1;

  source.connect(gain);
  gain.connect(c.destination);

  const effectiveOffset = Math.max(0, offsetSec + trimStartSec);
  const clampedOffset = Math.max(0, Math.min(effectiveOffset, buffer.duration - 0.01));
  source.start(0, clampedOffset);
  source.stop(c.currentTime + 0.3);

  source.onended = () => { scrubNode = null; };
  scrubNode = { source, gain };
}
```

## 5. Considerations & Best Practices

- AudioBufferSourceNodes are single-use; new nodes must be instantiated on playback restarts or seeks.
- Large uncompressed audio buffers consume memory; explicit disposeAudio() lifecycle cleanup is required.

## 6. Related Knowledge

- `Web Audio API node graphs (AudioBufferSourceNode, GainNode, AudioContext)`
- `Browser User-Activation Autoplay policies`

## 7. Source

- Harvest 1787127791371 Cc78a949.json
