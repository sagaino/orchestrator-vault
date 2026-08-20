---
title: "Non-Linear Timeline & Multi-Track Editor State Management"
type: pattern
tags: [pattern, frontend, state-management, zustand, video-editor, timeline, react]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787111041173-fb05651c
sources: ["Harvest 1787111041173 Fb05651c.json"]
---

# Non-Linear Timeline & Multi-Track Editor State Management

Hierarchical Zustand-based state store for multi-track non-linear media editing with asynchronous blob hydration, multi-selection, and debounced persistence.

## 1. Overview & Architecture

A centralized Zustand store managing multi-track scene hierarchies, multi-element selection, playback state, and dynamic asset URL hydration.

## 2. Implementation & Code Structure

src/editor/editor-store.ts (Zustand state store and actions) -> src/editor/use-editor.ts (Hook interface) -> src/pages/Editor/* (UI canvas and timeline views)

## 3. Key Implementation Points

- Separate persistent project hierarchy from transient runtime UI state (mediaUrls, isPlaying).
- Asynchronously hydrate media asset IDs into active memory Object URLs on project load.
- Provide granular actions for element additions, deletions, track muting, and selection.
- Centralize timeline view state (zoomLevel, scrollLeft, playheadTime).

## 4. Code Examples

### Zustand store handling project loading, media URL hydration, and timeline state

```typescript
export const useEditorStore = create<EditorState>((set, get) => ({
  project: null,
  isLoading: true,
  error: null,
  selectedElementIds: [],
  mediaUrls: {},
  isPlaying: false,

  loadProject: async (id) => {
    set({ isLoading: true, error: null });
    try {
      const result = await storageService.loadProject({ id });
      if (!result) {
        set({ isLoading: false, error: `Project "${id}" not found` });
        return;
      }
      const urls: Record<string, string> = {};
      const elements = result.project.scenes.tracks.elements;
      for (const el of elements) {
        if ('mediaId' in el && el.mediaId) {
          try {
            const asset = await storageService.loadMediaAsset({
              projectId: result.project.metadata.id,
              id: el.mediaId,
            });
            if (asset?.url) urls[el.mediaId] = asset.url;
          } catch {}
        }
      }
      set({ project: result.project, mediaUrls: urls, isLoading: false });
    } catch (err) {
      set({ isLoading: false, error: (err as Error).message });
    }
  },
}));
```

## 5. Considerations & Best Practices

- Revoke existing object URLs when loading a different project or deleting elements.
- Keep scene hierarchy modifications immutable for predictable rendering.

## 6. Related Knowledge

- `Zustand State Architecture`
- `Non-Linear Editor (NLE) Data Structures`
- `Object URL Hydration`

## 7. Source

- Harvest 1787111041173 Fb05651c.json
