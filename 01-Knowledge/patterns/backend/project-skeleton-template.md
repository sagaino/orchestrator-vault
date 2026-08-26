---
supersession: "ACTIVE"
review_by: "2027-02-26"
owner: "knowledge-curator"
confidence: 0.95
orchestrator_run: "onboarding-blueprint-genesis-backend-golang"
provenance_schema: 1
title: Backend Golang Project Skeleton Template & Modular Clean Boilerplate Guide
type: pattern
tags: [pattern, backend, golang, boilerplate, skeleton, project-setup, clean-architecture]
created: 2026-08-26
updated: 2026-08-26
sources: ["[[01-Knowledge/patterns/backend/modular-clean-skeleton-composition-root-engine.md]]", "[[01-Knowledge/patterns/backend/structured-domain-error-hierarchy-i18n-response-mapping-pattern.md]]"]
---

# Backend Golang Project Skeleton Template & Modular Clean Boilerplate Guide

Panduan cetak biru struktur folder dan berkas *starter* standar wajib saat menginisialisasi proyek Backend Golang baru menggunakan Modular Clean Architecture.

## Automation Contract

Section ini adalah kontrak authoritative untuk Personal AI Orchestrator saat menginisialisasi project backend Go baru:

- **Blueprint ID**: `backend-golang`.
- **Blueprint policy version**: `1`; **deterministic template version**: `1`.
- **Toolchain**: Go 1.22+ (Gin framework, GORM ORM, PostgreSQL driver, Redis cache, Enigma validator).
- Project dibuat menggunakan template deterministik dari `templates/backend-golang/`. Normal path tidak memanggil coding agent dan menggunakan `0` token AI.
- **Arsitektur Wajib**:
  - `cmd/api/api.go` (Composition Root, Gin Router group, GORM AutoMigrate, Healthcheck endpoint).
  - `internal/core/domain/` (Pure domain entities, GORM tags, UUID string ID `BaseEntity`).
  - `internal/core/usecase/<module>/` (`dto.go` untuk request/response contracts + `usecase.go` untuk business logic).
  - `internal/adapter/controller/` (`<module>.go` untuk HTTP request mapping + Gin JSON response).
  - `pkg/` (Kumpulan utilities infrastruktur: `db`, `logger`, `mapper`, `middleware`, `localize`, `health`, `localerror`).
  - `shared/` (`base/port.go` dan `payload/response.go`).
  - `resource/message/` (Localization bundles: `en.json`, `id.json`).
- **Standar Response Envelope**:
  ```json
  {
      "code": 200,
      "message": "OK",
      "result": {}
  }
  ```
- **Verification Defaults**: `go test ./...` dan `go vet ./...`.
- Initial Git setup wajib mengabaikan `.env` dan `.env.*`, serta menyediakan `.env.example`.
- Repository baru wajib memiliki project-local `graphify-out/graph.json` sebelum diregistrasikan ke Wiki.

---

## 📁 Pohon Direktori Standar Wajib

```text
<project-root>/
├── cmd/
│   └── api/
│       └── api.go                    # Composition root & AutoMigrate
├── internal/
│   ├── adapter/
│   │   └── controller/               # HTTP Gin Controllers
│   │       ├── product.go
│   │       ├── product_test.go
│   │       ├── contact.go
│   │       └── contact_test.go
│   ├── constant/                     # System constants (datetime, errors, etc.)
│   └── core/
│       ├── domain/                   # Pure domain entities
│       │   ├── base.go
│       │   ├── product.go
│       │   └── contact.go
│       └── usecase/                  # Business logic & DTO interfaces
│           ├── product/
│           │   ├── dto.go
│           │   └── product.go
│           └── contact/
│               ├── dto.go
│               └── usecase.go
├── pkg/                              # Core technical packages & infrastructure
│   ├── db/
│   ├── health/
│   ├── localerror/
│   ├── localize/
│   ├── logger/
│   ├── mapper/
│   └── middleware/
├── resource/
│   └── message/                      # i18n translation JSON bundles
│       ├── en.json
│       └── id.json
├── shared/
│   ├── base/
│   │   └── port.go                   # BasePort facade
│   └── payload/
│       └── response.go               # Standard {code, message, result} envelope
├── .env.example
├── .gitignore
├── Makefile
├── docker-compose.yml
├── go.mod
└── go.sum
```

---

## 🔒 Path Minimum yang Divalidasi Otomatis

```text
cmd/api/api.go
internal/core/domain/base.go
internal/adapter/controller/
pkg/db/
pkg/mapper/errors.go
shared/payload/response.go
go.mod
```

## 7. Related Knowledge

- [[01-Knowledge/patterns/backend/modular-clean-skeleton-composition-root-engine.md]]
- [[01-Knowledge/patterns/backend/structured-domain-error-hierarchy-i18n-response-mapping-pattern.md]]
- [[01-Knowledge/patterns/backend/declarative-route-registration-context-enriching-guard-pipeline.md]]
- [[01-Knowledge/patterns/backend/go-gorm-generic-repository-dynamic-expression-builder-pattern.md]]
