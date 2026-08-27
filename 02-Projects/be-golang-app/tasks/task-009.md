---
title: "Update JWT token expiration to 3 minutes"
type: task
task_id: TASK-009
project: be-golang-app
status: DONE
tags: [task, be-golang-app, orchestrator-intake]
created: 2026-08-26
updated: 2026-08-26
dependencies: []
verification: ["go test ./...", "go vet ./..."]
allowed_paths: ["internal/core/domain/auth.go", "internal/core/usecase/auth/**", "internal/adapter/controller/auth.go", "internal/adapter/controller/auth_test.go"]
requires_changes: true
risk: LOW
complexity: LOW
sources: []
objective_id: "obj-20260826092153-ceb81718"
plan_id: "plan-obj-20260826092153-ceb81718-r1"
orchestration_id: "orch-obj-20260826092153-ceb81718"
node_id: "task-update-jwt-expiration"
master_task: "obj-20260826092153-ceb81718"
orchestration_managed: true
orchestration_dependencies: []
role: "BACKEND"
node_type: "IMPLEMENTATION"
write_conflict_group: null
context_from: []
skill_assignments: []
---

# Update JWT token expiration to 3 minutes

## Permintaan User

expired token jwt di ganti ke 3 menit

Orchestration node: task-update-jwt-expiration

## Tujuan

Change the JWT token expiration time to 3 minutes and update corresponding tests.

## Scope

- `internal/core/domain/auth.go`
- `internal/core/usecase/auth/**`
- `internal/adapter/controller/auth.go`
- `internal/adapter/controller/auth_test.go`

## Hasil Yang Diharapkan

Node task-update-jwt-expiration memenuhi seluruh acceptance criteria yang disetujui pada plan plan-obj-20260826092153-ceb81718-r1.

## Acceptance Criteria

1. JWT token expiration duration is updated to 3 minutes
2. Unit and controller tests pass with the new expiration duration
3. All verification commands succeed
4. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
5. Verification `go test ./...` dan `go vet ./...` berhasil.

## Knowledge Decision

- Classification: `PROJECT_ONLY`
- Destination: `PROJECT`
- Source: [[03-Sources/other/orchestrator-runs/task-009-20260826T092230Z-45e98c26.json]]


## Error Log

Tidak ada error log saat pembuatan task.

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

- [2026-08-26] Task dibuat melalui orchestrator task intake oleh `orchestrator`.

---

## Orchestrator Run Log
- [2026-08-26T09:22:30.861Z] Human `local:sagaino` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-26T09:22:31.034Z] Run `task-009-20260826T092230Z-45e98c26` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-26T09:24:37.580Z] Run `task-009-20260826T092230Z-45e98c26`: coding agent, verification, dan Graphify selesai; menunggu human review.

## Knowledge Retrospective

<!-- orchestrator-run:task-009-20260826T092230Z-45e98c26 -->
- Classification: `PROJECT_ONLY`
- Summary: Perubahan durasi masa berlaku JWT token menjadi 3 menit pada layer domain dan controller auth beserta pembaruan test assertions.
- Rationale: Task TASK-009 berfokus spesifik pada perubahan parameter waktu kadaluarsa (expiration duration) JWT token menjadi 3 menit pada domain auth dan adapter controller di be-golang-app. Perubahan ini bersifat konfiguratif dan murni implementasi spesifik proyek tanpa adanya architectural pattern atau idiom reusable baru.
- Source: [[03-Sources/other/orchestrator-runs/task-009-20260826T092230Z-45e98c26.json]]
- [2026-08-26T09:28:49.052Z] Run `task-009-20260826T092230Z-45e98c26`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
