---
title: Project Registry
type: registry
tags: [registry, projects, control-center]
created: 2026-08-14
updated: 2026-08-16
sources: []
---

# Project Registry

The registry resolves an Obsidian task's `project` value to the real repository. Obsidian is the control center; it is not the coding agent's working directory.

| project_id | project page | repository | agent | graphify | graphify output |
|---|---|---|---|---|---|
| `gallery-fmfu` | [[02-Projects/gallery-fmfu/project]] | `/Users/sagaino/ciniru/gallery-fmfu` | `agy` | `true` | `/Users/sagaino/ciniru/gallery-fmfu/graphify-out/graph.json` |
| `starter-app` | [[02-Projects/starter-app/project]] | `/Users/sagaino/ciniru/starter-app` | `agy` | `true` | `/Users/sagaino/ciniru/starter-app/graphify-out/graph.json` |
| `personal-ai-orchestrator` | [[02-Projects/personal-ai-orchestrator/project]] | `/Users/sagaino/Documents/personal-ai-orchestrator` | `agy` | `true` | `/Users/sagaino/Documents/personal-ai-orchestrator/graphify-out/graph.json` |
| `orchestrator-dashboard` | [[02-Projects/orchestrator-dashboard/project]] | `/Users/sagaino/ciniru/orchestrator-dashboard` | `agy` | `true` | `/Users/sagaino/ciniru/orchestrator-dashboard/graphify-out/graph.json` |

## Registry Rules

1. Every active task must contain a `project` value matching a `project_id` here.
2. The runner must fail safely when a project is missing or the repository path does not exist.
3. A project page may document metadata and links, but source code and Graphify output remain in the repository.
4. New projects must be added here before their tasks move to `READY`.
