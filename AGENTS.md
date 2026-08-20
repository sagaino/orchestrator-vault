---
title: Personal AI Software Engineering System Wiki Schema
type: schema
tags: [core-system, rules, global, personal-ai]
created: 2026-08-14
updated: 2026-08-15
sources: ["[[03-Sources/documentation/llm-wiki-concept.md]]", "[[01-Knowledge/patterns/frontend/project-skeleton-template.md]]"]
---

# Personal AI Software Engineering System Wiki Schema

This file defines the operating contract for the Obsidian knowledge layer. It is a schema and workflow guide, not a source document. The system has six responsibilities:

- AI Orchestrator: primary user interface, control plane, task intake, queue, and lifecycle coordinator.
- Obsidian: persistent knowledge and task backend; direct user editing is an advanced or recovery path.
- LLM Wiki: global, reusable knowledge layer.
- Graphify: project-specific code intelligence inside each repository.
- Project repository: source of truth for source code and Graphify output.
- Coding agent: implementation executor.

## 1. Vault Architecture

```text
01-Knowledge/
├── concepts/
├── patterns/
├── snippets/
├── decisions/
└── debugging/

02-Projects/
├── <project-id>/
│   ├── project.md
│   └── tasks/
└── _templates/tasks/

03-Sources/
├── code/
├── articles/
├── documentation/
├── pdf/
├── other/
└── assets/

04-Inbox/
05-Knowledge-Candidates/
index.md
project-registry.md
wiki-log.md
```

### Source of truth rules

1. `03-Sources/` is immutable. Read it, classify it, and link to it; never rewrite or delete the original source during ingestion.
2. `01-Knowledge/` is the persistent, interlinked, LLM-maintained knowledge layer.
3. `02-Projects/` contains project metadata and task instructions, not source code.
4. `graph.json` and other Graphify outputs stay in the project repository. A project page or historical snapshot may link to them but must not replace them.
5. `04-Inbox/` contains unprocessed material.
6. `05-Knowledge-Candidates/` contains low-confidence or not-yet-promoted knowledge.

## 2. Page Metadata

Every Markdown page in the managed Wiki must begin with frontmatter:

```yaml
---
title: Page Title
type: concept | pattern | snippet | decision | debugging | candidate | project | task | task-template | registry | schema | project-snapshot
tags: [tag1, tag2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: ["`03-Sources/code/<source-file>`"]
---
```

Tasks additionally require:

```yaml
task_id: FE-001
project: starter-app
status: BACKLOG
dependencies: []
verification: [typecheck, build]
allowed_paths: [src/path/to/target-file.ts]
requires_changes: true
```

Valid task states are `BACKLOG`, `READY`, `IN_PROGRESS`, `REVIEW`, `DONE`, `FAILED`, and `BLOCKED`.

Implementation tasks should declare the narrowest practical `allowed_paths`. Set `requires_changes: true` when the task must produce a repository diff; use `false` only for explicit read-only audit or verification tasks. The orchestrator compares repository snapshots before and after the coding agent, rejects out-of-scope changes, and rejects empty results when a diff is required.

Conversational intake tasks may include `risk: LOW | MEDIUM | HIGH`. The intake planner must generate the task metadata and canonical body, update `index.md` and `wiki-log.md`, and ask one clarification question instead of creating an unsafe partial task when verification or write scope cannot be determined.

A project page may define `verification_defaults: [typecheck, build]`. Conversational intake must prefer these known-good baseline commands over every available package script, so unrelated project-wide technical debt does not silently expand task scope.

### Task Readiness Gate

Before a task may move from `BACKLOG` to `READY`, run the orchestrator's read-only `validate-task` gate. The gate must reject unresolved template placeholders, incomplete metadata, missing instructions or expected behavior, missing bug symptoms, missing acceptance criteria, unavailable verification scripts, unresolved dependencies, repository mismatches, and unsafe or missing bug target files.

The gate never changes task status. A human approves `BACKLOG → READY` only after the report returns `verdict: PASS`. Watcher and prepare flows must reject a `READY` task that fails the same gate.

## 3. Ingest: Source → Knowledge

When a source is added to `03-Sources/` or promoted from `04-Inbox/`:

1. Read the source completely, including code where applicable.
2. Extract concepts, patterns, implementation details, usage, behavior, caveats, relationships, and reusable knowledge.
3. Search `01-Knowledge/` before creating a page.
4. Classify each finding as `NEW`, `UPDATE`, `PROJECT_ONLY`, or `IGNORE`.
5. Create or update the smallest appropriate page under `concepts/`, `patterns/`, `snippets/`, `decisions/`, or `debugging/`.
6. Put uncertain findings in `05-Knowledge-Candidates/` instead of promoting them immediately.
7. Preserve the original source link in `sources` and add related wikilinks.
8. Update `index.md` and append an `ingest` entry to `wiki-log.md`.

