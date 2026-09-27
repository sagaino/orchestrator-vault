---
title: "Implement Auth Middleware Session Validation and Logout Endpoint"
type: task
task_id: TASK-013
project: be-golang-app
status: DONE
tags: [task, be-golang-app, orchestrator-intake]
created: 2026-08-27
updated: 2026-08-27
dependencies: ["TASK-012"]
verification: ["go test ./...", "go vet ./..."]
allowed_paths: ["internal/adapter/controller/auth.go", "internal/adapter/controller/auth_middleware.go", "cmd/api/api.go"]
requires_changes: true
risk: LOW
complexity: MEDIUM
sources: []
objective_id: "obj-20260827075934-ceadd95a"
plan_id: "plan-obj-20260827075934-ceadd95a-r2"
orchestration_id: "orch-obj-20260827075934-ceadd95a"
node_id: "node-auth-controller-middleware"
master_task: "obj-20260827075934-ceadd95a"
orchestration_managed: true
orchestration_dependencies: ["be-golang-app:TASK-012"]
role: "BACKEND"
node_type: "IMPLEMENTATION"
write_conflict_group: "auth-http"
context_from: ["node-auth-usecase-session"]
skill_assignments: ["go-modular-clean@1.0.0#5d6aed0a38849b74f08f3f85e4044d7d619cb8b6261a332bc5cf322cc60cdffa"]
---

# Implement Auth Middleware Session Validation and Logout Endpoint

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

Orchestration node: node-auth-controller-middleware

## Tujuan

Update auth middleware to verify Redis session existence and create POST /api/v1/auth/logout controller endpoint.

## Scope

- `internal/adapter/controller/auth.go`
- `internal/adapter/controller/auth_middleware.go`
- `cmd/api/api.go`

## Hasil Yang Diharapkan

Node node-auth-controller-middleware memenuhi seluruh acceptance criteria yang disetujui pada plan plan-obj-20260827075934-ceadd95a-r2.

## Acceptance Criteria

1. Auth middleware checks Redis for session:user:<userId> after validating JWT token signature
2. Auth middleware returns 401 Unauthorized with 'Session expired or invalid' if Redis session is not found
3. Auth middleware injects currentUser and userId into gin.Context when session is valid
4. Controller exposes POST /api/v1/auth/logout protected by auth middleware returning HTTP 200 'Logout successful'
5. Route is registered in cmd/api/api.go
6. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
7. Verification `go test ./...` dan `go vet ./...` berhasil.

## Knowledge Decision

- Classification: `UPDATE`
- Destination: `WIKI`
- Source: [[03-Sources/other/orchestrator-runs/task-013-20260827T080549Z-96fe5437.json]]


## Error Log

Tidak ada error log saat pembuatan task.

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

- [2026-08-27] Task dibuat melalui orchestrator task intake oleh `orchestrator`.

---

## Orchestrator Run Log
- [2026-08-27T08:05:49.315Z] Human `local:sagaino` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-27T08:05:49.606Z] Run `task-013-20260827T080549Z-96fe5437` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-27T08:08:29.793Z] Run `task-013-20260827T080549Z-96fe5437`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-08-27T08:09:01.902Z] Run `task-013-20260827T080549Z-96fe5437`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
