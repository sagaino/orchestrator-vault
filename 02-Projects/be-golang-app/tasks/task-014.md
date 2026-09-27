---
title: "Add Unit Tests for Redis Session Management and Middleware"
type: task
task_id: TASK-014
project: be-golang-app
status: DONE
tags: [task, be-golang-app, orchestrator-intake]
created: 2026-08-27
updated: 2026-08-27
dependencies: ["TASK-013"]
verification: ["go test ./...", "go vet ./..."]
allowed_paths: ["internal/core/usecase/auth/**", "internal/adapter/controller/auth_test.go", "internal/adapter/controller/auth_middleware_test.go"]
requires_changes: true
risk: LOW
complexity: MEDIUM
sources: []
objective_id: "obj-20260827075934-ceadd95a"
plan_id: "plan-obj-20260827075934-ceadd95a-r2"
orchestration_id: "orch-obj-20260827075934-ceadd95a"
node_id: "node-auth-unit-tests"
master_task: "obj-20260827075934-ceadd95a"
orchestration_managed: true
orchestration_dependencies: ["be-golang-app:TASK-013"]
role: "TESTING"
node_type: "IMPLEMENTATION"
write_conflict_group: null
context_from: ["node-auth-usecase-session", "node-auth-controller-middleware"]
skill_assignments: ["go-modular-clean@1.0.0#5d6aed0a38849b74f08f3f85e4044d7d619cb8b6261a332bc5cf322cc60cdffa"]
---

# Add Unit Tests for Redis Session Management and Middleware

## Permintaan User

Tolong implementasikan Redis Session Management pada autentikasi JWT di proyek be-golang-app dengan spesifikasi berikut:

1. **Arsitektur & Konvensi**:
   - Ikuti 3-Layer Modular Clean Architecture (Domain, Usecase, Controller).
   - Manfaatkan Redis client wrapper yang sudah ada di `pkg/cache/cache.go`.
   - Gunakan `pkg/localerror` dan `shared/payload` untuk standardisasi format error respons.

2. **Flow Login (`internal/core/usecase/auth/`)**:
   - Setelah validasi password via bcrypt berhasil dan token JWT dibuat, simpan snapshot data user ke Redis dengan key `session:user:<userId>`.
   - Set masa berlaku (TTL) session di Redis sama dengan masa berlaku token JWT (baca dari env `EXPIRED_TOKEN_JWT_MINUTES` atau default 3 menit).

3. **Flow Middleware Auth (`internal/adapter/controller/auth_middleware.go`)**:
   - Setelah tanda tangan JWT valid, periksa ketersediaan session di Redis menggunakan key `session:user:<userId>`.
   - Jika key tidak ditemukan di Redis (misal user sudah logout atau session dibatalkan), return `401 Unauthorized` dengan pesan "Session expired or invalid".
   - Jika session ada di Redis, set data user lengkap ke `gin.Context` (`c.Set("currentUser", user)` dan `c.Set("userId", claims.UserID)`).

4. **Flow Logout (`internal/adapter/controller/auth.go` & usecase)**:
   - Buat usecase dan endpoint baru `POST /api/v1/auth/logout` yang diproteksi oleh auth middleware.
   - Saat logout dipanggil, hapus key session user dari Redis (`cache.Delete`).
   - Kembalikan respons sukses HTTP 200 "Logout successful".

5. **Verifikasi & Unit Tests**:
   - Perbarui dan tambahkan unit tests di `internal/core/usecase/auth/usecase_test.go` dan `internal/adapter/controller/auth_middleware_test.go` untuk menguji skenario session hit, session missing (401), dan logout.
   - Pastikan seluruh suite `go test ./...` dan build lulus 100%.

Orchestration node: node-auth-unit-tests

## Tujuan

Add unit tests covering login session creation, middleware session validation (hit/miss), and logout session deletion.

## Scope

- `internal/core/usecase/auth/**`
- `internal/adapter/controller/auth_test.go`
- `internal/adapter/controller/auth_middleware_test.go`

## Hasil Yang Diharapkan

Node node-auth-unit-tests memenuhi seluruh acceptance criteria yang disetujui pada plan plan-obj-20260827075934-ceadd95a-r2.

## Acceptance Criteria

1. Unit tests in internal/core/usecase/auth/usecase_test.go verify session saving on login and deletion on logout
2. Unit tests in internal/adapter/controller/auth_test.go and auth_middleware_test.go test session hit, session miss (401), and logout handler
3. Mocked Redis cache assertions verify expected key names and TTLs
4. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
5. Verification `go test ./...` dan `go vet ./...` berhasil.

## Knowledge Decision

- Classification: `UPDATE`
- Destination: `WIKI`
- Source: [[03-Sources/other/orchestrator-runs/task-014-20260827T080907Z-618ab2c0.json]]


## Error Log

Tidak ada error log saat pembuatan task.

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

- [2026-08-27] Task dibuat melalui orchestrator task intake oleh `orchestrator`.

---

## Orchestrator Run Log
- [2026-08-27T08:09:07.354Z] Human `local:sagaino` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-27T08:09:07.617Z] Run `task-014-20260827T080907Z-618ab2c0` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-27T08:15:10.099Z] Run `task-014-20260827T080907Z-618ab2c0`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-08-27T08:15:51.244Z] Run `task-014-20260827T080907Z-618ab2c0`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
