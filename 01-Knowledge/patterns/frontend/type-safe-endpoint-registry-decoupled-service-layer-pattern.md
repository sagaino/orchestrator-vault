---
title: "Type-Safe Endpoint Registry & Decoupled Service Layer Pattern"
type: pattern
tags: [pattern, frontend, api, architecture, typescript, service-layer, rest]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787111639165-e1d30e37
sources: ["Harvest 1787111639165 E1d30e37.json"]
---

# Type-Safe Endpoint Registry & Decoupled Service Layer Pattern

Strongly-typed, decoupled API service layer and endpoint dictionary pattern isolating network calls from component code.

## 1. Overview & Architecture

An architectural pattern separating UI view logic from backend HTTP communication by encapsulating endpoints in a structured dictionary and exposing strongly-typed domain service methods.

## 2. Implementation & Code Structure

src/
├── config/
│   └── constant/
│       └── endpoint.ts      # Structured constant mapping of all backend endpoints
├── services/
│   ├── PersonAccess.ts      # Domain-specific API service functions
│   ├── auth.ts              # Authentication service methods
│   └── masterdata.ts        # Master data service methods
└── types/
    └── response/            # Strongly-typed response interfaces

## 3. Key Implementation Points

- Zero HTTP logic in UI components; all API interaction is channeled through domain service modules.
- Centralized endpoint registry eliminates magic URL strings throughout the project.
- Full end-to-end typing from request schema to response payload interfaces.

## 4. Code Examples

### Service layer isolating API transport logic, URL substitution, and TypeScript typing from UI components.

```typescript
import axios from "@/config/axios";
import { ENDPOINTS } from "@/config/constant/endpoint";
import type { AddCategoryRequest } from "@/pages/PersonAccess/schema";
import type { ICategoryResponse } from "@/pages/PersonAccess/types/CategoryResponse";
import type { IPersonAccessDefaultResponse } from "@/pages/PersonAccess/types/PersonAccessDefaultResponse";

export const PersonAccessServices = {
  createCategory: async (data: AddCategoryRequest): Promise<IPersonAccessDefaultResponse> => {
    const response = await axios.post<IPersonAccessDefaultResponse>(
      ENDPOINTS.PERSON_ACCESS.CATEGORY.CREATE,
      data
    );
    return response.data;
  },

  updateCategory: async ({ id, data }: { id: string; data: AddCategoryRequest }): Promise<IPersonAccessDefaultResponse> => {
    const URL = ENDPOINTS.PERSON_ACCESS.CATEGORY.UPDATE.replace(":id", id);
    const response = await axios.put<IPersonAccessDefaultResponse>(URL, data);
    return response.data;
  },

  deleteCategory: async ({ id, password }: { id: string; password: string }): Promise<IPersonAccessDefaultResponse> => {
    const URL = ENDPOINTS.PERSON_ACCESS.CATEGORY.DELETE.replace(":id", id);
    const response = await axios.delete<IPersonAccessDefaultResponse>(URL, {
      data: { password }
    });
    return response.data;
  },

  getAllCategory: async (params?: string): Promise<ICategoryResponse> => {
    const response = await axios.get<ICategoryResponse>(
      ENDPOINTS.PERSON_ACCESS.CATEGORY.GET_ALL,
      { params: { search: params } }
    );
    return response.data;
  },
};
```

## 5. Considerations & Best Practices

- Parametric endpoints using string templates (e.g. :id) must be replaced before execution.
- Services return raw response.data ensuring consumers handle typed data without Axios envelope pollution.

## 6. Related Knowledge

- [[01-Knowledge/patterns/frontend/api-service-layer.md]]
- Service Repository Pattern

## 7. Source

- Harvest 1787111639165 E1d30e37.json
