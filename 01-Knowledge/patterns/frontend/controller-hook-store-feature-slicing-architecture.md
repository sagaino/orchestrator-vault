---
title: "Controller-Hook-Store Feature Slicing Architecture"
type: pattern
tags: [pattern, frontend, architecture, modularity, zustand, react, feature-sliced]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787107893306-92ee562e
sources: ["Harvest 1787107893306 92ee562e.json"]
---

# Controller-Hook-Store Feature Slicing Architecture

Pemisahan logika interaksi kompleks kanvas/timeline ke dalam Controller (pure logic), Hooks (event binding), dan Store (Zustand state).

## 1. Overview & Architecture

Pola pemisahan fungsionalitas fitur kanvas/editor yang kompleks ke dalam modul terpisah: Controller (pure business & math logic), Hooks (event binding & lifecycle), Store (state management), dan Components (presentasional).

## 2. Implementation & Code Structure

src/pages/Editor/
├── components/ (Presentational React components: timeline, ruler, tracks)
├── controller/ (Pure math/business logic: seek, zoom, resize, snap)
├── hooks/ (Event bindings, gesture listeners, canvas observers)
├── store/ (Feature-level Zustand stores)
├── types/ (Domain data structures & interfaces)
└── utils/ (Creation & audio management helpers)

## 3. Key Implementation Points

- Separation of Concerns: Komputasi matematis dipisahkan ke modul controller murni tanpa dependensi JSX/React.
- Kustomisasi Hooks untuk isolasi event listener DOM, mouse drag, dan keyboard shortcuts.
- Zustand State Store mengelola global feature state dan menyinkronkan perubahan ke service storage secara terintegrasi.

## 4. Code Examples

### Zustand Feature Store dengan Auto-Persistence Side-Effects

```typescript
export const useEditorStore = create<EditorState>((set, get) => ({
  project: null,
  isLoading: true,
  error: null,
  selectedElementIds: [],
  mediaUrls: {},
  isPlaying: false,

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

  deleteSelectedElements: () => {
    const { project, selectedElementIds } = get();
    if (!project || selectedElementIds.length === 0) return;
    const selectedSet = new Set(selectedElementIds);
    const elements = project.scenes.tracks.elements.filter(
      (el) => !selectedSet.has(el.id),
    );
    const updated: TProject = {
      ...project,
      scenes: {
        ...project.scenes,
        tracks: { ...project.scenes.tracks, elements },
        updatedAt: new Date(),
      },
    };
    set({ project: updated, selectedElementIds: [] });
    storageService.saveProject({ project: updated });
  },
}));
```

## 5. Considerations & Best Practices

- Operasi penyimpanan otomatis (auto-save) pada store sebaiknya di-throttle/debounce pada perubahan intensitas tinggi (seperti dragging timeline).
- Pertahankan immutable state updates di dalam Zustand untuk mencegah bug re-rendering.

## 6. Related Knowledge

- `Zustand State Management`
- `Feature-Sliced Design`
- `Separation of Concerns`

## 7. Source

- Harvest 1787107893306 92ee562e.json
