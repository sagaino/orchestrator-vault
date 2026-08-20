---
title: "multi-source geometric snapping pipeline & magnetic gap collapse placement engine"
type: pattern
tags: [pattern, frontend, timeline, snapping, placement, collision-detection, media-editor]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127791371-cc78a949
sources: ["Harvest 1787127791371 Cc78a949.json"]
---

# multi-source geometric snapping pipeline & magnetic gap collapse placement engine

Multi-Source Geometric Snapping Pipeline & Magnetic Gap Collapse Placement Engine resolves timeline proximity snapping and enforces contiguous track placement.

## 1. Overview & Architecture

A geometric timeline positioning engine providing multi-source magnetic snapping, half-open interval collision validation, and primary track zero-anchor enforcement.

## 2. Implementation & Code Structure

src/pages/Editor/utils/snapping/* -> Snap point source aggregation and proximity resolver. src/pages/Editor/utils/placement/* -> Main track anchor enforcement, interval collision testing, and placement strategies.

## 3. Key Implementation Points

- Nearest-neighbor distance search within configurable threshold (maxSnapDistance).
- Functional composition of heterogeneous snap point sources (buildTimelineSnapPoints).
- Exclusion filters allowing moving/resizing elements to ignore their own historical positions during collision and snap calculations.

## 4. Code Examples

### Multi-source snapping resolver and magnetic gap collapse placement engine

```typescript
// 1. Snapping Pipeline (src/pages/Editor/utils/snapping/resolve.ts)
export function resolveTimelineSnap({
  targetTime,
  snapPoints,
  maxSnapDistance,
}: {
  targetTime: MediaTime;
  snapPoints: SnapPoint[];
  maxSnapDistance: number;
}): SnapResult {
  let closestSnapPoint: SnapPoint | null = null;
  let closestDistance = Infinity;

  for (const snapPoint of snapPoints) {
    const distance = Math.abs(targetTime - snapPoint.time);
    if (distance <= maxSnapDistance && distance < closestDistance) {
      closestDistance = distance;
      closestSnapPoint = snapPoint;
    }
  }

  return {
    snappedTime: closestSnapPoint ? closestSnapPoint.time : targetTime,
    snapPoint: closestSnapPoint,
    snapDistance: closestDistance,
  };
}

// 2. Main Track Zero Anchor Enforcement (src/pages/Editor/utils/placement/main-track.ts)
export function enforceMainTrackStart({
  track,
  requestedStartTime,
  excludeElementId,
}: {
  track: VideoTrack;
  requestedStartTime: MediaTime;
  excludeElementId?: string;
}): MediaTime {
  const earliestElement = getEarliestMainTrackElement({
    mainTrack: track,
    excludeElementId,
  });
  if (!earliestElement) return requestedStartTime;

  // Magnetic collapse: if element is placed before or at earliest element, snap to ZERO
  if (requestedStartTime <= earliestElement.startTime) {
    return ZERO_MEDIA_TIME;
  }
  return requestedStartTime;
}

// 3. Half-Open Interval Collision Check (src/pages/Editor/utils/placement/overlap.ts)
function wouldElementOverlap({
  elements,
  startTime,
  endTime,
  excludeElementId,
}: {
  elements: TimelineElement[];
  startTime: number;
  endTime: number;
  excludeElementId?: string;
}): boolean {
  return elements.some((element) => {
    if (excludeElementId && element.id === excludeElementId) return false;
    const elementEnd = element.startTime + element.duration;
    return startTime < elementEnd && endTime > element.startTime;
  });
}
```

## 5. Considerations & Best Practices

- Uses half-open temporal intervals [start, start + duration) allowing adjacent clips to touch perfectly without false collision positives.
- Zero-origin magnetic snapping enforces that the primary storytelling track never leaves an accidental leading gap.
- Snap point builder dynamically accepts any number of functional source generators (playhead, markers, other clips, grid ticks).

## 6. Related Knowledge

- [[01-Knowledge/patterns/frontend/nominal-integer-tick-engine-with-smpte-drop-frame-timecode-arithmetic.md]]
- [[01-Knowledge/patterns/frontend/sub-frame-dom-bypass-playhead-sync-loop.md]]

## 7. Source

- Harvest 1787127791371 Cc78a949.json
