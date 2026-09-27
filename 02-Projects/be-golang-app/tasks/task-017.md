---
title: "Add Refresh Token HTTP Handler and Route Registration"
type: task
task_id: TASK-017
project: be-golang-app
status: DONE
tags: [task, be-golang-app, orchestrator-intake]
created: 2026-08-27
updated: 2026-08-27
dependencies: ["TASK-016"]
verification: ["go test ./...", "go vet ./..."]
allowed_paths: ["internal/adapter/controller/auth.go", "internal/adapter/controller/auth_test.go", "cmd/api/api.go"]
requires_changes: true
risk: LOW
complexity: LOW
sources: []
objective_id: "obj-20260827084531-0e8e7f51"
plan_id: "plan-obj-20260827084531-0e8e7f51-r2"
orchestration_id: "orch-obj-20260827084531-0e8e7f51"
node_id: "subtask-2-auth-controller"
master_task: "obj-20260827084531-0e8e7f51"
orchestration_managed: true
orchestration_dependencies: ["be-golang-app:TASK-016"]
role: "BACKEND"
node_type: "IMPLEMENTATION"
write_conflict_group: "auth-controller"
context_from: ["subtask-1-auth-usecase"]
skill_assignments: ["go-modular-clean@1.0.0#5d6aed0a38849b74f08f3f85e4044d7d619cb8b6261a332bc5cf322cc60cdffa"]
---

# Add Refresh Token HTTP Handler and Route Registration

## Permintaan User

Tolong implementasikan fitur Refresh Token Rotation (RTR) dengan Sliding Expiration berbasis Redis pada proyek be-golang-app dengan spesifikasi berikut:

1. **Arsitektur & Konvensi**:
   - Ikuti 3-Layer Modular Clean Architecture (Domain, Usecase, Controller).
   - Manfaatkan Redis client wrapper di `pkg/cache/cache.go`.
   - Gunakan `pkg/localerror` dan `shared/payload` untuk standardisasi format error respons.

2. **Perubahan DTO (`internal/core/usecase/auth/dto.go`)**:
   - Perbarui `LoginResponse` agar menyertakan field `RefreshToken string json:"refreshToken"`.
   - Buat struct `RefreshTokenRequest`:
     ```go
     type RefreshTokenRequest struct {
         RefreshToken string `json:"refreshToken" binding:"required"`
     }
     ```
   - Buat struct `RefreshTokenResponse`:
     ```go
     type RefreshTokenResponse struct {
         AccessToken  string `json:"accessToken"`
         RefreshToken string `json:"refreshToken"`
         TokenType    string `json:"tokenType"`
     }
     ```

