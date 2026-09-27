---
title: "Standardize Auth Controller Responses"
type: task
task_id: TASK-019
project: be-golang-app
status: DONE
tags: [task, be-golang-app, orchestrator-intake]
created: 2026-08-27
updated: 2026-08-27
dependencies: []
verification: ["go test ./...", "go vet ./..."]
allowed_paths: ["internal/adapter/controller/auth.go", "internal/adapter/controller/auth_test.go"]
requires_changes: true
risk: LOW
complexity: LOW
sources: []
objective_id: "obj-20260827091620-a6ef6370"
plan_id: "plan-obj-20260827091620-a6ef6370-r2"
orchestration_id: "orch-obj-20260827091620-a6ef6370"
node_id: "node-refactor-auth"
master_task: "obj-20260827091620-a6ef6370"
orchestration_managed: true
orchestration_dependencies: []
role: "BACKEND"
node_type: "IMPLEMENTATION"
write_conflict_group: null
context_from: []
skill_assignments: ["go-modular-clean@1.0.0#5d6aed0a38849b74f08f3f85e4044d7d619cb8b6261a332bc5cf322cc60cdffa"]
---

# Standardize Auth Controller Responses

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

Orchestration node: node-refactor-auth

## Tujuan

Refactor Register, Login, RefreshToken, and Logout endpoints in auth controller to use shared/payload helper functions.

## Scope

- `internal/adapter/controller/auth.go`
- `internal/adapter/controller/auth_test.go`

## Hasil Yang Diharapkan

Node node-refactor-auth memenuhi seluruh acceptance criteria yang disetujui pada plan plan-obj-20260827091620-a6ef6370-r2.

## Acceptance Criteria

1. Use payload.DefaultErrorInvalidDataWithMessage for 400 Bad Request error responses.
2. Use payload.NewSuccessResponse for Login, Register, and RefreshToken successful responses.
3. Use payload.NewSuccessResponse / payload.NewSuccessResponseNoData for Logout responses.
4. Auth tests pass with go test ./...
5. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
6. Verification `go test ./...` dan `go vet ./...` berhasil.

## Knowledge Decision

- Classification: `PROJECT_ONLY`
- Destination: `PROJECT`
- Source: [[03-Sources/other/orchestrator-runs/task-019-20260827T091711Z-d8901c15.json]]


## Error Log

Tidak ada error log saat pembuatan task.

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

- [2026-08-27] Task dibuat melalui orchestrator task intake oleh `orchestrator`.

---

## Orchestrator Run Log
- [2026-08-27T09:17:11.448Z] Human `local:sagaino` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-27T09:17:11.737Z] Run `task-019-20260827T091711Z-d8901c15` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-27T09:18:38.468Z] Run `task-019-20260827T091711Z-d8901c15`: coding agent, verification, dan Graphify selesai; menunggu human review.

## Knowledge Retrospective

<!-- orchestrator-run:task-019-20260827T091711Z-d8901c15 -->
- Classification: `PROJECT_ONLY`
- Summary: Refactoring controller autentikasi (Register, Login, RefreshToken, Logout) pada be-golang-app untuk menstandarisasi pembuatan payload respons HTTP menggunakan helper functions dari shared/payload (DefaultErrorInvalidDataWithMessage, NewSuccessResponse, NewSuccessResponseNoData).
- Rationale: Refactoring ini merupakan penyesuaian internal layer controller pada repositori be-golang-app agar selaras dengan helper shared/payload yang sudah ada. Perubahan tidak memperkenalkan konsep arsitektur baru lintas-proyek ataupun memodifikasi kontrak global wiki.
- Source: [[03-Sources/other/orchestrator-runs/task-019-20260827T091711Z-d8901c15.json]]
- [2026-08-27T09:31:34.247Z] Run `task-019-20260827T091711Z-d8901c15`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
