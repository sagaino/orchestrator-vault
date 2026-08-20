---
title: Personal AI Orchestrator Product Roadmap
type: decision
tags: [personal-ai, orchestrator, dashboard, token-optimization, knowledge-workflow, roadmap]
created: 2026-08-16
updated: 2026-08-16
sources: ["[[AGENTS]]", "[[README]]"]
---

# Personal AI Orchestrator Product Roadmap

## Status Keputusan

Roadmap versi 1 ini disetujui sebagai baseline pengembangan Personal AI Orchestrator. Markdown ini adalah dokumen canonical yang dapat diperbarui secara terkontrol. PDF versi 1 adalah snapshot immutable dari keputusan awal dan tidak mengikuti revisi Markdown setelah snapshot dibuat.

## Visi Produk

Personal AI Orchestrator akan berkembang dari command-line control plane menjadi **Personal AI Software Engineering Platform** lokal. Dashboard menjadi interface utama untuk memberikan task, memantau pekerjaan, melakukan review, meminta revisi, menyetujui code, serta mengelola pembaruan knowledge. CLI tetap tersedia untuk advanced operation dan recovery.

Pengalaman utama user:

```text
Pilih project
  -> tulis task atau lampirkan desain
  -> klik Kerjakan
  -> pantau progress
  -> review code, UI, verification, dan knowledge
  -> Accept, Request Changes, atau Reject
```

User tidak perlu mengingat task ID, run ID, lokasi worktree, nama state internal, Graphify command, atau command lifecycle untuk flow normal.

## Prinsip Arsitektur

```text
CLI ---------+
             +--> Orchestrator Core --> Queue / Lifecycle / Security
Dashboard ---+             |
                           +--> Coding Agent
                           +--> Graphify
                           +--> Verification
                           +--> Isolated Git Worktree
                                      |
                                Human Accept
                                      |
                              Main Repository

Obsidian Wiki <-- Retrospective / Knowledge Approval / Wiki Sync
```

Prinsip yang tidak boleh berubah:

1. Repository utama tidak berubah sebelum human `accept`.
2. Dashboard dan CLI menggunakan Orchestrator Core yang sama.
3. Dashboard tidak mengedit manifest, task Wiki, atau repository secara langsung.
4. Agent hanya bekerja di isolated worktree dan di dalam `allowed_paths`.
5. Code approval dan knowledge decision tetap dapat diaudit.
6. `03-Sources/` tetap immutable setelah source disimpan.
7. Jumlah `request-changes` tidak dibatasi, tetapi context dikompaksi agar efisien.
8. Orchestrator tidak boleh menyetujui, mengaktifkan, atau menyelesaikan perubahan runtime dirinya sendiri tanpa human approval.
9. Source repository, stable runtime, dan updater/supervisor harus dipisahkan agar self-update dapat di-rollback.

## Hasil Akhir yang Ditargetkan

Platform akhir menggabungkan:

- AI coding agent orchestration.
- Project dan task management lokal.
- Approval-gated code review.
- Visual QA untuk UI slicing.
- Verification dan recovery automation.
- Token-aware model dan context routing.
- Persistent engineering knowledge dengan human approval.

Setiap task menghasilkan dua outcome: project membaik melalui perubahan code yang diverifikasi, dan sistem menjadi lebih pintar melalui knowledge yang diklasifikasikan serta disetujui.

## Strategi Self-Development dan Bootstrap

Orchestrator ditargetkan mampu mengembangkan dirinya sendiri, tetapi tidak langsung pada kondisi awal. Saat keputusan ini dibuat, source Personal AI Orchestrator belum menjadi Git repository dan belum terdaftar sebagai project yang dapat dikelola orchestrator. Fase bootstrap harus diawasi dari luar runtime yang sedang diubah.

### Pembagian Kepemilikan

- Fase 0 dan fondasi Fase 1 dikerjakan dengan ChatGPT Work/Codex sebagai bootstrap supervisor.
- Setelah guarded self-update, restart, health check, dan rollback terbukti aman, Fase 2 sampai Fase 4 dapat dikerjakan oleh orchestrator melalui task normal.
- Perubahan lifecycle, manifest schema, updater, security boundary, atau rollback tetap disarankan mendapat second review eksternal.
- ChatGPT Work/Codex tetap menjadi recovery path ketika orchestrator atau dashboard tidak dapat berjalan.

### System Project Policy

