---
supersession: "ACTIVE"
review_by: "2027-02-18"
owner: "knowledge-curator"
confidence: 0.5
orchestrator_run: "legacy-migration:p8-0-provenance-20260822T103839Z-efa8f1a9"
provenance_schema: 1
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
