---
title: "Bounded Asynchronous Daemon Job Queue with Project-Exclusive Mutual Exclusion"
type: pattern
tags: [pattern, backend, concurrency, job-queue, daemon, lock-management]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787130041232-c9802673
sources: ["Harvest 1787130041232 C9802673.json"]
---

# Bounded Asynchronous Daemon Job Queue with Project-Exclusive Mutual Exclusion

Pola daemon worker queue mandiri dengan bounded worker concurrency dan project-exclusive mutual exclusion untuk mencegah konflik git worktree dan race conditions.

## 1. Overview & Architecture

Pola worker pool asinkron dengan kontrol konkurensi terbatas dan mutual exclusion pada tingkat proyek, memastikan tugas dieksekusi secara teratur dan bebas dari race condition pada repositori yang sama.

## 2. Implementation & Code Structure

src/
├── daemon.mjs           # Daemon worker pool and orchestration loop
├── job-queue.mjs        # Persistent FIFO job queue and state transitions
└── run-manager.mjs      # Task execution life-cycle management

## 3. Key Implementation Points

- Batasi eksekusi paralel maksimum (`maxParallelJobs`) untuk mencegah saturasi resource CPU/RAM.
- Terapkan mutual exclusion pada level project (`runningProjects Set`) agar dua task pada repo yang sama tidak saling menimpa.
- Gunakan state machine formal (`QUEUED` -> `RUNNING` -> `REVIEW` -> `DONE`/`FAILED`).
- Bangun mekanisme auto-recovery saat daemon restart untuk mendeteksi job yang terhenti di tengah jalan.

## 4. Code Examples

### Bounded concurrency worker pool with project-exclusive lock enforcement

```javascript
export class DaemonWorkerPool {
  constructor({ runsRoot, maxParallelJobs = 2, pollIntervalMs = 1000 }) {
    this.runsRoot = runsRoot;
    this.maxParallelJobs = maxParallelJobs;
    this.pollIntervalMs = pollIntervalMs;
    this.activeWorkers = new Map(); // jobId -> workerPromise
    this.runningProjects = new Set(); // projectId (exclusive lock)
  }

  canScheduleProject(projectId) {
    return !this.runningProjects.has(projectId);
  }

  async processNext() {
    if (this.activeWorkers.size >= this.maxParallelJobs) return;
    const queuedJobs = listJobs(this.runsRoot).filter(j => j.state === JOB_STATES.QUEUED);
    
    for (const job of queuedJobs) {
      if (this.activeWorkers.size >= this.maxParallelJobs) break;
      if (this.canScheduleProject(job.projectId)) {
        this.runningProjects.add(job.projectId);
        const worker = this.executeJob(job).finally(() => {
          this.activeWorkers.delete(job.jobId);
          this.runningProjects.delete(job.projectId);
        });
        this.activeWorkers.set(job.jobId, worker);
      }
    }
  }
}
```

## 5. Considerations & Best Practices

- Pastikan pembebasan lock project terjadi di blok `finally` untuk mencegah deadlock saat worker crash.
- Worker pool harus mendukung graceful shutdown dengan sinyal SIGINT/SIGTERM untuk menyelesaikan pekerjaan aktif.
- State transisi harus dicatat secara persisten ke disk agar daemon dapat melanjutkan job setelah restart.

## 6. Related Knowledge

- Job Queues
- Concurrency Locks

## 7. Source

- Harvest 1787130041232 C9802673.json
