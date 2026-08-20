---
title: "nominal integer tick engine with smpte drop-frame timecode arithmetic"
type: pattern
tags: [pattern, frontend, media-time, timecode, smpte, drop-frame, branded-types]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127791371-cc78a949
sources: ["Harvest 1787127791371 Cc78a949.json"]
---

# nominal integer tick engine with smpte drop-frame timecode arithmetic

Nominal Integer Tick Engine with SMPTE Drop-Frame Timecode Arithmetic implements 120k tick/sec nominal branded types with NTSC drop-frame compensation for video timeline precision.

## 1. Overview & Architecture

A high-precision nominal integer media timing engine providing sub-frame tick calculations and standard SMPTE drop-frame timecode compensation without floating point drift.

## 2. Implementation & Code Structure

src/lib/media-time.ts -> Branded MediaTime nominal type, integer tick arithmetic, frame rounding, and SMPTE drop-frame timecode parser/formatter.

## 3. Key Implementation Points

- Zero-overhead branded nominal type pattern using unique symbol constraint.
- Pure integer math with assertions preventing fractional tick contamination.
- Comprehensive SMPTE timecode formatting and parsing supporting dynamic token mapping (HH, MM, SS, FF).

## 4. Code Examples

### Branded nominal integer tick engine with SMPTE drop-frame arithmetic and sub-frame precision

```typescript
export const TICKS_PER_SECOND = 120_000;

declare const MediaTimeBrand: unique symbol;
export type MediaTime = number & { [MediaTimeBrand]?: never };

export type FrameRate = {
  numerator: number;
  denominator: number;
};

export const ZERO_MEDIA_TIME = 0 as MediaTime;

export function mediaTime({ ticks }: { ticks: number }): MediaTime {
  if (!Number.isInteger(ticks)) {
    throw new Error(`mediaTime: expected integer ticks, got ${ticks}`);
  }
  return ticks as MediaTime;
}

export function mediaTimeFromSeconds({ seconds }: { seconds: number }): MediaTime {
  return mediaTime({ ticks: Math.round(seconds * TICKS_PER_SECOND) });
}

export function mediaTimeToSeconds({ time }: { time: MediaTime }): number {
  return time / TICKS_PER_SECOND;
}

export function roundFrameTime({ time, fps }: { time: MediaTime; fps: FrameRate }): MediaTime {
  const ticksPerFrame = (TICKS_PER_SECOND * fps.denominator) / fps.numerator;
  const rounded = Math.round(time / ticksPerFrame) * ticksPerFrame;
  return rounded as MediaTime;
}

function dropFrameCompensation(frameNumber: number, rate: FrameRate): number {
  const frameRate = Math.round(rate.numerator / rate.denominator);
  const dropsPerMinute = frameRate === 60 ? 4 : 2;
  const framesPerMinute = frameRate * 60;
  const framesPerTenMinutes = framesPerMinute * 10;

  const d = Math.floor(frameNumber / framesPerTenMinutes);
  const m = Math.floor((frameNumber % framesPerTenMinutes) / framesPerMinute);

  return d * dropsPerMinute * 9 + m * dropsPerMinute;
}

export function parseTimecode({ timeCode, format, rate }: { timeCode: string; format: string; rate: FrameRate }): number | null {
  // Regex builder supporting HH:MM:SS:FF formats with drop-frame compensation
  // ...
  return framesToTicks(totalF, rate);
}
```

## 5. Considerations & Best Practices

- TICKS_PER_SECOND = 120_000 is evenly divisible by standard frame rates (24, 25, 30, 48, 50, 60, 120 fps), eliminating floating-point rounding errors.
- Drop-frame compensation precisely accounts for NTSC 29.97 and 59.94 fps standards where 2 or 4 frame numbers are dropped per minute except every tenth minute.
- Branded nominal typing ensures raw floating-point numbers cannot accidentally be mixed with MediaTime ticks without explicit conversion.

## 6. Related Knowledge

- [[01-Knowledge/patterns/frontend/web-audio-api-hardware-buffer-graph-orchestrator.md]]
- [[01-Knowledge/patterns/frontend/sub-frame-dom-bypass-playhead-sync-loop.md]]

## 7. Source

- Harvest 1787127791371 Cc78a949.json
