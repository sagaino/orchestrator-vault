---
title: "Test Be Service"
type: project
project_id: test-be-service
repository: "/tmp/test-be-service"
agent: agy
graphify: true
graphify_output: "/tmp/test-be-service/graphify-out/graph.json"
verification_defaults: ["go test ./...", "go vet ./..."]
blueprint: backend-golang
template_version: 1
blueprint_policy_version: 1
template_checksum: "facfe62ede6bfdf3f538c801e80deeba77fafdf2b2622745462064a881130bfe"
scaffold_mode: DETERMINISTIC_TEMPLATE
tags: ["project", "backend", "golang"]
created: 2026-08-26
updated: 2026-08-26
sources: ["[[01-Knowledge/patterns/backend/project-skeleton-template]]"]
---

# Test Be Service

## Role in the Wiki

Project metadata used by Personal AI Orchestrator. Source code and Graphify output remain in the repository.

## Repository

- Repository: `/tmp/test-be-service`
- Coding agent: `agy`
- Graphify enabled: `true`
- Graphify output: `/tmp/test-be-service/graphify-out/graph.json`
- Verification defaults: `go test ./...`, `go vet ./...`
- Scaffold mode: `DETERMINISTIC_TEMPLATE`
- Template version: `1`

## Task Queue

- Tasks: `02-Projects/test-be-service/tasks/`

## Graphify Note

Graphify is refreshed in the repository by the orchestrator. Its output is never copied into the Vault.
