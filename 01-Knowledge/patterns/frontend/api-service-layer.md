---
title: Summary of API & Service Layer Rules
type: pattern
tags: [summary, api, axios, service-layer]
created: 2026-08-12
updated: 2026-08-14
sources: ["[[03-Sources/documentation/rules-api.md]]"]
---

# Summary: API & Service Layer Rules

## Key Takeaways
- **Mandatory Flow**: `UI → Custom Hook → Service Layer → Axios Client → Backend API`.
- **Service Layer (`src/services/`)**: Pure typed API calls only; no JSX, toast, navigation, or UI state.
- **Central Endpoints (`src/lib/constant/endpoints.ts`)**: All URLs centralized; no hardcoded endpoint strings.
- **Axios & Auth**: Centralized `useLocalStorage` for auth; no direct `localStorage` or manual Bearer headers.
- **Error Normalization**: All API errors processed through `src/lib/error-utils.ts`.

## Related Pages
- [[01-Knowledge/concepts/architecture/api-service-data-flow|API Service Data Flow]]
- [[01-Knowledge/concepts/api/axios-client|Axios Client & Services]]
- [[01-Knowledge/concepts/architecture/frontend-architecture|Frontend Architecture Hub]]