For source code, the knowledge page should normally cover: Overview, Purpose, API, Implementation, Usage, Behavior, Considerations, and Related Knowledge.

## 4. Query: Retrieve, Do Not Bulk Load

For a task or question:

1. Read `index.md` first.
2. Retrieve only relevant Wiki pages.
3. For code tasks, resolve the project via `project-registry.md`.
4. Query Graphify in the project repository; do not load the entire `graph.json`. During an orchestrated run, the orchestrator performs this targeted query and injects the result into coding-agent context.
5. Read only the relevant source files.
6. Cite the relevant Wiki pages and source links in the answer or task notes.
7. Save a durable synthesis to `01-Knowledge/` only when it is reusable beyond the immediate conversation.

## 4A. Project Onboarding

Project onboarding is owned by Personal AI Orchestrator so the user does not need to edit the registry, project page, index, or Wiki log manually.

1. `existing` accepts an external Git repository with `package.json`, runs supported project verification, bootstraps or refreshes project-local Graphify, and only then registers the project in the Vault.
2. `new` uses `01-Knowledge/patterns/frontend/project-skeleton-template.md` as the `frontend-vite` blueprint. It must initialize Vite through Shadcn, run `shadcn add --all`, and apply the version-matched deterministic template before verification.
3. A new baseline must pass `typecheck`, `lint`, and `build`; run `test` when available.
4. New project creation happens in a marked staging directory. Do not publish a partial target repository or partial Wiki registration after failure.
5. Registration is one transaction covering `02-Projects/<project-id>/project.md`, its `tasks/` directory, `project-registry.md`, `index.md`, `wiki-log.md`, and the onboarding audit.
6. Source code and `graphify-out/graph.json` remain in the external repository. The Vault stores metadata and the Graphify pointer only.
7. No project task may start until onboarding reports the project as valid.
8. Removing a project means unregistering it and archiving `project.md` plus task history under `03-Sources/other/removed-projects/`; it must never cascade-delete `01-Knowledge/`, Candidates, immutable run sources, the repository, or Graphify.
9. Project removal must fail while a task, background job, or run is active. Registry, active index links, archive, Wiki log, and removal audit form one rollback-capable transaction.
10. The `frontend-vite` blueprint uses policy version `3`: Shadcn CLI `4.18.0`, TypeScript `~5.9.3`, and deterministic template version `2`. Run a package-lock-only dependency preflight before full installation, permit one deterministic policy retry for dependency resolution errors, and never bypass peer constraints with `--force` or `--legacy-peer-deps`.
11. New-project onboarding must emit live stage progress and a periodic heartbeat during long-running processes. Progress reporting is best-effort and must never alter onboarding state.
12. Initial Git setup must ignore `.env` and `.env.*`, explicitly permit `.env.example`, and inspect staged paths before committing. A staged environment file other than `.env.example` must fail onboarding and trigger rollback.
13. The normal new-project path must not invoke a coding agent. Only a deterministic verification failure may trigger one error-focused fallback agent; its changes must remain inside template-owned files, exclude `src/components/ui/`, pass repeated dependency/verification checks, and be recorded in telemetry. `ORCHESTRATOR_ONBOARDING_AI_FALLBACK=off` must provide a strict zero-AI mode.
14. Treat the complete Shadcn component directory as generated vendor output. ESLint may exclude the generated UI paths, while strict TypeScript compilation remains enabled and project-owned source continues to be linted.
15. `purge-project-archive <project-id> --confirm` is the only controlled exception for moving a removed-project archive out of immutable `03-Sources/`. It must reject active projects, active work, and backlinks outside the removable index entry; move the archive transactionally to the orchestrator run quarantine instead of hard-deleting it; update the index, Wiki log, and audit; and preserve the repository, Graphify, global knowledge, Candidates, and run history.

## 5. Task Workflow

