---
title: "Implement Redis Session in Auth Usecase"
type: task
task_id: TASK-012
project: be-golang-app
status: DONE
tags: [task, be-golang-app, orchestrator-intake]
created: 2026-08-27
updated: 2026-08-27
dependencies: []
verification: ["go test ./...", "go vet ./..."]
allowed_paths: ["internal/core/usecase/auth/**"]
requires_changes: true
risk: LOW
complexity: MEDIUM
sources: []
objective_id: "obj-20260827075934-ceadd95a"
plan_id: "plan-obj-20260827075934-ceadd95a-r2"
orchestration_id: "orch-obj-20260827075934-ceadd95a"
node_id: "node-auth-usecase-session"
master_task: "obj-20260827075934-ceadd95a"
orchestration_managed: true
orchestration_dependencies: []
role: "BACKEND"
node_type: "IMPLEMENTATION"
write_conflict_group: "auth-core"
context_from: []
skill_assignments: ["go-modular-clean@1.0.0#5d6aed0a38849b74f08f3f85e4044d7d619cb8b6261a332bc5cf322cc60cdffa"]
---

# Implement Redis Session in Auth Usecase

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

Orchestration node: node-auth-usecase-session

## Tujuan

Update login usecase to persist user session in Redis with TTL and implement logout usecase to invalidate user session.

## Scope

- `internal/core/usecase/auth/**`

## Hasil Yang Diharapkan

Node node-auth-usecase-session memenuhi seluruh acceptance criteria yang disetujui pada plan plan-obj-20260827075934-ceadd95a-r2.

## Acceptance Criteria

1. Login usecase saves user session snapshot in Redis with key session:user:<userId> after successful bcrypt validation and JWT creation
2. Redis session TTL matches JWT expiration duration from EXPIRED_TOKEN_JWT_MINUTES or default 3 minutes
3. Logout usecase deletes session:user:<userId> key from Redis via cache.Delete
4. Standardized errors returned using pkg/localerror
5. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
6. Verification `go test ./...` dan `go vet ./...` berhasil.

## Knowledge Decision

- Classification: `PROJECT_ONLY`
- Destination: `PROJECT`
- Source: [[03-Sources/other/orchestrator-runs/task-012-20260827T080142Z-77f2d7fa.json]]


## Error Log

Tidak ada error log saat pembuatan task.

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

- [2026-08-27] Task dibuat melalui orchestrator task intake oleh `orchestrator`.

---

## Orchestrator Run Log
- [2026-08-27T08:01:42.216Z] Human `local:sagaino` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-27T08:01:42.512Z] Run `task-012-20260827T080142Z-77f2d7fa` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-27T08:05:24.013Z] Run `task-012-20260827T080142Z-77f2d7fa`: coding agent, verification, dan Graphify selesai; menunggu human review.

## Knowledge Retrospective

<!-- orchestrator-run:task-012-20260827T080142Z-77f2d7fa -->
- Classification: `PROJECT_ONLY`
- Summary: Implementasi Redis session management pada Auth Usecase (login user session cache snapshot dan logout cache deletion) di be-golang-app serta auto-recovery penambahan import 'strings' pada unit test.
- Rationale: Implementasi session cache Redis pada layer usecase auth (login/logout) dan perbaikan import 'strings' pada unit test merupakan implementasi spesifik project be-golang-app. Pola umum stateful cache session sudah tercatat di Wiki global (01-Knowledge/patterns/backend/stateful-cache-backed-jwt-session-guard-with-context-manifest-activity-tracking.md). Kesalahan kompilasi test yang diperbaiki melalui recovery hanya penambahan import 'strings' untuk strings.Contains, sehingga tidak memerlukan entri knowledge global baru ataupun update pada pola existing.
- Source: [[03-Sources/other/orchestrator-runs/task-012-20260827T080142Z-77f2d7fa.json]]
- [2026-08-27T08:05:48.225Z] Run `task-012-20260827T080142Z-77f2d7fa`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
