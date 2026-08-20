---
title: "Sub-Frame DOM Bypass Playhead Sync Loop"
type: pattern
tags: [pattern, frontend, hardware-timing, raf-loop, dom-bypass, audio-bridge, timeline]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127791371-cc78a949
sources: ["Harvest 1787127791371 Cc78a949.json"]
---

# Sub-Frame DOM Bypass Playhead Sync Loop

Sub-Frame DOM Bypass Playhead Sync Loop with Dual-Viewport Auto-Scroll & Audio Bridge

## 1. Overview & Architecture

Decouples high-frequency 60fps timeline playhead animation from React component reconciliation using direct DOM mutations while synchronizing dual scroll viewports and Web Audio hardware nodes.

## 2. Implementation & Code Structure

usePlayback Hook -> High-Precision Clock (performance.now) -> Direct DOM style injection -> Audio node intersection diff -> Dual-viewport scroll sync.

## 3. Key Implementation Points

- React Virtual DOM bypass via direct style.left DOM manipulation inside 60fps rAF tick.
- Dual-viewport synchronization keeping tracks and ruler scroll containers in lockstep.
- Active audio channel diffing dynamically mounting/stopping Web Audio nodes on timeline intersection.

## 4. Code Examples

### High-performance requestAnimationFrame playhead sync loop with direct DOM mutation and audio intersection diffing

```typescript
useEffect(() => {
  if (!isPlaying) return;

  resumeAudioContext();
  const wallStartRef = { current: performance.now() };
  const playheadStartRef = { current: playheadTimeRef.current };
  let rafId: number;

  const tick = () => {
    if (!isPlayingRef.current) return;

    const elapsedMs = performance.now() - wallStartRef.current;
    const elapsed = mediaTimeFromSeconds({ seconds: elapsedMs / 1000 });
    const nextTime = addMediaTime({ a: playheadStartRef.current, b: elapsed });

    if (nextTime >= effectiveDuration) {
      seekRef.current(effectiveDuration);
      pauseRef.current();
      return;
    }

    const tracksEl = tracksScrollRef.current;
    const scrollLeft = tracksEl?.scrollLeft ?? 0;
    const centerPixel = timelineTimeToPixels({
      time: nextTime,
      zoomLevel: zoomLevelRef.current,
    });
    const playheadLeft = getCenteredLineLeft({ centerPixel }) - scrollLeft;

    if (playheadRef.current) {
      playheadRef.current.style.left = `${playheadLeft}px`;
    }

    const playheadSec = mediaTimeToSeconds({ time: nextTime });
    const nowPlaying = new Set<string>();

    for (const ael of audioElementsRef.current) {
      const startSec = mediaTimeToSeconds({ time: ael.startTime });
      const endSec = startSec + mediaTimeToSeconds({ time: ael.duration });
      if (playheadSec >= startSec && playheadSec < endSec) {
        nowPlaying.add(ael.mediaId);
        if (!playingMediaIds.current.has(ael.mediaId)) {
          playAudio({
            mediaId: ael.mediaId,
            offsetSec: playheadSec - startSec,
            muted: isAudioMutedRef.current,
            trimStartSec: mediaTimeToSeconds({ time: ael.trimStart }),
          });
        }
      }
    }

    for (const id of playingMediaIds.current) {
      if (!nowPlaying.has(id)) stopAudio(id);
    }
    playingMediaIds.current = nowPlaying;

    rafId = requestAnimationFrame(tick);
  };

  rafId = requestAnimationFrame(tick);
  return () => {
    cancelAnimationFrame(rafId);
    seekRef.current(currentTimeRef.current);
    stopAllAudio();
  };
}, [isPlaying, effectiveDuration]);
```

## 5. Considerations & Best Practices

- Zoom and layout state updates during playback must update mutable refs immediately to avoid jitter.
- React state commit must occur on teardown to persist exact stopping position.

## 6. Related Knowledge

- `High-precision timer (performance.now) & requestAnimationFrame timing loop`
- `Direct DOM mutations for high-frequency UI updates in React`

## 7. Source

- Harvest 1787127791371 Cc78a949.json
