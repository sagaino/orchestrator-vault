---
title: Personal AI Orchestrator
type: project
project_id: personal-ai-orchestrator
project_type: orchestrator-system
self_update: guarded
requires_full_regression: true
requires_runtime_backup: true
requires_health_check: true
requires_human_accept: true
allow_self_accept: false
repository: /Users/sagaino/Documents/personal-ai-orchestrator
agent: agy
graphify: true
graphify_output: /Users/sagaino/Documents/personal-ai-orchestrator/graphify-out/graph.json
verification_defaults: [test]
tags: [project, orchestrator, system-core]
created: 2026-08-16
updated: 2026-08-16
sources: ["[[01-Knowledge/decisions/personal-ai-orchestrator-product-roadmap.md]]", "System Project Policy"]
---

# Personal AI Orchestrator

## Role in the System
Personal AI Orchestrator is the autonomous control plane and execution engine that coordinates coding agents, git worktrees, verification gates, telemetry, and Wiki knowledge synchronization.

## System Project Policy
- Project Type: `orchestrator-system`
- Self Update: `guarded` (requires human review, runtime backup, queue drain, and automatic rollback failure drill)
- Self Accept: `false` (prohibited from self-approving or self-completing without human approval)
- Verification Default: `npm run test` (runs unit tests and full smoke regression suite)

## Repository
- Repository: `/Users/sagaino/Documents/personal-ai-orchestrator`
- Coding agent: `agy`
- Graphify enabled: `true`
- Graphify output: `/Users/sagaino/Documents/personal-ai-orchestrator/graphify-out/graph.json`

## Task Queue
- Tasks: `02-Projects/personal-ai-orchestrator/tasks/`
