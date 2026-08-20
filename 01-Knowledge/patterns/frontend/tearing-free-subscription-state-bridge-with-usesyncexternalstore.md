---
title: "Tearing-Free Subscription State Bridge with useSyncExternalStore"
type: pattern
tags: [pattern, frontend, react, usesyncexternalstore, state-bridge, concurrency]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127791371-cc78a949
sources: ["Harvest 1787127791371 Cc78a949.json"]
---

# Tearing-Free Subscription State Bridge with useSyncExternalStore

Tearing-Free Subscription State Bridge with useSyncExternalStore.

## 1. Overview & Architecture

React 18 useSyncExternalStore integration bridging an external Zustand store to guarantee tearing-free state reads and selective component re-renders.

## 2. Implementation & Code Structure

- src/editor/use-editor.ts: Custom useSyncExternalStore bridge hook
- src/editor/editor-store.ts: External Zustand store instance

## 3. Key Implementation Points

- Uses React 18 useSyncExternalStore for tearing-free concurrent rendering
- TypeScript function overloads provide exact type inference for slice selectors
- Conditional subscription optimization when selector is provided

## 4. Code Examples

### useSyncExternalStore wrapper hook for Zustand store

```typescript
import { useSyncExternalStore } from 'react'
import { useEditorStore } from './editor-store'
import type { EditorState } from './editor-store'

const subscribeNone = () => () => {}

export function useEditor(): EditorState
export function useEditor<T>(selector: (state: EditorState) => T): T
export function useEditor<T>(
  selector?: (state: EditorState) => T,
): EditorState | T {
  const subscribe = (onChange: () => void) =>
    useEditorStore.subscribe(onChange)

  const getSnapshot = (): EditorState | T => {
    const state = useEditorStore.getState()
    return selector ? selector(state) : state
  }

  return useSyncExternalStore(
    selector ? subscribe : subscribeNone,
    getSnapshot,
    getSnapshot,
  ) as EditorState | T
}
```

## 5. Considerations & Best Practices

- Selector functions must return stable or primitive values to prevent infinite render loops

## 6. Related Knowledge

- `react/use-sync-external-store`
- `state/fine-grained-reactivity`

## 7. Source

- Harvest 1787127791371 Cc78a949.json