```text
User sends natural-language request to orchestrator
  ↓
Orchestrator resolves project and drafts canonical Wiki task
  ↓
Clarify only if safe scope or verification cannot be determined
  ↓
Task BACKLOG and persistent job queued
  ↓
Readiness Gate PASS
  ↓
The user's explicit “kerjakan/jalankan” request is execution approval
  ↓
Orchestrator promotes READY, prepares, approves, and atomically claims
  ↓
Resolve project via project-registry.md
  ↓
Retrieve relevant Wiki knowledge
  ↓
Create an isolated Git worktree from the current project baseline
  ↓
Orchestrator bootstraps and queries project-local Graphify in the worktree
  ↓
Start coding agent in the worktree with retrieved context
  ↓
Read relevant source files
  ↓
Implement within allowed_paths
  ↓
Orchestrator scope audit and required-diff guard
  ↓
Orchestrator runs test / lint / typecheck / build and refreshes project-local Graphify
  ├─ PASS → continue to REVIEW
  └─ FAIL → deterministic retry without AI
               ├─ PASS → continue to REVIEW
               └─ FAIL → bounded recovery agent repair inside allowed_paths
                            ├─ PASS after repeated scope/verification → continue to REVIEW
                            └─ EXHAUSTED → Task FAILED; preserve workspace for manual recover/retry
  ↓
Task REVIEW
  ↓
Retrospective and knowledge classification
  ↓
Persist deduplicated notification and deliver desktop banner when available
  ↓
Human previews, requests changes, accepts, or rejects through orchestrator using task context
  ↓
preview: open isolated review worktree in VS Code; user may run dev server manually; main repository unchanged
request-changes: keep worktree → Task IN_PROGRESS → agent revision → scope/verification/Graphify/retrospective → Task REVIEW
accept: conflict guard → apply diff → reverify main repository → knowledge approval → Wiki Sync → Task DONE
reject: archive audit diff → discard worktree → Task FAILED
```

The agent must not mark a task `DONE` merely because a process exited successfully. It must record verification, relevant test results, and the knowledge decision first.

The recommended human interface starts with a natural-language request containing explicit execution intent, which creates and queues the task. During review, `preview` opens the isolated worktree in VS Code without starting a dev server or changing the main repository. `request-changes` continues the same worktree and agent conversation, returns the task to `IN_PROGRESS`, then repeats scope audit, dependency reconciliation, verification, Graphify, and retrospective before returning to `REVIEW`. Review-change iterations currently have no hard limit, but every iteration must remain inside `allowed_paths`, produce a new diff, and pass all gates. `accept` authorizes applying the verified worktree diff, the retrospective proposal, knowledge routing, Wiki Sync, and completion. `reject` is final for the run and discards the reviewed worktree without changing the main repository. User-facing status and review resolve the latest task or a task ID; run IDs remain audit details. Internal states remain in the manifest for audit and recovery, and the granular lifecycle commands remain available for advanced operation.

Conversational jobs are persisted under the orchestrator run directory and processed asynchronously by a bounded daemon worker pool. Different projects may run concurrently, but a project is exclusively reserved while one of its jobs is `RUNNING`, `CHANGES_REQUESTED`, or `REVIEW`; later jobs for that project must remain `QUEUED` until `accept` or `reject` releases the reservation. A `request-changes` cycle retains that reservation. The default pool size is `2`, configurable from `1` to `8` through `ORCHESTRATOR_MAX_PARALLEL_JOBS`. Every worker must retain its own run manifest, event log, telemetry, notification, and isolated worktree. Failure in one worker must not cancel or mutate another worker's job. The daemon may execute these jobs because the originating user request records execution approval, but it must never perform final acceptance or completion automatically.

The managed watcher may automatically create or reconcile a `PENDING_APPROVAL` run when a task is changed to `READY` outside conversational intake or `start-task`. That manual-Wiki path must never approve, claim, execute, or complete the run automatically. Deduplication is based on the persisted task fingerprint: identical active runs are reused, while an unexecuted stale manifest becomes `SUPERSEDED` when the task changes before approval.

The default Antigravity adapter configuration is `model: gemini-3.7-flash-high` with `effort: high` for coding execution, automatic recovery, and retrospective. The orchestrator must pass both values explicitly and record them in the run audit; environment overrides are allowed only through `ORCHESTRATOR_AGY_MODEL` and `ORCHESTRATOR_AGY_EFFORT`.

When a task changes project code, consult the project's `project.md` and targeted repository-local Graphify context before editing. In orchestrated execution, create a detached Git worktree under the orchestrator run directory and seed it from the current working-tree baseline so pre-existing tracked and untracked user changes remain visible. The coding agent, review revisions, dependency reconciliation, verification, and Graphify must operate in that isolated workspace. `preview` may open this workspace in VS Code for manual code and visual review, but the user owns starting and stopping `npm run dev`. The main repository must remain unchanged until final `accept` passes the baseline conflict guard; a failed apply must restore the backup. `reject` and replacement `retry` must archive the task diff and remove the worktree. Do not copy `graph.json` into the vault.

