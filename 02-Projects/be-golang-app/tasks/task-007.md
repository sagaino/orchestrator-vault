---
title: "Implement HTTP Controllers, Auth Middleware, and Wire API Routes"
type: task
task_id: TASK-007
project: be-golang-app
status: DONE
tags: [task, be-golang-app, orchestrator-intake]
created: 2026-08-26
updated: 2026-08-26
dependencies: ["TASK-006"]
verification: ["go test ./...", "go vet ./..."]
allowed_paths: ["internal/adapter/controller/auth.go", "internal/adapter/controller/auth_test.go", "internal/adapter/controller/user.go", "internal/adapter/controller/user_test.go", "cmd/api/api.go"]
requires_changes: true
risk: LOW
complexity: MEDIUM
sources: []
objective_id: "obj-20260826084453-b2260009"
plan_id: "plan-obj-20260826084453-b2260009-r1"
orchestration_id: "orch-obj-20260826084453-b2260009"
node_id: "node-2"
master_task: "obj-20260826084453-b2260009"
orchestration_managed: true
orchestration_dependencies: ["be-golang-app:TASK-006"]
role: "BACKEND"
node_type: "IMPLEMENTATION"
write_conflict_group: "adapter-controller"
context_from: ["node-1"]
skill_assignments: []
---

# Implement HTTP Controllers, Auth Middleware, and Wire API Routes

## Permintaan User

Di project be-golang-app, tolong buatkan fitur Authentication dan Manajemen User.
Fiturnya:
1. Register user baru (nama, email, password) dengan password terenkripsi.
2. Login user (email dan password) yang menghasilkan JWT token.
3. Get Profile user yang sedang login (menggunakan Bearer JWT token).
Sertakan unit test untuk usecase dan controllernya.

Orchestration node: node-2

## Tujuan

Implement HTTP controllers for auth and user endpoints, Bearer JWT authentication middleware, controller unit tests, and route wiring in cmd/api/api.go.

## Scope

- `internal/adapter/controller/auth.go`
- `internal/adapter/controller/auth_test.go`
- `internal/adapter/controller/user.go`
- `internal/adapter/controller/user_test.go`
- `cmd/api/api.go`

## Hasil Yang Diharapkan

Node node-2 memenuhi seluruh acceptance criteria yang disetujui pada plan plan-obj-20260826084453-b2260009-r1.

## Acceptance Criteria

1. HTTP handlers for register and login endpoints are implemented in internal/adapter/controller/auth.go
2. HTTP handler for get profile endpoint and JWT Bearer [REDACTED] are implemented in internal/adapter/controller/user.go
3. Controller unit tests in internal/adapter/controller/auth_test.go and internal/adapter/controller/user_test.go pass
4. Routes and dependencies are wired in cmd/api/api.go
5. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
6. Verification `go test ./...` dan `go vet ./...` berhasil.

## Knowledge Decision

- Classification: `UPDATE`
- Destination: `WIKI`
- Source: [[03-Sources/other/orchestrator-runs/task-007-20260826T085100Z-102b7d58.json]]


## Error Log

Tidak ada error log saat pembuatan task.

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

- [2026-08-26] Task dibuat melalui orchestrator task intake oleh `orchestrator`.

---

## Orchestrator Run Log
- [2026-08-26T08:51:00.766Z] Human `local:sagaino` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-26T08:51:00.978Z] Run `task-007-20260826T085100Z-102b7d58` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-26T08:55:04.867Z] Run `task-007-20260826T085100Z-102b7d58`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-08-26T09:02:45.354Z] Run `task-007-20260826T085100Z-102b7d58`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
