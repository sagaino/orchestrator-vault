---
title: "Verify Project Test Suite and Static Analysis"
type: task
task_id: TASK-018
project: be-golang-app
status: DONE
tags: [task, be-golang-app, orchestrator-intake]
created: 2026-08-27
updated: 2026-08-27
dependencies: ["TASK-017"]
verification: ["go test ./...", "go vet ./..."]
allowed_paths: []
requires_changes: false
risk: LOW
complexity: LOW
sources: []
objective_id: "obj-20260827084531-0e8e7f51"
plan_id: "plan-obj-20260827084531-0e8e7f51-r2"
orchestration_id: "orch-obj-20260827084531-0e8e7f51"
node_id: "subtask-3-verify-project"
master_task: "obj-20260827084531-0e8e7f51"
orchestration_managed: true
orchestration_dependencies: ["be-golang-app:TASK-017"]
role: "TESTING"
node_type: "VERIFICATION"
write_conflict_group: null
context_from: ["subtask-1-auth-usecase", "subtask-2-auth-controller"]
skill_assignments: []
---

# Verify Project Test Suite and Static Analysis

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

Orchestration node: subtask-3-verify-project

## Tujuan

Run full test suite and go vet across the codebase to ensure complete verification and absence of regressions.

## Scope


## Hasil Yang Diharapkan

Node subtask-3-verify-project memenuhi seluruh acceptance criteria yang disetujui pada plan plan-obj-20260827084531-0e8e7f51-r2.

## Acceptance Criteria

1. All Go package tests pass cleanly via go test ./...
2. Static analysis passes cleanly via go vet ./...
3. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
4. Verification `go test ./...` dan `go vet ./...` berhasil.

## Knowledge Decision

- Classification: `IGNORE`
- Destination: `NONE`
- Source: [[03-Sources/other/orchestrator-runs/task-018-20260827T085549Z-22ac7b94.json]]


## Error Log

Tidak ada error log saat pembuatan task.

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

- [2026-08-27] Task dibuat melalui orchestrator task intake oleh `orchestrator`.

---

## Orchestrator Run Log
- [2026-08-27T08:55:49.238Z] Human `local:sagaino` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-27T08:55:49.535Z] Run `task-018-20260827T085549Z-22ac7b94` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-27T08:56:31.009Z] Run `task-018-20260827T085549Z-22ac7b94`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-08-27T08:57:14.677Z] Run `task-018-20260827T085549Z-22ac7b94`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
