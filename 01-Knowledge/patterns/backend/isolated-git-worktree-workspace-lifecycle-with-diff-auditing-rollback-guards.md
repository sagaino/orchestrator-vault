---
title: "Isolated Git Worktree Workspace Lifecycle with Diff Auditing & Rollback Guards"
type: pattern
tags: [pattern, backend, git-worktree, isolation, scope-guard, rollback]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787130041232-c9802673
sources: ["Harvest 1787130041232 C9802673.json"]
---

# Isolated Git Worktree Workspace Lifecycle with Diff Auditing & Rollback Guards

Pola isolasi workspace menggunakan detached Git worktrees, path whitelist scope enforcement, dan rollback backup guards untuk eksekusi agent yang aman.

## 1. Overview & Architecture

Pola isolasi lingkungan eksekusi menggunakan ephemeral Git worktree dan audit jangkauan modifikasi (scope guard). Pola ini memastikan perubahan kode agen terisolasi total dari repositori utama hingga lolos seluruh tahap review dan verifikasi.

## 2. Implementation & Code Structure

src/
├── workspace-manager.mjs # Git worktree isolation, backup snapshot, and rollback
├── review-workflow.mjs   # Scope enforcement, diff extraction, and conflict verification
└── task-workflow.mjs     # Execution pipeline binding workspace to agent lifecycle

## 3. Key Implementation Points

- Isolasi direktori kerja agen ke dalam detached Git worktree independen.
- Terapkan pengawasan ketat `allowed_paths` untuk mencegah modifikasi file di luar lingkup tugas.
- Sediakan snapshot backup otomatis sebelum modifikasi diterapkan ke repositori utama.
- Sertakan guardrail rollback instan saat verifikasi atau gate gagal.

## 4. Code Examples

### Ephemeral detached Git worktree workspace creation and cleanup

```javascript
export function createWorktreeWorkspace({ repoPath, runsRoot, runId }) {
  const worktreeDir = path.join(runsRoot, "workspaces", runId);
  fs.mkdirSync(path.dirname(worktreeDir), { recursive: true });

  // Create detached worktree from current HEAD/baseline
  execFileSync("git", ["worktree", "add", "--detach", worktreeDir, "HEAD"], {
    cwd: repoPath,
    stdio: "pipe",
  });

  return {
    worktreeDir,
    cleanup: () => {
      try {
        execFileSync("git", ["worktree", "remove", "--force", worktreeDir], {
          cwd: repoPath,
          stdio: "pipe",
        });
      } catch {}
    }
  };
}
```

### Scope audit comparing modified repository files against allowed_paths whitelist

```javascript
export function auditWorkspaceScope({ worktreeDir, allowedPaths }) {
  const output = execFileSync("git", ["status", "--porcelain"], {
    cwd: worktreeDir,
    encoding: "utf8",
  });
  
  const modifiedFiles = output
    .split("\n")
    .map(line => line.slice(3).trim())
    .filter(Boolean);

  const violations = modifiedFiles.filter(file => 
    !allowedPaths.some(allowed => file === allowed || file.startsWith(`${allowed}/`))
  );

  return {
    inScope: violations.length === 0,
    modifiedFiles,
    violations,
  };
}
```

## 5. Considerations & Best Practices

- Pastikan git worktree dibersihkan (`git worktree prune / remove`) saat task ditolak atau selesai.
- Lakukan snapshot backup untracked files sebelum menerapkan diff ke main repository.
- Gunakan `git diff --check` dan dry-run patch untuk mendeteksi merge conflicts sebelum final accept.

## 6. Related Knowledge

- Workspace Isolation
- Git Worktree Management

## 7. Source

- Harvest 1787130041232 C9802673.json
