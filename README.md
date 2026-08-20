# Personal AI Software Engineering System

This Obsidian vault is the control center for a persistent LLM Wiki and project task system for frontend development.

## Architecture

- `01-Knowledge/`: global reusable concepts, patterns, snippets, decisions, and debugging knowledge.
- `02-Projects/`: project metadata and task queues. Each task resolves to a real repository through `project-registry.md`.
- `03-Sources/`: immutable raw sources and local assets.
- `04-Inbox/`: unprocessed notes, clippings, and incoming material.
- `05-Knowledge-Candidates/`: uncertain knowledge awaiting promotion.
- `index.md`: catalog of managed Wiki pages.
- `project-registry.md`: project-to-repository mapping.
- `wiki-log.md`: chronological append-only activity log.
- `AGENTS.md`: schema and operating rules.

## Core Loop

```text
Source → Ingest → Knowledge Extraction → Wiki
                         ↓
Task → Retrieval → Graphify-aware Agent → Implementation
                         ↓
             Retrospective → Wiki Sync
```

## Daily Workflow

1. For a new repository, ask the orchestrator to use `add-project existing` or `add-project new`; use `remove-project` to unregister and archive a project without deleting its repository or knowledge. If that inactive archive must leave Obsidian, use orchestrator-owned `purge-project-archive --confirm` so it is moved to recoverable run quarantine and audited instead of deleted manually.
2. Put raw material in `03-Sources/` or temporary material in `04-Inbox/`.
3. Ingest it into the Wiki and update `index.md` and `wiki-log.md`.
4. Send the request and project name to the orchestrator; include explicit execution intent such as “kerjakan” when it should start immediately.
5. Let conversational intake create the canonical Wiki task, choose verification and narrow `allowed_paths`, classify risk, update the index/log, and enqueue execution.
6. Let the bounded worker pool process different projects in parallel, with one isolated Git worktree per task; tasks from the same project remain queued until the active review is accepted or rejected.
7. Let the orchestrator's native `Personal AI Orchestrator` notifier alert you when review, recovery, failure, or Candidate decisions need attention; use `notifications` if a desktop banner was missed.
8. Ask the orchestrator for status or review by task ID—or simply the latest task—without opening run manifests; the response includes compact per-stage token and duration telemetry.
9. Use `preview <task-id>` to open the isolated review worktree in VS Code, then run `npm run dev` manually when visual inspection is needed. The main repository remains unchanged.
10. Use `request-changes <task-id> --reason <feedback>` when the implementation should be revised instead of discarded. It retains the same workspace and agent conversation, repeats all technical gates, and currently permits unlimited review iterations.
11. Use `telemetry` only when detailed call-level or cross-project usage analysis is needed; warning thresholds are observability-only.
12. Use `accept` or `reject` as the final task decision. `accept` checks for conflicts, applies and reverifies the worktree diff in the main repository, then performs knowledge routing, Wiki Sync, and completion. `reject` archives the audit diff and discards the worktree without changing the main repository.
13. Run `knowledge-health` when you need a Wiki quality report; use `--fix-safe` only for automatic index maintenance.
14. Review low-confidence Candidate knowledge through `knowledge-review`; use `promote-knowledge` or `reject-knowledge` without editing the Vault manually.

New-project onboarding uses a versioned deterministic template after pinned Shadcn initialization. The normal path consumes zero AI tokens; one error-focused, scope-audited agent fallback is available only when deterministic verification fails and can be disabled completely. Dependency preflight, live progress, sensitive `.env` protection, rollback, Git, and Graphify remain orchestrator-owned.

Direct task editing and the managed watcher remain available for advanced operation or recovery. When a task is changed to `READY` manually, the watcher only creates a `PENDING_APPROVAL` manifest and never executes automatically.

Automatic recovery first retries orchestrator-owned checks without AI. If a verified-scope dependency, test/build, or Graphify failure persists, a recovery agent gets at most two attempts by default to repair only `allowed_paths` inside the worktree. Unsafe failures stop as `FAILED`; recovery never bypasses final human `accept`.

Notifications are stored persistently and deduplicated by outcome before delivery through the native macOS `Personal AI Orchestrator` helper is attempted. `ACCEPTED_BY_MACOS` records Notification Center acceptance, not guaranteed visibility through Focus. A missed or failed banner remains visible through `npm run notifications`; notification delivery never changes approval or task state.

Token telemetry uses provider-reported usage for intake, implementation, AI recovery, and retrospective. The default per-run threshold produces warnings only; it never blocks execution or replaces human review. Historical event logs remain immutable and are read on demand for backward-compatible reports.

Knowledge quality is owned by the orchestrator. Its default health check is read-only and reports metadata, wikilink, index, Candidate provenance, duplicate/similarity, contradiction candidates, and project-registry drift. Safe-fix may only add missing index entries and append a lint audit; it never edits source material, rewrites knowledge content, merges, or deletes pages. Candidate promotion is blocked on unverifiable provenance and requires an explicit existing target when a near-duplicate is detected.

Parallel execution is bounded and cross-project only. The daemon runs two workers by default, isolates every job's worktree, manifest, telemetry, and failure, and reserves a project through execution, review, and every `request-changes` cycle. A second task for that project remains queued until the first receives `accept` or `reject`; final approval and Wiki Sync remain per-task human decisions.

See [AGENTS.md](<AGENTS.md>) for the complete schema.
