---
title: "generic-envelope-dto-standardized-paginated-response-contract"
type: pattern
tags: [pattern, frontend, dto, envelope, pagination, api-contract, services]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787132260416-42b894f9
sources: ["Harvest 1787132260416 42b894f9.json"]
---

# generic-envelope-dto-standardized-paginated-response-contract

Standarisasi kontrak Generic Envelope DTO dan metadata pagination untuk unboxing respons API yang konsisten dan type-safe.

## 1. Overview & Architecture

Pola arsitektur standarisasi kontrak respons data transfer object (DTO) dengan format generic envelope dan metadata pagination yang terpadu di seluruh domain service enterprise. Memastikan konsistensi penanganan data antara backend API, service repository, dan UI components.

## 2. Implementation & Code Structure

src/types/response/ (Global response contract declarations)
├── src/pages/**/types/response/ (Domain-specific envelope & paginated models)
└── src/services/ (Service layer methods typed against envelope DTOs)

## 3. Key Implementation Points

- Standarisasi Envelope DTO yang konsisten pada seluruh endpoint API (`code`, `message`, `result`).
- Struktur generic pagination cursor (`values`, `element_total`, `page_total`, `empty`, `first`, `last`).
- Type contract decoupling antara domain payload item dengan pagination metadata.
- Direct typed integration dengan Axios instance dan TanStack Query repository layer.

## 4. Code Examples

### Generic envelope and pagination response DTO contract definition.

```typescript
// src/pages/MasterData/types/response/responseMasterDataArea.ts
export interface ResponeseMasterDataArea {
  code: number;
  message: string;
  result: PaginatedResult;
}

export interface AreaItem {
  code: string;
  name: string;
  description: string;
  total_floor: number;
  total_floor_created: number;
  area_key: string;
}

export interface PaginatedResult {
  values: AreaItem[];
  element_total: number;
  page: number;
  page_total: number;
  empty: boolean;
  first: boolean;
  last: boolean;
}
```

### Standardized service unboxing typed envelope responses.

```typescript
// src/services/dropDown.ts
export const dropDownServices = {
  getAllArea: async (
    param: RequestDeviceConfigGetAllAreaOrFloorOrLocaltion
  ): Promise<ResponseDeviceConfigAllAreaOrFloorOrLocation> => {
    const response = await axios.get<ResponseDeviceConfigAllAreaOrFloorOrLocation>(
      ENDPOINTS.DROPDOWN.AREA,
      {
        params: param,
      }
    );
    return response.data;
  },
};
```

## 5. Considerations & Best Practices

- Struktur envelope seragam (code, message, result) memudahkan abstraksi error handler dan notification toaster global.
- Metadata pagination (`element_total`, `page_total`, `first`, `last`) harus selalu disinkronkan dengan pagination UI component.
- Nullability guard pada field item result mencegah runtime crash saat backend mengembalikan null values.

## 6. Related Knowledge

- `Data Transfer Object (DTO) Pattern`
- `Envelope Response Pattern`
- `Pagination Metadata Standards`

## 7. Source

- Harvest 1787132260416 42b894f9.json
