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
│   │   └── controller/               # HTTP Gin Controllers (e.g. order.go, order_test.go)
│   ├── constant/                     # System constants (datetime, errors, etc.)
│   └── core/
│       ├── domain/                   # Pure domain entities
│       │   └── base.go               # BaseEntity (UUID, timestamps)
│       └── usecase/                  # Business logic & DTO interfaces
│           └── <module>/             # e.g. order/ (dto.go, usecase.go, usecase_test.go)
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

---

## 5. Canonical Implementation Blueprint (Pola Standar Pembuatan Modul/API Baru)

Setiap pembuatan modul API baru oleh AI OS wajib mengikuti 4 langkah terstruktur berikut:

### 1. Domain Entity (`internal/core/domain/<entity>.go`)
```go
package domain

type Order struct {
	BaseEntity
	CustomerName string  `gorm:"column:customer_name;type:varchar(255);not null;default:''" json:"customer_name"`
	TotalAmount  float64 `gorm:"column:total_amount;type:decimal(10,2);not null;default:0" json:"total_amount"`
	Status       string  `gorm:"column:status;type:varchar(50);not null;default:'PENDING'" json:"status"`
}
```

### 2. Usecase DTO & Repository Interface (`internal/core/usecase/<module>/dto.go`)
```go
package order

import (
	"context"
	"<module-name>/internal/core/domain"
)

type CreateOrderRequest struct {
	CustomerName string  `json:"customer_name" validate:"required"`
	TotalAmount  float64 `json:"total_amount" validate:"required,min=1"`
}

type OrderRepository interface {
	Store(ctx context.Context, data domain.Order) (domain.Order, error)
	FindAll(ctx context.Context) ([]domain.Order, error)
	FindOneByID(ctx context.Context, id interface{}) (domain.Order, error)
}

type Usecase interface {
	Create(ctx context.Context, req CreateOrderRequest) (domain.Order, error)
	GetAll(ctx context.Context) ([]domain.Order, error)
}
```

### 3. Usecase Business Logic (`internal/core/usecase/<module>/usecase.go`)
```go
package order

import (
	"context"
	"<module-name>/internal/core/domain"
	"<module-name>/pkg/db"
	"<module-name>/shared/base"
)

type orderUsecase struct {
	base.Port
	repo OrderRepository
}

func NewUsecase(port base.Port) Usecase {
	return &orderUsecase{
		Port: port,
		repo: db.NewRepository[domain.Order](port.DB()),
	}
}

func (u *orderUsecase) Create(ctx context.Context, req CreateOrderRequest) (domain.Order, error) {
	data := domain.Order{
		CustomerName: req.CustomerName,
		TotalAmount:  req.TotalAmount,
		Status:       "PENDING",
	}
	return u.repo.Store(ctx, data)
}

func (u *orderUsecase) GetAll(ctx context.Context) ([]domain.Order, error) {
	return u.repo.FindAll(ctx)
}
```

### 4. Adapter HTTP Controller (`internal/adapter/controller/<entity>.go`)
```go
package controller

import (
	"net/http"
	"github.com/gin-gonic/gin"
	orderUc "<module-name>/internal/core/usecase/order"
	"<module-name>/shared/base"
	"<module-name>/shared/payload"
	"gorm.io/gorm"
)

type OrderController struct {
	base.BaseController
	uc orderUc.Usecase
}

func NewOrderController(dbConn *gorm.DB, port base.Port, ctrl base.BaseController) *OrderController {
	return &OrderController{
		BaseController: ctrl,
		uc:             orderUc.NewUsecase(port),
	}
}

func (ctrl *OrderController) Route(r *gin.RouterGroup) {
	g := r.Group("/orders")
	g.POST("", ctrl.Create)
	g.GET("", ctrl.GetAll)
}

func (ctrl *OrderController) Create(c *gin.Context) {
	var req orderUc.CreateOrderRequest
	if errs := ctrl.Enigma.BindAndValidate(c, &req); len(errs) > 0 {
		c.JSON(http.StatusBadRequest, payload.DefaultInvalidInputFormResponse(errs))
		return
	}
	result, err := ctrl.uc.Create(c.Request.Context(), req)
	ctrl.Mapper.NewResponse(c, payload.NewSuccessResponse(result, "OrderCreated"), err)
}

func (ctrl *OrderController) GetAll(c *gin.Context) {
	result, err := ctrl.uc.GetAll(c.Request.Context())
	ctrl.Mapper.NewResponse(c, payload.NewSuccessResponse(result, "OrdersRetrieved"), err)
}
```

### 5. Composition Root & AutoMigrate (`cmd/api/api.go`)
```go
// 1. Daftarkan Controller ke Engine Gin
start.Register(func(dbConn *gorm.DB, port base.Port, ctrl base.BaseController) api.Router {
	return controller.NewOrderController(dbConn, port, ctrl)
})

// 2. Daftarkan Entity ke GORM AutoMigrate
start.DB().AutoMigrate(&domain.Order{})
```

---

## 6. Related Knowledge

- [[01-Knowledge/patterns/backend/modular-clean-skeleton-composition-root-engine.md]]
- [[01-Knowledge/patterns/backend/structured-domain-error-hierarchy-i18n-response-mapping-pattern.md]]
- [[01-Knowledge/patterns/backend/declarative-route-registration-context-enriching-guard-pipeline.md]]
- [[01-Knowledge/patterns/backend/go-gorm-generic-repository-dynamic-expression-builder-pattern.md]]

