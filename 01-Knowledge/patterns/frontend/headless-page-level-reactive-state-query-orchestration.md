---
title: "Headless Page-Level Reactive State & Query Orchestration"
type: pattern
tags: [pattern, frontend, react, tanstack-query, custom-hooks, state-management]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787112360086-3c0f5be1
sources: ["Harvest 1787112360086 3c0f5be1.json"]
---

# Headless Page-Level Reactive State & Query Orchestration

Headless page-level state management pattern integrating TanStack Query, filter synchronization, memoized transformations, and error boundary suppression.

## 1. Overview & Architecture

Decoupled page architecture where all asynchronous data fetching, mutation lifecycles, filter parameters, and view transformations are managed inside headless custom hooks.

## 2. Implementation & Code Structure

src/renderer/src/pages/<Feature>/
├── index.tsx                     # Declarative view container
├── components/                   # Pure UI presentation components
├── hooks/
│   └── use<Feature>State.ts     # Headless query, mutation, filter, and pagination coordinator
└── data/
    └── index.ts                  # Local UI view model types

## 3. Key Implementation Points

- Encapsulates all TanStack Query logic (`useQuery`, `useMutation`, `useQueryClient`) in a dedicated feature hook.
- Synchronizes multi-parameter filter state with automatic reset to page 1 on search/filter updates.
- Implements background polling (`refetchInterval`) for near-real-time dashboard status.
- Transforms raw backend DTO payloads into memoized UI presentation models (`paginatedData`).

## 4. Code Examples

### Headless page state hook with TanStack Query, filter derivation, polling, and memoized transformation

```typescript
// src/renderer/src/pages/Device/hooks/useDeviceManagementState.ts
export const useDeviceManagementState = () => {
  const queryClient = useQueryClient()
  const [filter, setFilter] = useState<IDeviceManagementFilter>({
    page: 1,
    size: 10,
    search: '',
    status: '',
    available: '',
    divisionCode: '',
  })

  const { data, isLoading, error } = useQuery({
    queryKey: ['device-list', filter.page, filter.size, filter.search, filter.status, filter.available, filter.divisionCode],
    queryFn: () =>
      DeviceService.page({
        page: filter.page,
        size: filter.size,
        search: filter.search,
        status: filter.status,
        available: filter.available,
        order: 'desc',
        sortBy: 'id',
        divisionCode: filter.divisionCode,
      }),
    refetchOnMount: 'always',
    refetchInterval: 10000,
    refetchOnWindowFocus: true,
  })

  useEffect(() => {
    if (!error) return
    if (error instanceof IpcServiceError && error.statusCode === 401) return
    toast.error(error instanceof Error ? error.message : 'Gagal mengambil data perangkat')
  }, [error])

  const paginatedData = useMemo(() => {
    const values = data?.result?.values ?? []
    return values.map((item) => ({
      code: item.code,
      hwid: item.hwid,
      serialNumber: item.serial_number,
      status: item.is_active ? 'active' : 'inactive',
    }))
  }, [data?.result?.values])

  return {
    filter,
    isLoading,
    paginatedData,
    setPage: (page: number) => setFilter(prev => ({ ...prev, page: Math.max(page, 1) })),
    setSearch: (search: string) => setFilter(prev => ({ ...prev, search, page: 1 })),
  }
}
```

## 5. Considerations & Best Practices

- Keep query keys fully exhaustive with all active filter variables to guarantee proper cache invalidation on filter change.
- Always ignore 401 errors in page-level error `useEffect` since they are globally handled by the IPC auth interceptor.
- Transform server DTOs into clean view models via `useMemo` to shield UI presentation components from backend schema changes.

## 6. Related Knowledge

- Headless State Management
- Tanstack Query Patterns

## 7. Source

- Harvest 1787112360086 3c0f5be1.json
