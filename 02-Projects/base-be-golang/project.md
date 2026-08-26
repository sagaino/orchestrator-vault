---
title: "Base Be Golang"
type: project
project_id: base-be-golang
repository: "/Users/sagaino/belajar/base-be-golang"
agent: agy
graphify: true
graphify_output: "/Users/sagaino/belajar/base-be-golang/graphify-out/graph.json"
verification_defaults: ["go test ./...", "go vet ./..."]
tags: ["project", "backend", "golang"]
created: 2026-08-25
updated: 2026-08-25
sources:
  - "[[01-Knowledge/patterns/backend/modular-clean-skeleton-composition-root-engine.md]]"
  - "[[01-Knowledge/patterns/backend/structured-domain-error-hierarchy-i18n-response-mapping-pattern.md]]"
  - "[[01-Knowledge/patterns/backend/declarative-route-registration-context-enriching-guard-pipeline.md]]"
  - "[[01-Knowledge/patterns/backend/go-gorm-generic-repository-dynamic-expression-builder-pattern.md]]"
---

# Base Be Golang

## Role in the Wiki

Project metadata used by Personal AI Orchestrator. Source code and Graphify output remain in the repository.

## Repository

- Repository: `/Users/sagaino/belajar/base-be-golang`
- Coding agent: `agy`
- Graphify enabled: `true`
- Graphify output: `/Users/sagaino/belajar/base-be-golang/graphify-out/graph.json`
- Verification defaults: `go test ./...`, `go vet ./...`

## Architecture & Knowledge Patterns

- [[01-Knowledge/patterns/backend/modular-clean-skeleton-composition-root-engine.md|Modular Clean Skeleton & Composition Root Engine]]: Composition Root di `cmd/api`, BaseController, dan struktur Modular Clean Architecture (Domain -> Usecase -> Adapter/Controller).
- [[01-Knowledge/patterns/backend/structured-domain-error-hierarchy-i18n-response-mapping-pattern.md|Structured Domain Error Hierarchy & I18n Response Mapping Pattern]]: Standar respons HTTP JSON menggunakan `ctrl.Mapper.NewResponse` dan validasi via `ctrl.Enigma.BindAndValidate`.
- [[01-Knowledge/patterns/backend/declarative-route-registration-context-enriching-guard-pipeline.md|Declarative Route Registration & Context-Enriching Guard Pipeline]]: Registrasi rute Gin fungsional melalui `start.Register(...)` dan middleware pipeline.
- [[01-Knowledge/patterns/backend/go-gorm-generic-repository-dynamic-expression-builder-pattern.md|Go GORM Generic Repository & Dynamic Expression Builder Pattern]]: Pola database GORM, Unit of Work, dan query expression builder.

## Task Queue

- Tasks: `02-Projects/base-be-golang/tasks/`

## Graphify Note

Graphify is refreshed in the repository by the orchestrator. Its output is never copied into the Vault.