3. **Flow Login & Refresh Token di Usecase (`internal/core/usecase/auth/usecase.go`)**:
   - **Saat Login**:
     * Buat `AccessToken` (JWT berumur pendek, misal 3-15 menit).
     * Buat `RefreshToken` unik (UUID v4 / secure token).
     * Simpan `RefreshToken` ke Redis dengan key `refresh:token:[REDACTED] berisi data user snapshot dan TTL 7 hari (baca dari env `REFRESH_TOKEN_TTL_DAYS` atau default 7 hari).
   - **Method Baru `RefreshToken(ctx context.Context, req RefreshTokenRequest)`**:
     * Cari key `refresh:token:[REDACTED] di Redis.
     * Jika tidak ditemukan (expired atau tidak valid), kembalikan error `401 Unauthorized` ("Invalid or expired refresh token").
     * Jika ditemukan, terapkan **Refresh Token Rotation (RTR)**:
       1. Hapus refresh token lama dari Redis (`cache.Delete`).
       2. Buat `AccessToken` baru dan `RefreshToken` baru.
       3. Simpan `RefreshToken` baru ke Redis dengan TTL 7 hari dari sekarang (*Sliding Expiration*).
       4. Perbarui session user di Redis `session:user:<userId>`.
       5. Kembalikan `RefreshTokenResponse` dengan pasangan token baru.
   - **Saat Logout**:
     * Hapus key session `session:user:<userId>` dan hapus key `refresh:token:[REDACTED] dari Redis.

4. **Controller & Routing (`internal/adapter/controller/auth.go`)**:
   - Tambahkan handler `RefreshToken(c *gin.Context)`.
   - Daftarkan endpoint publik baru: `POST /api/v1/auth/refresh`.
   - Perbarui endpoint `POST /api/v1/auth/logout` agar dapat menerima dan menghapus refresh token jika dikirimkan oleh client.

5. **Verifikasi & Unit Tests**:
   - Tambahkan unit test komprehensif di `internal/core/usecase/auth/usecase_test.go` dan `internal/adapter/controller/auth_test.go`:
     * Test case refresh token sukses dan verifikasi token lama terhapus serta token baru aktif.
     * Test case refresh token gagal (401 Unauthorized) jika token palsu / sudah expired.
     * Test case logout membersihkan session dan refresh token.
   - Pastikan seluruh suite `go test ./...` dan build lulus 100%.

Orchestration node: subtask-2-auth-controller

## Tujuan

Implement RefreshToken Gin handler, update Logout handler to accept refresh token, register POST /api/v1/auth/refresh, and add controller tests.

## Scope

- `internal/adapter/controller/auth.go`
- `internal/adapter/controller/auth_test.go`
- `cmd/api/api.go`

## Hasil Yang Diharapkan

Node subtask-2-auth-controller memenuhi seluruh acceptance criteria yang disetujui pada plan plan-obj-20260827084531-0e8e7f51-r2.

## Acceptance Criteria

1. Add RefreshToken handler in internal/adapter/controller/auth.go using shared/payload.Response and pkg/localerror
2. Update Logout handler in internal/adapter/controller/auth.go to invalidate refresh token if provided
3. Register public endpoint POST /api/v1/auth/refresh in cmd/api/api.go or internal/adapter/controller/auth.go
4. Add controller unit tests in internal/adapter/controller/auth_test.go
5. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
6. Verification `go test ./...` dan `go vet ./...` berhasil.

## Knowledge Decision

- Classification: `PROJECT_ONLY`
- Destination: `PROJECT`
- Source: [[03-Sources/other/orchestrator-runs/task-017-20260827T084941Z-f7f1b77c.json]]


## Error Log

Tidak ada error log saat pembuatan task.

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

- [2026-08-27] Task dibuat melalui orchestrator task intake oleh `orchestrator`.

---

## Orchestrator Run Log
- [2026-08-27T08:49:41.440Z] Human `local:sagaino` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-27T08:49:41.725Z] Run `task-017-20260827T084941Z-f7f1b77c` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-27T08:51:52.562Z] Run `task-017-20260827T084941Z-f7f1b77c`: coding agent, verification, dan Graphify selesai; menunggu human review.

## Knowledge Retrospective

<!-- orchestrator-run:task-017-20260827T084941Z-f7f1b77c -->
- Classification: `PROJECT_ONLY`
- Summary: Implementasi handler Gin RefreshToken dan Logout dengan invalidasi refresh token serta registrasi endpoint POST /api/v1/auth/refresh pada be-golang-app.
- Rationale: Perubahan pada TASK-017 adalah implementasi controller HTTP handler (RefreshToken dan penyesuaian Logout) serta registrasi route di Gin untuk endpoint refresh token. Ini merupakan implementasi spesifik controller layer untuk kebutuhan fitur Refresh Token Rotation di project be-golang-app tanpa memperkenalkan konsep atau pattern baru di luar pattern arsitektur backend yang sudah terdokumentasi di Wiki.
- Source: [[03-Sources/other/orchestrator-runs/task-017-20260827T084941Z-f7f1b77c.json]]
- [2026-08-27T08:55:46.898Z] Run `task-017-20260827T084941Z-f7f1b77c`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