Setelah fondasi aman, Personal AI Orchestrator didaftarkan sebagai project khusus dengan policy minimum:

```yaml
project_type: orchestrator-system
self_update: guarded
requires_full_regression: true
requires_runtime_backup: true
requires_health_check: true
requires_human_accept: true
allow_self_accept: false
```

### Guarded Self-Update Flow

```text
Task orchestrator
  -> isolated Git worktree
  -> coding agent
  -> security, unit, integration, dan full regression
  -> human code dan knowledge review
  -> human accept
  -> build release candidate
  -> drain active queue
  -> backup stable runtime
  -> stop old runtime
  -> activate release candidate
  -> health check
       PASS -> publish version dan complete task
       FAIL -> rollback stable runtime dan preserve diagnostics
```

Source code yang diterima tidak langsung menimpa proses aktif. Updater/supervisor berada di luar runtime utama sehingga tetap dapat melakukan rollback jika daemon baru gagal start, gagal membaca manifest, atau tidak menghasilkan heartbeat sehat.

## Fase 0 - Safety, Data Accuracy, dan Fondasi

### Tujuan

Memastikan dashboard dibangun di atas orchestrator yang aman, konsisten, dan memiliki telemetry akurat.

### Scope

- Blokir `.env`, `.env.*`, private key, credential, certificate, dan secret lain dari worktree agent; `.env.example` tetap boleh digunakan.
- Gunakan tracked dan untracked non-ignored file sebagai sumber workspace, lalu terapkan deny-pattern tambahan.
- Validasi symlink agar tidak keluar repository.
- Simpan nilai verification script saat claim dan blokir perubahan script yang tidak diotorisasi.
- Deduplicate telemetry explicit dan event-log inference.
- Jadikan `request-changes` transactionally rollback-safe untuk task, manifest, lock, dan job.
- Tambahkan schema validation dasar untuk manifest dan job.
- Pisahkan Orchestrator Core dari handler CLI.
- Inisialisasi Git repository orchestrator, tetapkan baseline terverifikasi, dan dokumentasikan release boundary.
- Definisikan system-project policy, runtime compatibility contract, dan desain external updater/supervisor.
- Tambahkan unit test security, telemetry, dan transaction failure.

### Token Quick Wins

- Tambahkan konfigurasi model dan effort per stage.
- Gunakan model lebih ringan untuk intake dan retrospective normal.
- Pertahankan model kuat untuk implementation dan recovery kompleks.
- Gunakan event-log inference hanya jika explicit telemetry tidak tersedia.

### Definition of Done

- Secret test fixtures tidak pernah masuk workspace agent.
- Verification script yang berubah tidak pernah dieksekusi tanpa otorisasi eksplisit.
- Satu invocation menghasilkan satu telemetry record.
- Simulated write failure mengembalikan task, manifest, lock, dan job ke keadaan konsisten.
- Source orchestrator memiliki Git baseline terverifikasi dan self-hosting threat model yang disetujui.
- Regression suite tetap lulus.

## Fase 1 - Local Orchestrator API

### Tujuan

Menyediakan jembatan resmi dan aman antara dashboard dan Orchestrator Core.

### Read-only API

- Daftar dan detail project.
- Task, job, run, progress, dan event.
- Verification, Graphify, workspace, dan telemetry summary.
- Notification dan Knowledge Candidate.

### Mutation API

- Membuat dan menjalankan task.
- Membuka isolated workspace.
- `request-changes`, `accept`, dan `reject`.
- `recover` dan `retry`.
- Approve, merge, project-only, atau reject knowledge.
- Menandai notification telah dibaca.

### Guardrail

- Bind hanya ke `127.0.0.1`.
- Gunakan local access token dan origin validation.
- Tindakan destructive membutuhkan confirmation.
- Mutation menggunakan idempotency key.
- Progress dikirim melalui Server-Sent Events.
- Semua endpoint memanggil lifecycle service yang sama dengan CLI.
- Self-update endpoint tidak mengaktifkan runtime secara langsung; ia hanya menyiapkan release candidate untuk human approval dan external updater.

### Definition of Done

- Seluruh operasi normal tersedia melalui API.
- CLI dan API menghasilkan state, guard, serta audit yang sama.
- Duplicate request tidak menciptakan duplicate task atau approval.
- API tidak membuka akses jaringan publik secara default.
- Guarded updater dapat melakukan queue drain, backup, restart, health check, dan automatic rollback melalui failure drill.
- Personal AI Orchestrator dapat didaftarkan sebagai `orchestrator-system` tanpa kemampuan self-accept.

