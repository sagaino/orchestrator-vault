---
title: "Full Verification Suite"
type: task
task_id: TASK-015
project: be-golang-app
status: DONE
tags: [task, be-golang-app, orchestrator-intake]
created: 2026-08-27
updated: 2026-08-27
dependencies: ["TASK-014"]
verification: ["go test ./...", "go vet ./..."]
allowed_paths: []
requires_changes: false
risk: LOW
complexity: LOW
sources: []
objective_id: "obj-20260827075934-ceadd95a"
plan_id: "plan-obj-20260827075934-ceadd95a-r2"
orchestration_id: "orch-obj-20260827075934-ceadd95a"
node_id: "node-verification"
master_task: "obj-20260827075934-ceadd95a"
orchestration_managed: true
orchestration_dependencies: ["be-golang-app:TASK-014"]
role: "TESTING"
node_type: "VERIFICATION"
write_conflict_group: null
context_from: ["node-auth-unit-tests"]
skill_assignments: []
---

# Full Verification Suite

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

Orchestration node: node-verification

## Tujuan

Verify all package tests and static analysis pass across the repository.

## Scope


## Hasil Yang Diharapkan

Node node-verification memenuhi seluruh acceptance criteria yang disetujui pada plan plan-obj-20260827075934-ceadd95a-r2.

## Acceptance Criteria

1. go test ./... passes completely with zero failures
2. go vet ./... passes with zero diagnostics
3. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
4. Verification `go test ./...` dan `go vet ./...` berhasil.

## Knowledge Decision

- Classification: `IGNORE`
- Destination: `NONE`
- Source: [[03-Sources/other/orchestrator-runs/task-015-20260827T081553Z-58f3fd6e.json]]


## Error Log

Tidak ada error log saat pembuatan task.

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

- [2026-08-27] Task dibuat melalui orchestrator task intake oleh `orchestrator`.

---

## Orchestrator Run Log
- [2026-08-27T08:15:53.477Z] Human `local:sagaino` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-27T08:15:53.752Z] Run `task-015-20260827T081553Z-58f3fd6e` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-27T08:16:50.896Z] Run `task-015-20260827T081553Z-58f3fd6e`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-08-27T08:17:07.557Z] Run `task-015-20260827T081553Z-58f3fd6e`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
