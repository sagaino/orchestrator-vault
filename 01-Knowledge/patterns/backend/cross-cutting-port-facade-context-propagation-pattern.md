---
supersession: "ACTIVE"
review_by: "2027-02-22"
owner: "knowledge-curator"
confidence: 0.5
provenance_schema: 1
title: "Cross-Cutting Port Facade & Context Propagation Pattern"
type: pattern
tags: [pattern, backend, golang, port-and-adapter, context-propagation, clean-architecture]
created: 2026-08-26
updated: 2026-08-26
orchestrator_run: "harvest-1787715147310-7bd85203"
sources: ["[[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]"]
---

# Cross-Cutting Port Facade & Context Propagation Pattern

Pola agregasi kapabilitas lintas-sektoral seperti caching, timezone context, error tracing, dan generator ke dalam satu struct Port yang dialirkan ke usecase layer.

## 1. Overview & Architecture

Pola ini menggunakan struct facade Port dan BaseController untuk membungkus berbagai layanan lintas sektoral seperti Cache, Clock/Timezone, Error Handling, Token Generator, Validator, dan Response Mapper. Usecase layer meng-embed base.Port sehingga mendapatkan akses kapabilitas tanpa perlu dependency injection terpisah untuk setiap utilitas.

## 2. Implementation & Code Structure

shared/
└── base/
    └── port.go                  # Port & BaseController facades
pkg/
├── cache/cache.go               # Cache driver abstraction (Redis)
├── clock/time.go                # Timezone & Context-aware Clock
├── davinci/davinci.go           # Cryptographic, OTP, and Token generator
└── localerror/util.go           # Structured error handling & Sentry reporter
internal/
└── core/
    └── usecase/
        └── product/
            └── product.go       # Usecase embedding base.Port

## 3. Key Implementation Points

- Pengelompokan kapabilitas cross-cutting ke dalam struct facade base.Port.
- Embedding base.Port ke dalam struct Usecase untuk kemudahan akses logging, caching, dan context time.
- Propagasi context (c.Request.Context()) dari controller Gin ke Usecase hingga Repository.

## 4. Code Examples

### Definisi Port dan BaseController sebagai agregator kapabilitas cross-cutting dan helper input/output.

```go
package base

import (
	"base-be-golang/pkg/cache"
	"base-be-golang/pkg/clock"
	"base-be-golang/pkg/davinci"
	"base-be-golang/pkg/environment"
	"base-be-golang/pkg/localerror"
	"base-be-golang/pkg/logger"
	"base-be-golang/pkg/mailing"
	"base-be-golang/pkg/mapper"
	"base-be-golang/pkg/middleware"
	"time"

	"github.com/gin-gonic/gin"
	"gorm.io/gorm"
)

type Port struct {
	ErrHandler ErrHandler
	Cache      Cache
	Env        Environment
	Davinci    Generator
	Mailing    Mailing
	Clock      Clock
}

func NewPort(dbConn *gorm.DB, dbCache cache.DbClient, zero *logger.ReZero) Port {
	return Port{
		ErrHandler: localerror.NewHandlerError(zero),
		Cache:      &dbCache,
		Env:        environment.NewEnvironment(),
		Davinci:    davinci.DefaultDavinci(),
		Mailing:    mailing.NewConfig(),
		Clock:      clock.Default(),
	}
}

type BaseController struct {
	Mapper Mapper
	Enigma Validator
	Idem   Idempotent
}

func NewBaseController(db *gorm.DB, dbCache cache.DbClient) BaseController {
	return BaseController{
		Mapper: mapper.NewMapper(),
		Enigma: middleware.NewEnigma(),
		Idem:   middleware.NewIdempotent(dbCache),
	}
}
```

### Implementasi Usecase yang menyematkan base.Port untuk mengakses kapabilitas infrastruktur secara konsisten.

```go
package product

import (
	"base-be-golang/internal/core/domain"
	"base-be-golang/pkg/db"
	"base-be-golang/shared/base"
	"context"

	"github.com/google/uuid"
	"gorm.io/gorm"
)

type Usecase struct {
	base.Port
	productRepo ProductRepository
}

func NewUsecase(dbConn *gorm.DB, port base.Port) Usecase {
	return Usecase{
		Port:        port,
		productRepo: db.NewGenericeRepoPointr(dbConn, domain.Product{}),
	}
}

func (u Usecase) Create(ctx context.Context, request CreateProductRequest) (domain.Product, error) {
	product := domain.Product{
		BaseEntity: domain.BaseEntity{
			ID: uuid.New().String(),
		},
		Name:     request.Name,
		Price:    request.GetPrice(),
		Quantity: request.GetQuantity(),
	}
	product.SetCreated("system")

	created, err := u.productRepo.Store(ctx, product)
	if err != nil {
		if u.ErrHandler != nil {
			return domain.Product{}, u.ErrHandler.ErrorReturn(err)
		}
		return domain.Product{}, err
	}

	return created, nil
}
```

## 5. Considerations & Best Practices

- Interface di dalam Port harus tetap murni dan tidak mengekspos tipe dependensi pihak ketiga (misal Redis client atau DB driver).
- Setiap pemanggilan I/O eksternal harus meneruskan context.Context dari HTTP handler untuk memastikan timeout dan cancellation propagation.

## 6. Related Knowledge

- Dependency Injection
- Facade Pattern

## 7. Source

- [[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]
