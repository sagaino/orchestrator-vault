---
title: "Unified Port & Base Controller Dependency Hub"
type: pattern
tags: [pattern, backend, golang, dependency-injection, ports-and-adapters, cross-cutting-concerns]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787128418632-596cfbe7
sources: ["Harvest 1787128418632 596cfbe7.json"]
---

# Unified Port & Base Controller Dependency Hub

Unified Port & Base Controller Dependency Hub untuk manajemen dependency injection lintas layer di Go.

## 1. Overview & Architecture

Pola Unified Port & Base Controller Dependency Hub memecahkan masalah kompleksitas dependency injection di Golang dengan mengelompokkan cross-cutting services ke dalam container terpadu berbasis interface (`base.Port` untuk domain/use case dan `base.BaseController` untuk transport/HTTP adapter).

## 2. Implementation & Code Structure

shared/
  └── base/
      ├── port.go                    <-- Port & BaseController Aggregation Hub
      └── iam_module.go              <-- Auth contract adapter
pkg/
  ├── cache/                         <-- Cache client interface
  ├── clock/                         <-- Clock & Timezone interface
  ├── localerror/                    <-- Standardized Error handler
  ├── mapper/                        <-- Standardized API Response mapper
  └── middleware/                    <-- Enigma Validator, Idempotent, Sentry

## 3. Key Implementation Points

- Struktur `base.Port` mengagregasikan layanan utilitas domain (Clock, Environment, Security, Cache, Error Handler, Mailing, Key Generator).
- Struktur `base.BaseController` mengagregasikan fungsionalitas HTTP helper (Validation Enigma, Response Mapper, Idempotency Middleware, Security).
- Controller dan Use Case meng-embed struct ini secara langsung untuk akses metode seragam tanpa kode redundan.

## 4. Code Examples

### Definisi Port & BaseController sebagai cross-cutting dependency aggregation hub

```go
// shared/base/port.go
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
	"gorm.io/gorm"
)

type Port struct {
	Security   Security
	ErrHandler ErrHandler
	Cache      Cache
	Env        Environment
	Davinci    Generator
	Mailing    Mailing
	Clock      Clock
}

func NewPort(dbConn *gorm.DB, dbCache cache.DbClient, zero *logger.ReZero) Port {
	return Port{
		Security:   NewAuth(dbConn, dbCache),
		ErrHandler: localerror.NewHandlerError(zero),
		Cache:      &dbCache,
		Env:        environment.NewEnvironment(),
		Davinci:    davinci.DefaultDavinci(),
		Mailing:    mailing.NewConfig(),
		Clock:      clock.Default(),
	}
}

type BaseController struct {
	Mapper   Mapper
	Enigma   Validator
	Security Security
	Idem     Idempotent
}

func NewBaseController(db *gorm.DB, dbCache cache.DbClient) BaseController {
	return BaseController{
		Mapper:   mapper.NewMapper(),
		Enigma:   middleware.NewEnigma(),
		Security: NewAuth(db, dbCache),
		Idem:     middleware.NewIdempotent(dbCache),
	}
}
```

### Controller embedding BaseController dan Use Case embedding Port

```go
// internal/adapter/controller/product.go
package controller

import (
	"base-be-golang/internal/core/usecase/product"
	"base-be-golang/shared/api"
	"base-be-golang/shared/base"
	"base-be-golang/shared/payload"
	"net/http"

	"github.com/gin-gonic/gin"
	"gorm.io/gorm"
)

type ProductController struct {
	base.BaseController
	uc product.Usecase
}

func NewProductController(dbConn *gorm.DB, port base.Port, ctrl base.BaseController) api.Router {
	return ProductController{
		BaseController: ctrl,
		uc:             product.NewUsecase(dbConn, port),
	}
}

func (ctrl ProductController) Create(c *gin.Context) {
	var req product.CreateProductRequest
	if errs := ctrl.Enigma.BindAndValidate(c, &req); errs != nil {
		c.JSON(http.StatusBadRequest, payload.DefaultInvalidInputFormResponse(errs))
		return
	}

	result, err := ctrl.uc.Create(c.Request.Context(), req)
	ctrl.Mapper.NewResponse(c, payload.NewSuccessResponse(result, "ProductCreated"), err)
}
```

## 5. Considerations & Best Practices

- Menghindari 'constructor bloat' pada use case dan controller di mana puluhan service/utility harus di-pass satu per satu.
- Semua dependensi di dalam Port dan BaseController didefinisikan dalam bentuk interface, sehingga mempermudah mock testing untuk setiap unit test.
- Embedding BaseController dan Port memberikan standar konsisten di semua layer controller dan usecase.

## 6. Related Knowledge

- Dependency Injection Hub
- Hexagonal Ports And Adapters

## 7. Source

- Harvest 1787128418632 596cfbe7.json
