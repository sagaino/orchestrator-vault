---
title: "Standardized Response Envelope DTO Architecture with Recursive Biometric Asset URL Normalization"
type: pattern
tags: [pattern, frontend, dto-envelope, normalization, biometrics, services, electron-main]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127443075-f8fd0716
sources: ["Harvest 1787127443075 F8fd0716.json"]
---

# Standardized Response Envelope DTO Architecture with Recursive Biometric Asset URL Normalization

Standardized Response Envelope DTO Architecture with Recursive Biometric Asset URL Normalization in Electron Main Process.

## 1. Overview & Architecture

A backend envelope DTO handling and recursive biometric asset URL normalization architecture where raw backend DTO envelopes are validated and deeply transformed in the Main process service layer, ensuring the Renderer layer receives fully-qualified, render-ready asset URLs without leaking environment storage configurations to the client.

## 2. Implementation & Code Structure

src/
├── shared/
│   ├── admin/types.ts               # Envelope DTOs, record models, and pagination contracts
│   └── endpoint.ts                  # Centralized endpoint registry
└── main/
    ├── utils/
    │   └── normalize-face-url.ts    # Biometric asset path normalization utility
    └── services/
        └── admin.service.ts         # Service layer executing deep response transformation

## 3. Key Implementation Points

- Standardized DTO envelope structure ({ code, message, result }) wrapping scalar entities or paginated collections.
- Recursive Main process interceptor traversing nested DTO collections (e.g. values[i].face_list[j].face) to sanitize asset endpoints before IPC return.
- Decoupled storage URL resolution driven by environment variables in the Main process boundary.

## 4. Code Examples

### Generic response envelope and paginated result contracts.

```typescript
// src/shared/admin/types.ts
export interface AdminManagementResult {
  values: AdminRecord[]
  element_total: number
  page: number
  page_total: number
  first: boolean
  empty: boolean
  last: boolean
}

export interface AdminManagementResponse {
  code: number
  message: string
  result: AdminManagementResult
}

export interface AdminDetailResponse {
  code: number
  message: string
  result: AdminRecord
}
```

### URL normalizer utility handling relative path canonicalization and trailing slash trimming.

```typescript
// src/main/utils/normalize-face-url.ts
const IMAGE_BASE_URL = process.env.IMAGE_BASE_URL || 'http://localhost:8080/storage'

export const normalizeFaceUrl = (faceUrl: string): string => {
  if (!faceUrl) return ''
  if (faceUrl.startsWith('http://') || faceUrl.startsWith('https://')) return faceUrl

  const baseUrl = IMAGE_BASE_URL.replace(/\/+$/, '')
  const normalizedPath = faceUrl.startsWith('/') ? faceUrl : `/${faceUrl}`

  return `${baseUrl}${normalizedPath}`
}
```

### Main process service intercepting raw backend envelopes and mapping nested biometric asset URLs into canonical addresses.

```typescript
// src/main/services/admin.service.ts
export const AdminService = {
  management: async (payload: AdminManagementRequest): Promise<AdminManagementResponse> => {
    try {
      const response = await axios.get<AdminManagementResponse>(
        `${API_BASE_URL}${ENDPOINTS.ADMIN.MANAGEMENT}`,
        {
          headers: {
            Authorization: `Bearer ${payload.token}`,
            ...buildRequestSecurityHeaders({}),
          },
          params: {
            page: Math.max(payload.page - 1, 0),
            size: payload.size,
            search: payload.search ?? '',
            status: payload.status ?? '',
            order: payload.order ?? 'desc',
            sortBy: payload.sortBy ?? 'id',
          },
          timeout: 15000,
        },
      )

      return {
        ...response.data,
        result: {
          ...response.data.result,
          values: response.data.result.values.map((item) => ({
            ...item,
            face_list: item.face_list.map((face) => ({
              ...face,
              face: normalizeFaceUrl(face.face),
            })),
          })),
        },
      }
    } catch (error) {
      if (axios.isAxiosError(error)) {
        const statusCode = error.response?.status
        const apiMessage =
          (error.response?.data as { message?: string } | undefined)?.message ||
          error.message ||
          'Gagal mengambil data admin'

        throw new Error(
          statusCode ? `HTTP_STATUS_${statusCode}::${apiMessage}` : apiMessage
        )
      }
      throw error
    }
  },
}
```

## 5. Considerations & Best Practices

- Prevent double-prefixing by inspecting if the asset URL already begins with http:// or https://.
- Trailing slashes in IMAGE_BASE_URL and leading slashes in relative paths must be sanitized to avoid malformed URL paths.
- Performing URL canonicalization in the Main process shields the Renderer UI components from having to manage backend storage host configurations.

## 6. Related Knowledge

- Generic Api Envelope Dto Handling
- Biometric Asset Url Canonicalization

## 7. Source

- Harvest 1787127443075 F8fd0716.json
