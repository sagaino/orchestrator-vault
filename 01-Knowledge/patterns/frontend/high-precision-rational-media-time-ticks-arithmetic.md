---
title: "High-Precision Rational Media Time & Ticks Arithmetic"
type: pattern
tags: [pattern, frontend, media-time, video-editor, branded-types, rational-arithmetic, timeline]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787111041173-fb05651c
sources: ["Harvest 1787111041173 Fb05651c.json"]
---

# High-Precision Rational Media Time & Ticks Arithmetic

Integer-ticks-based media time representation using TypeScript branded types to eliminate floating-point inaccuracies in audio/video timeline editing.

## 1. Overview & Architecture

A pure TypeScript rational time representation using branded integer ticks to eliminate floating-point drift and rounding errors across audio/video timeline calculations.

## 2. Implementation & Code Structure

src/lib/media-time.ts (Branded MediaTime type, constructor, arithmetic, conversions) -> src/lib/math.ts (Fractional frame math) -> src/editor/editor-store.ts (Timeline state consumption)

## 3. Key Implementation Points

- Use TypeScript branded type (number & { [MediaTimeBrand]?: never }) to prevent raw number assignment bugs.
- Define TICKS_PER_SECOND = 120,000 as a common multiple for 24, 25, 30, 48, and 60 FPS.
- Enforce integer assertions at constructor boundaries.
- Implement pure integer arithmetic functions (addMediaTime, subtractMediaTime, roundMediaTime).

## 4. Code Examples

### Branded MediaTime type with integer ticks arithmetic and second conversions

```typescript
export const TICKS_PER_SECOND = 120_000;

declare const MediaTimeBrand: unique symbol;
export type MediaTime = number & { [MediaTimeBrand]?: never };

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

export function addMediaTime({ a, b }: { a: MediaTime; b: MediaTime }): MediaTime {
  return (a + b) as MediaTime;
}
```

## 5. Considerations & Best Practices

- Ensure all arithmetic is performed with integer ticks before converting to seconds for UI display.
- Serializing to JSON should convert MediaTime to integer ticks or seconds based on contract.

## 6. Related Knowledge

- `Branded Types in TypeScript`
- `SMPTE Timecode Representation`
- `Fixed-Point Arithmetic`

## 7. Source

- Harvest 1787111041173 Fb05651c.json
