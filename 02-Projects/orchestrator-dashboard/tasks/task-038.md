---
title: "Configure collapsible dropdowns for Objective page sections"
type: task
task_id: TASK-038
project: orchestrator-dashboard
status: DONE
tags: [task, orchestrator-dashboard, orchestrator-intake]
created: 2026-08-24
updated: 2026-08-24
dependencies: []
verification: ["typecheck", "lint", "build"]
allowed_paths: ["src/pages/**", "src/components/**"]
requires_changes: true
risk: LOW
complexity: LOW
sources: []
objective_id: "obj-20260824185632-6160ba13"
plan_id: "plan-obj-20260824185632-6160ba13-r1"
orchestration_id: "orch-obj-20260824185632-6160ba13"
node_id: "node-1"
master_task: "obj-20260824185632-6160ba13"
orchestration_managed: true
orchestration_dependencies: []
role: "FRONTEND"
node_type: "IMPLEMENTATION"
write_conflict_group: null
context_from: []
skill_assignments: []
---

# Configure collapsible dropdowns for Objective page sections

## Permintaan User

di page objective bagian Objective visual & E2E evidence buat default dropdown menjadi tertutup. Dan untuk Delivery certification, Node outcomes & evidence disamakan dengan Objective visual & E2E evidence menggunakan dropdown

Orchestration node: node-1

## Tujuan

Set Objective visual & E2E evidence dropdown to closed by default and refactor Delivery certification and Node outcomes & evidence sections into matching collapsible dropdowns

## Scope

- `src/pages/**`
- `src/components/**`

## Hasil Yang Diharapkan

Node node-1 memenuhi seluruh acceptance criteria yang disetujui pada plan plan-obj-20260824185632-6160ba13-r1.

## Acceptance Criteria

1. Objective visual & E2E evidence dropdown defaults to closed on page render
2. Delivery certification section is converted into a collapsible dropdown matching the evidence dropdown style and defaults to closed
3. Node outcomes & evidence section is converted into a collapsible dropdown matching the evidence dropdown style and defaults to closed
4. All three sections can be toggled open and closed independently
5. Project verification scripts pass (typecheck, lint, build)
6. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
7. Verification `typecheck` dan `lint` dan `build` berhasil.

## Knowledge Decision

- Classification: `PROJECT_ONLY`
- Destination: `PROJECT`
- Source: [[03-Sources/other/orchestrator-runs/task-038-20260824T185711Z-3cb60809.json]]


## Error Log

Tidak ada error log saat pembuatan task.

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

- [2026-08-24] Task dibuat melalui orchestrator task intake oleh `orchestrator`.

---

## Orchestrator Run Log
- [2026-08-24T18:57:11.806Z] Human `local:sagaino` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-24T18:57:11.967Z] Run `task-038-20260824T185711Z-3cb60809` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-24T19:00:28.560Z] Run `task-038-20260824T185711Z-3cb60809`: coding agent, verification, dan Graphify selesai; menunggu human review.

## Knowledge Retrospective

<!-- orchestrator-run:task-038-20260824T185711Z-3cb60809 -->
- Classification: `PROJECT_ONLY`
- Summary: TASK-038 melakukan modifikasi murni internal komponen/halaman/konfigurasi project (orchestrator-dashboard). Diklasifikasikan secara deterministik sebagai PROJECT_ONLY.
- Rationale: Perubahan cakupan file berada di dalam lapisan presentasi/konfigurasi/pengujian internal project tanpa abstraksi generic yang reusable untuk global knowledge vault.
- Source: [[03-Sources/other/orchestrator-runs/task-038-20260824T185711Z-3cb60809.json]]
- [2026-08-24T19:07:04.350Z] Run `task-038-20260824T185711Z-3cb60809`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