## Fase 2 - Dashboard Core dan Knowledge Center

### Tujuan

Memungkinkan user menyelesaikan workflow normal tanpa terminal.

### Overview

- Daemon health.
- Running, queued, review, failed, dan done.
- Project aktif dan notification belum dibaca.
- Ringkasan token dan action required.

### Project Center

- Repository, Graphify health, dan verification defaults.
- Task aktif dan history.
- Asset desain project.
- Aksi membuka project di VS Code.

### Task Composer

- Pilih project.
- Tulis natural-language request.
- Pilih feature, bug, refactor, audit, atau UI slicing.
- Lampirkan screenshot atau asset.
- Pilih langsung kerjakan atau review plan dahulu.
- Preview tujuan, acceptance criteria, allowed paths, verification, dan risk.

### Live Progress

- Stage aktif, durasi, agent status, verification progress, dan token per tahap.
- Error diterjemahkan menjadi tindakan yang aman dan mudah dipahami.

### Review Center

- Ringkasan implementasi dan daftar file berubah.
- Code diff, verification, Graphify, workspace, telemetry, dan revision history.
- Aksi `Open VS Code`, `Request Changes`, `Accept`, dan `Reject`.
- Default view sederhana; detail teknis tersedia melalui bagian Advanced.

### Knowledge Center

- Proposal `NEW`, `UPDATE`, `PROJECT_ONLY`, atau `IGNORE`.
- Confidence, provenance, target Wiki, dan similar/duplicate knowledge.
- Preview perubahan knowledge sebelum task diterima.
- Aksi `Approve as New`, `Merge into Existing`, `Keep Project Only`, dan `Reject Knowledge`.
- Antrean Candidate dan riwayat keputusan knowledge.

### Definition of Done

Tanpa command terminal, user dapat membuat task, menjalankannya, memantau progress, membuka review, meminta revisi, menentukan knowledge, serta menerima atau menolak hasil. Semua keputusan tercatat di audit yang sama dengan CLI.

## Fase 3 - Visual QA untuk UI Slicing

### Tujuan

Menjadikan UI slicing sebagai workflow first-class dengan bukti visual yang dapat direview.

### Asset Convention

```text
03-Sources/assets/<project-id>/<task-id>/
```

Asset yang sudah masuk sebagai source tidak ditimpa. Revisi referensi disimpan sebagai source baru yang dapat ditautkan ke task.

### Scope

- Dev-server lifecycle sementara yang dimiliki orchestrator.
- Route atau URL target per task.
- Screenshot Playwright untuk desktop, tablet, dan mobile.
- Side-by-side comparison dan visual difference overlay.
- Deteksi overflow, horizontal scroll, serta elemen terpotong.
- Accessibility scan dasar.
- Screenshot sebelum dan sesudah `request-changes`.
- Review artifacts di bawah run directory, bukan di repository utama.
- Feedback berbasis area visual.

### Definition of Done

Task UI slicing dapat menerima referensi, menghasilkan screenshot multi-viewport, menampilkan visual comparison, menerima feedback revisi, menghasilkan bukti visual baru, dan tetap memerlukan human approval sebelum apply.

## Fase 4 - Advanced Token Optimization dan Stabilization

### Tujuan

Menurunkan penggunaan token tanpa menurunkan kualitas implementasi, verification, atau pembaruan knowledge.

### Intake

- Deterministic fast path untuk task sederhana.
- AI planner hanya ketika scope atau acceptance criteria ambigu.
- Reuse verification defaults dan metadata project.
- Hindari memuat seluruh task history.

### Implementation

- Retrieval berdasarkan target dan `allowed_paths`.
- Graphify context dengan budget yang sesuai kompleksitas.
- Batasi knowledge pages dan source yang tidak relevan.
- Model routing berdasarkan risk, scope, dan kompleksitas.

### Request Changes

- Kirim hanya feedback, ringkasan state, diff terbaru, file relevan, dan Graphify delta.
- Buat context checkpoint serta ringkasan revision history.
- Tidak menerapkan hard limit jumlah revisi.

### Retrospective dan Knowledge

- Retrospective tidak dihapus.
- Gunakan deterministic classification untuk perubahan yang jelas `PROJECT_ONLY`.
- Gunakan model ringan untuk proposal normal dan model kuat untuk keputusan arsitektur kompleks.
- Human override tetap tersedia di Knowledge Center.

