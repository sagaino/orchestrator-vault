---
title: "Service-Repository & Query Key Factory Architecture for Server State Flow"
type: pattern
tags: [pattern, frontend, react-query, state-management, service-layer, axios, api]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787132260416-42b894f9
sources: ["Harvest 1787132260416 42b894f9.json"]
---

# Service-Repository & Query Key Factory Architecture for Server State Flow

Service-Repository & Query Key Factory Architecture for Server State Flow

## 1. Overview & Architecture

Arsitektur 3-lapis pengelolaan Server State: (1) HTTP Client & Request Signing Interceptors, (2) Pure Type-Safe Service Repositories, dan (3) TanStack React Query Hooks dengan Query Key Factory terstandarisasi.

## 2. Implementation & Code Structure

src/
├── config/
│   ├── axios.ts                  # Axios instance with Signature & Auth Interceptors
│   └── constant/endpoint.ts      # REST API endpoint mapping
├── services/
│   └── doorAccessServices.ts     # Service layer functions returning typed DTOs
└── pages/DoorAccess/
    ├── hooks/
    │   └── useAccessCardQueries.ts  # TanStack Query & Mutation hooks with Query Key Factory
    └── types/
        └── api.ts                # Request and response contract interfaces

## 3. Key Implementation Points

- Deterministic Query Key Factory using TypeScript `as const` tuple arrays.
- Separation of API transport (Services) from UI cache management (React Query hooks).
- Unified error handling and automatic query invalidation upon successful mutation.

## 4. Code Examples

### Axios request interceptor dynamically calculating request signature and injecting auth headers.

```typescript
// src/config/axios.ts (HMAC Request Signature & Interceptor)
axios.interceptors.request.use(async (config) => {
  const token = getData(LOCALSTORAGE_KEY.TOKEN);
  const timestamp = getTimestamp();

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  const signature = makeSignature(
    0,
    timestamp,
    config.data || {},
    config.data instanceof FormData
  );

  config.headers["sig"] = signature;
  config.headers["timestamp"] = timestamp.toString();
  return config;
});
```

### Typed API Service Layer encapsulating backend REST endpoints.

```typescript
// src/services/doorAccessServices.ts
export const DoorAccessServices = {
  getAccessCards: async (params: RequestAccessCardList): Promise<ResponseAccessCard> => {
    const response = await axios.get<ResponseAccessCard>(
      ENDPOINTS.DOOR_ACCESS.ACCESS_CARD_LIST,
      { params }
    );
    return response.data;
  },
  createAccessCard: async (data: RequestAccessCardCreate): Promise<ResponseAccessCardCreate> => {
    const response = await axios.post<ResponseAccessCardCreate>(
      ENDPOINTS.DOOR_ACCESS.ACCESS_CARD_CREATE,
      data
    );
    return response.data;
  },
};
```

### Query Key Factory pattern and reactive mutation hooks managing server state.

```typescript
// src/pages/DoorAccess/hooks/useAccessCardQueries.ts
export const ACCESS_CARD_QUERY_KEYS = {
  all: ['access-cards'] as const,
  lists: () => [...ACCESS_CARD_QUERY_KEYS.all, 'list'] as const,
  list: (params: RequestAccessCardList) => [...ACCESS_CARD_QUERY_KEYS.lists(), params] as const,
  details: () => [...ACCESS_CARD_QUERY_KEYS.all, 'detail'] as const,
  detail: (code: string) => [...ACCESS_CARD_QUERY_KEYS.details(), code] as const,
};

export function useAccessCards(params: RequestAccessCardList) {
  return useQuery({
    queryKey: ACCESS_CARD_QUERY_KEYS.list(params),
    queryFn: () => DoorAccessServices.getAccessCards(params),
    staleTime: 5 * 60 * 1000,
  });
}

export function useCreateAccessCard() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: RequestAccessCardCreate) => DoorAccessServices.createAccessCard(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ACCESS_CARD_QUERY_KEYS.lists() });
      toast.success('Access card created successfully');
    },
    onError: (error: any) => {
      toast.error(error.response?.data?.message || 'Failed to create access card');
    },
  });
}
```

## 5. Considerations & Best Practices

- Hierarchical query keys prevent accidental cache invalidation across unrelated query scopes.
- Centralized API service isolation simplifies unit testing and mocking without component involvement.
- Automatic toast feedback within mutation lifecycle reduces boilerplate in UI view layers.

## 6. Related Knowledge

- `TanStack React Query v5 Best Practices`
- `Query Key Factory Pattern`
- `Repository & Service Pattern in Frontend`

## 7. Source

- Harvest 1787132260416 42b894f9.json
