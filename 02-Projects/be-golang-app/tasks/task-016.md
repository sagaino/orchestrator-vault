---
title: "Implement Refresh Token Rotation in Auth Usecase and DTOs"
type: task
task_id: TASK-016
project: be-golang-app
status: DONE
tags: [task, be-golang-app, orchestrator-intake]
created: 2026-08-27
updated: 2026-08-27
dependencies: []
verification: ["go test ./...", "go vet ./..."]
allowed_paths: ["internal/core/usecase/auth/**"]
requires_changes: true
risk: MEDIUM
complexity: MEDIUM
sources: []
objective_id: "obj-20260827084531-0e8e7f51"
plan_id: "plan-obj-20260827084531-0e8e7f51-r2"
orchestration_id: "orch-obj-20260827084531-0e8e7f51"
node_id: "subtask-1-auth-usecase"
master_task: "obj-20260827084531-0e8e7f51"
orchestration_managed: true
orchestration_dependencies: []
role: "BACKEND"
node_type: "IMPLEMENTATION"
write_conflict_group: "auth-core"
context_from: []
skill_assignments: ["go-modular-clean@1.0.0#5d6aed0a38849b74f08f3f85e4044d7d619cb8b6261a332bc5cf322cc60cdffa"]
---

# Implement Refresh Token Rotation in Auth Usecase and DTOs

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

Orchestration node: subtask-1-auth-usecase

## Tujuan

Update LoginResponse, define RefreshTokenRequest and RefreshTokenResponse DTOs, and implement RefreshToken rotation with Redis sliding expiration and unit tests in auth usecase.

## Scope

- `internal/core/usecase/auth/**`

## Hasil Yang Diharapkan

Node subtask-1-auth-usecase memenuhi seluruh acceptance criteria yang disetujui pada plan plan-obj-20260827084531-0e8e7f51-r2.

## Acceptance Criteria

1. Update LoginResponse in internal/core/usecase/auth/dto.go to include RefreshToken string json:refreshToken
2. Define RefreshTokenRequest and RefreshTokenResponse structs in internal/core/usecase/auth/dto.go
3. Implement RefreshToken(ctx context.Context, req RefreshTokenRequest) in internal/core/usecase/auth/usecase.go with Redis lookup, key deletion, new token generation, and 7-day sliding TTL
4. Update Login and Logout flows in internal/core/usecase/auth/usecase.go to persist and clear refresh:token:[REDACTED] and session:user:<userId> keys in Redis
5. Add comprehensive unit tests for RTR, invalid/expired token rejection, and logout in internal/core/usecase/auth/usecase_test.go
6. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
7. Verification `go test ./...` dan `go vet ./...` berhasil.

## Knowledge Decision

- Classification: `NEW`
- Destination: `WIKI`
- Source: [[03-Sources/other/orchestrator-runs/task-016-20260827T084612Z-362b3dbb.json]]


## Error Log

Tidak ada error log saat pembuatan task.

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

- [2026-08-27] Task dibuat melalui orchestrator task intake oleh `orchestrator`.

---

## Orchestrator Run Log
- [2026-08-27T08:46:12.783Z] Human `local:sagaino` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-27T08:46:13.061Z] Run `task-016-20260827T084612Z-362b3dbb` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-27T08:48:59.782Z] Run `task-016-20260827T084612Z-362b3dbb`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-08-27T08:49:40.006Z] Run `task-016-20260827T084612Z-362b3dbb`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
