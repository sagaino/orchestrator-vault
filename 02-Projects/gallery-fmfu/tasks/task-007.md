---
title: "Configure conditional media endpoints based on BIB in endpoint.ts"
type: task
task_id: GFM-007
project: gallery-fmfu
status: FAILED
tags: [task, gallery-fmfu, orchestrator-intake]
created: 2026-09-01
updated: 2026-09-01
dependencies: []
verification: ["typecheck", "build"]
allowed_paths: ["src/lib/constant/endpoint.ts"]
requires_changes: true
risk: LOW
complexity: LOW
sources: []
objective_id: "obj-20260901035415-1f7597a2"
plan_id: "plan-obj-20260901035415-1f7597a2-r1"
orchestration_id: "orch-obj-20260901035415-1f7597a2"
node_id: "node-1"
master_task: "obj-20260901035415-1f7597a2"
orchestration_managed: true
orchestration_dependencies: []
role: "FRONTEND"
node_type: "IMPLEMENTATION"
write_conflict_group: null
context_from: []
skill_assignments: ["frontend-vite@1.0.0#bde6d8aa3f18070d43d2124d250edec6622d08e3cb03e4a17eb60979c39bce21"]
---

# Configure conditional media endpoints based on BIB in endpoint.ts

## Permintaan User

buat condition dimana jika BIB adalah DUMMY001 (buat customizeable) maka di src/lib/constant/endpoint.ts untuk POST_MEDIA_ALL, POST_MEDIA_THUMBNAIL, POST_MEDIA_DOWNLOAD menggunakan media-mapping. jika bukan maka menggunakan post-media

Orchestration node: node-1

## Tujuan

Update POST_MEDIA_ALL, POST_MEDIA_THUMBNAIL, and POST_MEDIA_DOWNLOAD in src/lib/constant/endpoint.ts to conditionally use media-mapping when BIB matches DUMMY001 (or customizable dummy list) and post-media otherwise.

## Scope

- `src/lib/constant/endpoint.ts`

## Hasil Yang Diharapkan

Node node-1 memenuhi seluruh acceptance criteria yang disetujui pada plan plan-obj-20260901035415-1f7597a2-r1.

## Acceptance Criteria

1. POST_MEDIA_ALL, POST_MEDIA_THUMBNAIL, and POST_MEDIA_DOWNLOAD point to media-mapping when BIB is DUMMY001 or configured customizable dummy values.
2. Endpoints fallback to post-media when BIB is not a dummy identifier.
3. Code passes typecheck and build without regressions.
4. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
5. Verification `typecheck` dan `build` berhasil.

## Knowledge Decision

Belum ditentukan oleh retrospective orchestrator.

## Error Log

Tidak ada error log saat pembuatan task.

## Log Perubahan

🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]
- [2026-09-01] Task dibuat melalui orchestrator task intake oleh `orchestrator`.

---

## Orchestrator Run Log
- [2026-09-01T03:55:00.725Z] Human `local:sagaino` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-09-01T03:55:01.012Z] Run `gfm-007-20260901T035500Z-5b0db9a1` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-09-01T03:57:03.023Z] Run `gfm-007-20260901T035500Z-5b0db9a1`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-09-01T04:02:49.467Z] Human `local:sagaino` meminta revisi run `gfm-007-20260901T035500Z-5b0db9a1`: saya mau mengganti dari pada menggunakan BIB gunakan name, untuk name bisa di cek dari local storage user
- [2026-09-01T04:03:20.504Z] Run `gfm-007-20260901T035500Z-5b0db9a1`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-09-01T04:04:48.816Z] Run `gfm-007-20260901T035500Z-5b0db9a1`: human review ditolak oleh local:sagaino: Rejected by user
