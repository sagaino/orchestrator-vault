---
title: "Centralized Domain Store with Asynchronous Storage Sidecar Sync"
type: pattern
tags: [pattern, frontend, state, zustand, reactive, sidecar-sync]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127791371-cc78a949
sources: ["Harvest 1787127791371 Cc78a949.json"]
---

# Centralized Domain Store with Asynchronous Storage Sidecar Sync

Centralized Zustand domain store with asynchronous storage sidecar synchronization.

## 1. Overview & Architecture

Zustand domain store managing complex project tree models while orchestrating asynchronous persistence to underlying storage engines in the background.

## 2. Implementation & Code Structure

- src/editor/editor-store.ts: Central Zustand store with state, actions, and sidecar sync
- src/services/storage/services.ts: Multi-tier storage persistence service
- src/types/project.ts & src/types/scene.ts: Domain model types

## 3. Key Implementation Points

- In-memory state updated immutably and rendered immediately (optimistic UI)
- Persistence calls to storageService executed asynchronously without blocking the UI thread
- Automatic cleanup of orphaned media assets when elements are deleted

## 4. Code Examples

### Zustand store with async storage sidecar synchronization

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
      for (const el of result.project.scenes.tracks.elements) {
        if ('mediaId' in el && el.mediaId) {
          const asset = await storageService.loadMediaAsset({
            projectId: id,
            id: (el as { mediaId: string }).mediaId,
          });
          if (asset?.url) urls[el.id] = asset.url;
        }
      }
      set({ project: result.project, isLoading: false, mediaUrls: urls });
    } catch (error) {
      set({ isLoading: false, error: error instanceof Error ? error.message : String(error) });
    }
  },

  updateScene: (patch) => {
    const { project } = get();
    if (!project) return;
    const updated: TProject = {
      ...project,
      scenes: { ...project.scenes, ...patch, updatedAt: new Date() },
    };
    set({ project: updated });
    storageService.saveProject({ project: updated });
  },
}));
```

## 5. Considerations & Best Practices

- High-frequency mutations (e.g. playhead scrub) should be debounced before storage persist
- Object URLs created during asset loading must be revoked on cleanup

## 6. Related Knowledge

- `state/zustand-async-persistence`
- `storage/opfs-indexeddb-coordination`

## 7. Source

- Harvest 1787127791371 Cc78a949.json