Automatic recovery is allowed only after the coding agent has completed and scope audit has established a safe baseline. For dependency, verification, or Graphify failure, retry the orchestrator-owned checks once without AI first. If the failure persists, a recovery agent may repair the existing implementation in the same worktree, using only the selected task, error tail, retrieved knowledge, targeted Graphify context, and `allowed_paths`. Re-run dependency reconciliation, scope audit, verification, and Graphify after every repair. The default maximum is two AI attempts and may be configured from `0` to `3`. Never automatically repair scope violations, missing required diffs, coding-agent failures, or workspace bootstrap failures. Recovery success moves to human `REVIEW`, never directly to `accept` or `DONE`; exhaustion moves to `FAILED` and preserves the workspace for manual `recover` or reviewed `retry`.

The notification layer must persist each deduplicated event under the orchestrator run directory before attempting external delivery. Notify for normal review readiness, successful automatic recovery, failed/exhausted execution, reconciled outcomes after daemon restart, and new Knowledge Candidates requiring a separate decision. On macOS, desktop delivery uses a dedicated native `Personal AI Orchestrator` helper so notification permission and app identity remain explicit; `osascript` is not a supported delivery identity. Desktop delivery is best-effort and must never change task state or convert a successful run into failure. `ACCEPTED_BY_MACOS` means Notification Center accepted the request, while Focus and user settings may still suppress a visible banner. The notification inbox is the durable source; delivery status, read timestamp, task/run/project identity, and suggested command must remain auditable. Notifications may suggest `review`, `status`, or `knowledge-candidates`, but must never execute approval, rejection, completion, or knowledge promotion.

The telemetry layer must record each Antigravity call separately for task intake, implementation, automatic recovery, and retrospective. Use only provider-reported usage and duration; retain input, output, thinking, cache-read, total, model, effort, status, and conversation identity when available. Persist intake telemetry before a run exists, attach it to the run during queue handoff, and retain recovery attempts as separate records. Historical event logs may be read to produce backward-compatible reports but must not be rewritten. Token thresholds are warning-only and must never stop execution, bypass review, alter task state, or estimate monetary cost without authoritative provider pricing.

Every completed task must contain the exact watermark below as the first line under its `## Log Perubahan` section:

```text
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]
```

## 6. Wiki Sync and Promotion

After task execution, the agent must ask:

1. Is there a new concept?
2. Is there a reusable pattern or snippet?
3. Did existing knowledge change?
4. Is there an architectural decision?
5. Is there debugging knowledge?
6. Is the result project-specific only?

Promote only reusable and sufficiently understood knowledge. Do not create duplicate pages such as `use-local-storage-v2.md` when an existing page can be updated.

For orchestrated task knowledge, apply the following routing policy:

1. `NEW` with confidence at or above `0.90`, complete verification, a valid `01-Knowledge/` target, durable content, and an immutable run source may be promoted automatically to Wiki during `accept`.
2. A high-confidence exact title or target duplicate must be treated as `UPDATE`, not a second page.
3. A fuzzy near-duplicate must not be auto-promoted as a new page. Route it to Candidate and require an explicit existing `--target` if human review confirms it should be merged as an `UPDATE`.
4. `NEW` that fails any automatic gate or matches an existing Candidate remains in `05-Knowledge-Candidates/`.
5. `PROJECT_ONLY` stays on the project task and `IGNORE` creates no knowledge page.
6. Candidate promotion requires explicit human approval through the orchestrator's `promote-knowledge` command. `knowledge-review` must verify provenance and similarity first. Rejection uses `reject-knowledge` and must archive the candidate before removal.
7. Promotion and rejection must update `index.md`, append `wiki-log.md`, record approver and timestamp, and create an immutable decision artifact under `03-Sources/other/`.

The default auto-promotion threshold may be overridden only through `ORCHESTRATOR_KNOWLEDGE_AUTO_PROMOTE_CONFIDENCE` with a value from `0` to `1`.

## 7. Lint

The orchestrator's `knowledge-health` command is the canonical quality report. It must cover broken or ambiguous wikilinks, missing/invalid frontmatter, unindexed knowledge, orphan candidates, exact and near duplicates, candidate provenance, stale project metadata, and contradiction candidates.

The default mode is read-only. `--fix-safe` may only add missing knowledge/Candidate entries to `index.md` and append a `lint` entry to `wiki-log.md`. It must never infer semantic frontmatter, edit immutable sources, rewrite knowledge content, resolve links by guessing, merge duplicates, delete pages, or decide contradictions. Findings that require content judgment remain explicit `ERROR` or `WARNING` items for targeted review.

## 8. Editing Policy

- Keep source files immutable.
- Prefer focused edits and reuse existing knowledge pages.
- Do not invent project paths, API contracts, or Graphify data.
- Do not place project-specific implementation details in global knowledge unless they are explicitly marked `PROJECT_ONLY` or are demonstrably reusable.
- Update `index.md` and `wiki-log.md` whenever managed Wiki pages are added, removed, or materially changed.
