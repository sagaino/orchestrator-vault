---
title: "Modular Clean Skeleton & Composition Root Engine"
type: pattern
tags: [pattern, backend, architecture, golang, clean-architecture, composition-root]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787128418632-596cfbe7
sources: ["Harvest 1787128418632 596cfbe7.json"]
---

# Modular Clean Skeleton & Composition Root Engine

Modular Clean Architecture Skeleton & Composition Root Engine untuk Golang backend application.

## 1. Overview & Architecture

Pola Composition Root & Modular Clean Architecture memisahkan siklus hidup aplikasi (bootstrap, migrasi database, lifecycle HTTP server) dari logika bisnis dan rute HTTP. Struktur ini mengelompokkan kode ke dalam Clean/Hexagonal layers (Domain -> Usecase -> Adapter/Controller -> Infrastructure) serta memungkinkan plug-and-play modul secara independen melalui interface pendaftaran Router.

## 2. Implementation & Code Structure

cmd/
  └── api/
      └── api.go                     <-- Composition Root & Wiring
shared/
  ├── api/
  │   ├── api.go                     <-- Engine struct, Router interface & lifecycle
  │   ├── default.go                 <-- Default engine constructor & middleware init
  │   └── register_api.go            <-- Functional registration hook
  └── base/
      └── port.go                    <-- Port & BaseController definitions
internal/
  ├── adapter/
  │   └── controller/                <-- HTTP Controllers (Adapters)
  └── core/
      ├── domain/                    <-- Domain Entities (Pure models)
      └── usecase/                   <-- Use cases & Business Logic

## 3. Key Implementation Points

- Pola Composition Root memusatkan seluruh inisialisasi infrastruktur (DB, Redis, Sentry, MinIO) pada shared/api/default.go.
- Registrasi controller menggunakan factory closure `func(dbConn *gorm.DB, port base.Port, ctrl base.BaseController) api.Router` sehingga injeksi dependensi dilakukan secara lazy dan seragam.
- Semua modul bisnis dan infrastruktur mengekspor rute melalui implementasi `Route(handler *gin.RouterGroup)`.

## 4. Code Examples

### Composition Root bootstrap in cmd/api registering controllers conditionally with dependency factory

```go
// cmd/api/api.go
package main

import (
	"base-be-golang/internal/adapter/controller"
	"base-be-golang/internal/core/domain"
	"base-be-golang/shared/api"
	"base-be-golang/shared/base"
	"flag"
	"log"
	"os"
	"strconv"

	"github.com/joho/godotenv"
	iamctrl "github.com/rdhmuhammad/base-be-golang/iam-module/shared/adapter/controller"
	"gorm.io/gorm"
)

func main() {
	var envFile string
	flag.StringVar(&envFile, "env", ".env.stag", "Provide env file path")
	flag.Parse()
	if err := godotenv.Load(envFile); err != nil {
		log.Println(err)
		panic(err)
	}

	start := api.Default()

	// IAM Module Toggle Registration
	if t, _ := strconv.ParseBool(os.Getenv("IAM_MODULE_OFF")); !t {
		start.Register(func(dbConn *gorm.DB, port base.Port, ctrl base.BaseController) api.Router {
			return iamctrl.NewAuthController(dbConn, port, ctrl)
		})
		start.Register(func(dbConn *gorm.DB, port base.Port, ctrl base.BaseController) api.Router {
			return iamctrl.NewUserManagementController(dbConn, port, ctrl)
		})
	}

	// Business Modules Registration
	start.Register(func(dbConn *gorm.DB, port base.Port, ctrl base.BaseController) api.Router {
		return controller.NewProductController(dbConn, port, ctrl)
	})

	start.DB().AutoMigrate(&domain.Product{}, &domain.Account{})

	if err := start.Start(); err != nil {
		panic(err)
	}
}
```

### API Engine Lifecycle orchestrating router aggregation under /api/v1 prefix

```go
// shared/api/api.go
package api

import (
	"base-be-golang/pkg/cache"
	"base-be-golang/pkg/logger"
	"base-be-golang/pkg/miniostorage"
	"os"

	"github.com/gin-gonic/gin"
	"gorm.io/gorm"
)

type Api struct {
	server   *gin.Engine
	db       *gorm.DB
	cache    cache.DbClient
	minioStr miniostorage.StorageMinio
	reZero   *logger.ReZero
	routers  []Router
}

type Router interface {
	Route(handler *gin.RouterGroup)
}

func (a *Api) DB() *gorm.DB { return a.db }

func (a *Api) Start() error {
	root := a.server.Group("/api/v1")
	for _, router := range a.routers {
		router.Route(root)
	}

	port := os.Getenv("APP_PORT")
	return a.server.Run("0.0.0.0:" + port)
}
```

## 5. Considerations & Best Practices

- Composition Root di cmd/api/api.go memusatkan instansiasi dependensi sehingga package internal tidak bergantung langsung ke konfigurasi runtime environment.
- Pemisahan modul IAM dan Business modul menggunakan feature flag dinamis (IAM_MODULE_OFF) memudahkan skenario deployment modular monolith ataupun transisi ke microservice.
- Router interface memastikan semua controller hanya mengekspos metode registrasi rute tanpa mengekspos dependensi internal ke engine HTTP.

## 6. Related Knowledge

- Hexagonal Architecture
- Dependency Injection Composition Root

## 7. Source

- Harvest 1787128418632 596cfbe7.json