### Reliability dan Maintainability

- Job lease, heartbeat, dan crash recovery.
- Manifest migration serta corrupt-state quarantine.
- Pecah test menjadi unit, integration, security, lifecycle, dan end-to-end.
- Modularisasi executor, onboarding, knowledge workflow, dan CLI handlers.
- Dashboard packaging dan auto-start bersama daemon.

### Definition of Done

- Median token per benchmark task turun minimal 30% dari baseline telemetry yang sudah dideduplicate.
- Verification pass rate tidak menurun.
- Setiap task memiliki knowledge decision eksplisit.
- Tidak ada code atau knowledge yang diterapkan tanpa policy approval yang sesuai.

## Urutan dan Dependency

```text
Fase 0: Safety dan data accuracy
  -> Fase 1: Local API
  -> Fase 2: Dashboard dan Knowledge Center
  -> Fase 3: Visual QA
  -> Fase 4: Advanced optimization dan stabilization
```

Token optimization dilakukan dalam dua gelombang: perbaikan measurement dan model routing pada Fase 0, lalu context compaction serta deterministic routing pada Fase 4.

## Kepemilikan Pengembangan per Fase

```text
Fase 0 dan fondasi Fase 1
  -> ChatGPT Work/Codex atau Antigravity sebagai bootstrap supervisor

Guarded self-update dan rollback drill lulus
  -> Human mengaktifkan orchestrator-system

Fase 2 sampai Fase 4
  -> Orchestrator menjadi primary implementation flow
  -> ChatGPT Work/Codex menjadi second reviewer dan recovery path
```

Peralihan kepemilikan tidak terjadi hanya karena test code lulus. Gate self-hosting juga membutuhkan stable runtime backup, external updater, queue drain, restart, heartbeat health check, rollback drill, dan larangan self-accept yang telah diverifikasi.

## Checklist Approval per Fase

Sebelum melanjutkan ke fase berikutnya:

1. Seluruh Definition of Done fase aktif harus lulus.
2. Scope audit dan regression suite harus lulus.
3. Perubahan kontrak diperbarui di Wiki.
4. Knowledge hasil fase diklasifikasikan dan disetujui.
5. Human menerima hasil fase melalui orchestrator.
6. Untuk perubahan runtime orchestrator, release candidate harus lulus compatibility check, health check, dan rollback drill.

## Success Metrics Produk

- Flow normal dapat diselesaikan tanpa terminal.
- Repository utama tidak pernah berubah sebelum approval.
- Zero known secret exposure ke agent workspace.
- Satu invocation tercatat tepat satu kali.
- Median token benchmark turun minimal 30%.
- Semua task memiliki code decision dan knowledge decision yang auditable.
- UI slicing memiliki bukti visual desktop, tablet, dan mobile.
- Wiki index, provenance, dan health tetap valid setelah knowledge sync.
- Orchestrator dapat menjalankan guarded self-update tanpa self-approval dan dapat rollback ke stable runtime setelah simulated startup failure.

## Decision Log

- 2026-08-16: Dashboard dipilih sebagai interface utama; CLI tetap menjadi advanced dan recovery interface.
- 2026-08-16: Knowledge Center dimasukkan ke Dashboard Core, bukan ditunda sebagai add-on.
- 2026-08-16: Token optimization dibagi menjadi quick wins dan advanced optimization.
- 2026-08-16: `request-changes` tetap tanpa hard limit; efisiensi dijaga melalui context compaction.
- 2026-08-16: Markdown ditetapkan sebagai canonical roadmap dan PDF versi 1 sebagai immutable approved snapshot.
- 2026-08-16: Fase 0 dan fondasi Fase 1 menggunakan external bootstrap supervision; Fase 2 sampai Fase 4 dapat dipindahkan ke orchestrator setelah guarded self-update dan rollback drill lulus.
- 2026-08-16: Self-development menggunakan project policy khusus, external updater, human accept, runtime health check, dan larangan self-approval.

## Related Knowledge

- [[AGENTS|Personal AI Software Engineering System Wiki Schema]]
- [[README|Personal AI Software Engineering System]]
- [[01-Knowledge/patterns/frontend/workflow-and-verification|Workflow & Verification]]
- [[01-Knowledge/concepts/definition-of-done|Definition of Done]]
