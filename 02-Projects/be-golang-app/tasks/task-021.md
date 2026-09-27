---
title: "Standardize Product and Category Controller Responses"
type: task
task_id: TASK-021
project: be-golang-app
status: DONE
tags: [task, be-golang-app, orchestrator-intake]
created: 2026-08-27
updated: 2026-08-27
dependencies: []
verification: ["go test ./...", "go vet ./..."]
allowed_paths: ["internal/adapter/controller/product.go", "internal/adapter/controller/product_test.go", "internal/adapter/controller/category.go", "internal/adapter/controller/category_test.go"]
requires_changes: true
risk: LOW
complexity: LOW
sources: []
objective_id: "obj-20260827091620-a6ef6370"
plan_id: "plan-obj-20260827091620-a6ef6370-r2"
orchestration_id: "orch-obj-20260827091620-a6ef6370"
node_id: "node-refactor-catalog"
master_task: "obj-20260827091620-a6ef6370"
orchestration_managed: true
orchestration_dependencies: []
role: "BACKEND"
node_type: "IMPLEMENTATION"
write_conflict_group: null
context_from: []
skill_assignments: ["go-modular-clean@1.0.0#5d6aed0a38849b74f08f3f85e4044d7d619cb8b6261a332bc5cf322cc60cdffa"]
---

# Standardize Product and Category Controller Responses

## Permintaan User

Tolong lakukan refactoring di proyek be-golang-app untuk menstandarisasi seluruh respons API di layer controller menggunakan helper function dari `shared/payload/response.go`:

1. **Tujuan Refactoring**:
   - Ganti seluruh pembuatan struct manual `payload.ErrorResponse{Code: http.StatusBadRequest, ...}` dengan helper `payload.DefaultErrorInvalidDataWithMessage(...)`.
   - Ganti seluruh pembuatan struct manual `payload.Response{Code: http.StatusOK, ...}` dengan helper `payload.NewSuccessResponse(...)` atau `payload.NewSuccessResponseNoData(...)`.

2. **File Target**:
   - `internal/adapter/controller/auth.go` (Register, Login, RefreshToken, Logout)
   - `internal/adapter/controller/product.go` (Create, List, Delete)
   - `internal/adapter/controller/category.go` (Create, List, Delete)
   - `internal/adapter/controller/user.go` (GetProfile)

3. **Pola Standarisasi**:

   - **A. Error Validasi / Bad Request (HTTP 400)**:
     ```go
     // Ganti dari:
     c.JSON(http.StatusBadRequest, payload.ErrorResponse{
         Code:    http.StatusBadRequest,
         Message: err.Error(),
     })
     // Menjadi:
     c.JSON(http.StatusBadRequest, payload.DefaultErrorInvalidDataWithMessage(err.Error()))
     ```

   - **B. Success Response dengan Data (HTTP 200 / 201)**:
     ```go
     // Ganti dari:
     c.JSON(http.StatusOK, payload.Response{
         Code:    http.StatusOK,
         Message: "Login successful",
         Result:  res,
     })
     // Menjadi:
     c.JSON(http.StatusOK, payload.NewSuccessResponse(res, "Login successful"))
     ```
     *(Untuk status 201 Created pada Register/Create, tetap gunakan `http.StatusCreated` dengan struct atau `payload.NewSuccessResponse` yang sesuai).*

   - **C. Success Response Tanpa Data / Aksi Selesai (Logout / Delete)**:
     ```go
     // Ganti dari:
     c.JSON(http.StatusOK, payload.Response{
         Code:    http.StatusOK,
         Message: "Logout successful",
     })
     // Menjadi:
     c.JSON(http.StatusOK, payload.NewSuccessResponse(nil, "Logout successful"))
     // atau:
     c.JSON(http.StatusOK, payload.NewSuccessResponseNoData("Logout successful"))
     ```

4. **Verifikasi**:
   - Pastikan struktur JSON respons (`code`, `message`, `result`) tetap 100% kompatibel dan tidak ada *breaking change*.
   - Jalankan seluruh suite test `go test ./...` dan pastikan 100% PASS.

Orchestration node: node-refactor-catalog

## Tujuan

Refactor Create, List, and Delete endpoints in product and category controllers to use shared/payload helper functions.

## Scope

- `internal/adapter/controller/product.go`
- `internal/adapter/controller/product_test.go`
- `internal/adapter/controller/category.go`
- `internal/adapter/controller/category_test.go`

## Hasil Yang Diharapkan

Node node-refactor-catalog memenuhi seluruh acceptance criteria yang disetujui pada plan plan-obj-20260827091620-a6ef6370-r2.

## Acceptance Criteria

1. Use payload.DefaultErrorInvalidDataWithMessage for validation and bad request errors.
2. Use payload.NewSuccessResponse for Create and List responses.
3. Use payload.NewSuccessResponse / payload.NewSuccessResponseNoData for Delete responses.
4. Product and category tests pass with go test ./...
5. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
6. Verification `go test ./...` dan `go vet ./...` berhasil.

## Knowledge Decision

- Classification: `PROJECT_ONLY`
- Destination: `PROJECT`
- Source: [[03-Sources/other/orchestrator-runs/task-021-20260827T093136Z-464d9e49.json]]


## Error Log

Tidak ada error log saat pembuatan task.

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

- [2026-08-27] Task dibuat melalui orchestrator task intake oleh `orchestrator`.

---

## Orchestrator Run Log
- [2026-08-27T09:31:36.095Z] Human `local:sagaino` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-27T09:31:36.355Z] Run `task-021-20260827T093136Z-464d9e49` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-27T09:33:22.007Z] Run `task-021-20260827T093136Z-464d9e49`: coding agent, verification, dan Graphify selesai; menunggu human review.

## Knowledge Retrospective

<!-- orchestrator-run:task-021-20260827T093136Z-464d9e49 -->
- Classification: `PROJECT_ONLY`
- Summary: Standarisasi pembuatan response controller pada product dan category menggunakan shared/payload helper function (NewSuccessResponse dan DefaultErrorInvalidDataWithMessage).
- Rationale: Refactoring standarisasi response controller menggunakan helper bawaan repository (shared/payload) merupakan konvensi dan implementasi spesifik untuk proyek be-golang-app tanpa memunculkan pola arsitektur global baru.
- Source: [[03-Sources/other/orchestrator-runs/task-021-20260827T093136Z-464d9e49.json]]
- [2026-08-27T09:33:56.002Z] Run `task-021-20260827T093136Z-464d9e49`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
