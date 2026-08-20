# FMFU App Service API

**Version:** v1  
**Base URL:** `http://172.19.253.36:8001`  
**Generated:** 2026-08-04  

Event/race platform API: JWT auth, bulk user onboarding, and read-only post-media access.

---

## Table of Contents

**Auth**

- [POST /api/v1/auth/register/bulk](#post-api-v1-auth-register-bulk) — bulkRegister
- [POST /api/v1/auth/register/bulk/retry-failed](#post-api-v1-auth-register-bulk-retry-failed) — retryFailed
- [POST /api/v1/auth/refresh](#post-api-v1-auth-refresh) — refresh
- [POST /api/v1/auth/logout](#post-api-v1-auth-logout) — logout
- [POST /api/v1/auth/login](#post-api-v1-auth-login) — login
- [GET /api/v1/auth/register/bulk/{jobId}/status](#get-api-v1-auth-register-bulk-jobid-status) — bulkRegisterStatus
- [GET /api/v1/auth/register/bulk/{jobId}/report](#get-api-v1-auth-register-bulk-jobid-report) — bulkRegisterReport

**Admin Auth**

- [POST /api/v1/admin/login](#post-api-v1-admin-login) — login_1

**Post Media**

- [GET /api/v1/post-media/{photoId}/thumbnail](#get-api-v1-post-media-photoid-thumbnail) — thumbnail
- [GET /api/v1/post-media/{photoId}/download](#get-api-v1-post-media-photoid-download) — download
- [GET /api/v1/post-media/all](#get-api-v1-post-media-all) — list

**Admin Upload Sessions**

- [GET /api/v1/admin/upload/{sessionId}](#get-api-v1-admin-upload-sessionid) — detail
- [GET /api/v1/admin/upload/all](#get-api-v1-admin-upload-all) — all

---

## Auth

Login, token refresh/logout, and bulk user registration (the only registration path)

### `POST` /api/v1/auth/register/bulk

#### Request Body

**Content-Type:** `multipart/form-data`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file` | `string` | &#10003; |  |

#### Responses

| Status | Description |
|--------|-------------|
| `200` | OK |

#### Security

Requires: `bearerAuth`

---

### `POST` /api/v1/auth/register/bulk/retry-failed

#### Responses

| Status | Description |
|--------|-------------|
| `200` | OK |

#### Security

Requires: `bearerAuth`

---

### `POST` /api/v1/auth/refresh

#### Request Body

**Content-Type:** `application/json`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `refresh_token` | `string` | &#10003; |  |

#### Responses

| Status | Description |
|--------|-------------|
| `200` | OK |

#### Security

Requires: `bearerAuth`

---

### `POST` /api/v1/auth/logout

#### Request Body

**Content-Type:** `application/json`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `refresh_token` | `string` | &#10003; |  |

#### Responses

| Status | Description |
|--------|-------------|
| `200` | OK |

#### Security

Requires: `bearerAuth`

---

### `POST` /api/v1/auth/login

#### Request Body

**Content-Type:** `application/json`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `bib` | `string` | &#10003; |  |
| `phone` | `string` | &#10003; |  |

#### Responses

| Status | Description |
|--------|-------------|
| `200` | OK |

#### Security

Requires: `bearerAuth`

---

### `GET` /api/v1/auth/register/bulk/{jobId}/status

#### Parameters

| Name | In | Type | Required | Description |
|------|----|------|----------|-------------|
| `jobId` | path | `string` | &#10003; |  |

#### Responses

| Status | Description |
|--------|-------------|
| `200` | OK |

#### Security

Requires: `bearerAuth`

---

### `GET` /api/v1/auth/register/bulk/{jobId}/report

#### Parameters

| Name | In | Type | Required | Description |
|------|----|------|----------|-------------|
| `jobId` | path | `string` | &#10003; |  |
| `page` | query | `integer` |  |  |
| `size` | query | `integer` |  |  |
| `status` | query | `string` |  |  |

#### Responses

| Status | Description |
|--------|-------------|
| `200` | OK |

#### Security

Requires: `bearerAuth`

---

## Admin Auth

Login for ADMIN-role accounts only

### `POST` /api/v1/admin/login

#### Request Body

**Content-Type:** `application/json`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `email` | `string` | &#10003; |  |
| `password` | `string` | &#10003; |  |

#### Responses

| Status | Description |
|--------|-------------|
| `200` | OK |

#### Security

Requires: `bearerAuth`

---

## Post Media

Read-only access to post-media photos owned by the external upload pipeline

### `GET` /api/v1/post-media/{photoId}/thumbnail

#### Parameters

| Name | In | Type | Required | Description |
|------|----|------|----------|-------------|
| `principal` | query | `object` | &#10003; |  |
| `photoId` | path | `string` | &#10003; |  |

#### Responses

| Status | Description |
|--------|-------------|
| `200` | OK |

#### Security

Requires: `bearerAuth`

---

### `GET` /api/v1/post-media/{photoId}/download

#### Parameters

| Name | In | Type | Required | Description |
|------|----|------|----------|-------------|
| `principal` | query | `object` | &#10003; |  |
| `photoId` | path | `string` | &#10003; |  |

#### Responses

| Status | Description |
|--------|-------------|
| `200` | OK |

#### Security

Requires: `bearerAuth`

---

### `GET` /api/v1/post-media/all

#### Parameters

| Name | In | Type | Required | Description |
|------|----|------|----------|-------------|
| `principal` | query | `object` | &#10003; |  |
| `page` | query | `integer` |  |  |
| `size` | query | `integer` |  |  |
| `sort` | query | `string` |  |  |

#### Responses

| Status | Description |
|--------|-------------|
| `200` | OK |

#### Security

Requires: `bearerAuth`

---

## Admin Upload Sessions

Read-only ADMIN proxy in front of an external, unauthenticated upload service

### `GET` /api/v1/admin/upload/{sessionId}

#### Parameters

| Name | In | Type | Required | Description |
|------|----|------|----------|-------------|
| `sessionId` | path | `string` | &#10003; |  |
| `page` | query | `integer` | &#10003; |  |
| `size` | query | `integer` | &#10003; |  |

#### Responses

| Status | Description |
|--------|-------------|
| `200` | OK |

#### Security

Requires: `bearerAuth`

---

### `GET` /api/v1/admin/upload/all

#### Parameters

| Name | In | Type | Required | Description |
|------|----|------|----------|-------------|
| `page` | query | `integer` | &#10003; |  |
| `size` | query | `integer` | &#10003; |  |
| `status` | query | `string` |  |  |
| `userId` | query | `string` |  |  |
| `dateFrom` | query | `string` |  |  |
| `dateTo` | query | `string` |  |  |

#### Responses

| Status | Description |
|--------|-------------|
| `200` | OK |

#### Security

Requires: `bearerAuth`

---

## Schemas

### ApiResponseBulkRegisterJobAcceptedResponse

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `code` | `integer` |  |  |
| `message` | `string` |  |  |
| `result` | `object` |  |  |

### BulkRegisterJobAcceptedResponse

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `job_id` | `string` |  |  |
| `status` | `string` |  |  |
| `total_rows` | `integer` |  |  |

### ApiResponseObject

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `code` | `integer` |  |  |
| `message` | `string` |  |  |
| `result` | `object` |  |  |

### RefreshRequest

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `refresh_token` | `string` | &#10003; |  |

### ApiResponseLoginResponse

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `code` | `integer` |  |  |
| `message` | `string` |  |  |
| `result` | `object` |  |  |

### LoginResponse

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `access_token` | `string` |  |  |
| `refresh_token` | `string` |  |  |
| `expires_in` | `integer` |  |  |
| `name` | `string` |  |  |
| `user_id` | `string` |  |  |

### LogoutRequest

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `refresh_token` | `string` | &#10003; |  |

### ApiResponseVoid

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `code` | `integer` |  |  |
| `message` | `string` |  |  |
| `result` | `object` |  |  |

### LoginRequest

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `bib` | `string` | &#10003; |  |
| `phone` | `string` | &#10003; |  |

### AdminLoginRequest

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `email` | `string` | &#10003; |  |
| `password` | `string` | &#10003; |  |

### CustomUserDetails

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `user_id` | `string` |  |  |
| `bib` | `string` |  |  |
| `role` | `string` |  |  |
| `enabled` | `boolean` |  |  |
| `authorities` | `array[GrantedAuthority]` |  |  |
| `account_non_expired` | `boolean` |  |  |
| `credentials_non_expired` | `boolean` |  |  |
| `account_non_locked` | `boolean` |  |  |
| `password` | `string` |  |  |
| `username` | `string` |  |  |

### GrantedAuthority

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `authority` | `string` |  |  |

### ApiResponsePageResultPostMediaListItemResponse

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `code` | `integer` |  |  |
| `message` | `string` |  |  |
| `result` | `object` |  |  |

### PageResultPostMediaListItemResponse

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `content` | `array[PostMediaListItemResponse]` |  |  |
| `page` | `integer` |  |  |
| `size` | `integer` |  |  |
| `total_elements` | `integer` |  |  |
| `total_pages` | `integer` |  |  |

### PostMediaListItemResponse

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | `string` |  |  |
| `user_id` | `string` |  |  |
| `user_name` | `string` |  |  |
| `caption` | `string` |  |  |
| `location` | `string` |  |  |
| `visibility` | `boolean` |  |  |
| `photo_id` | `string` |  |  |
| `original_photo_url` | `string` |  |  |
| `original_photo_size` | `integer` |  |  |
| `thumbnail_photo_url` | `string` |  |  |
| `thumbnail_photo_size` | `integer` |  |  |
| `detect_type` | `string` |  |  |
| `score` | `number` |  |  |
| `created_date` | `string` |  |  |

### ApiResponseBulkRegisterJobStatusResponse

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `code` | `integer` |  |  |
| `message` | `string` |  |  |
| `result` | `object` |  |  |

### BulkRegisterJobStatusResponse

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `job_id` | `string` |  |  |
| `status` | `string` |  |  |
| `total_rows` | `integer` |  |  |
| `processed` | `integer` |  |  |
| `success_count` | `integer` |  |  |
| `failed_count` | `integer` |  |  |

### ApiResponseBulkRegisterJobReportResponse

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `code` | `integer` |  |  |
| `message` | `string` |  |  |
| `result` | `object` |  |  |

### BulkRegisterJobReportResponse

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `job_id` | `string` |  |  |
| `status` | `string` |  |  |
| `total_rows` | `integer` |  |  |
| `success_count` | `integer` |  |  |
| `failed_count` | `integer` |  |  |
| `content` | `array[BulkRegisterRowResult]` |  |  |
| `page` | `integer` |  |  |
| `size` | `integer` |  |  |
| `total_elements` | `integer` |  |  |
| `total_pages` | `integer` |  |  |

### BulkRegisterRowResult

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `row` | `integer` |  |  |
| `bib` | `string` |  |  |
| `status` | `string` |  |  |
| `message` | `string` |  |  |


---

*Generated by [markdown.co.in](https://markdown.co.in/tools/Swagger-to-markdown.html)*
