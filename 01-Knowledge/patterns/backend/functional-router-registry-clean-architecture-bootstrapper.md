---
supersession: "ACTIVE"
review_by: "2027-02-22"
owner: "knowledge-curator"
confidence: 0.5
provenance_schema: 1
title: "Functional Router Registry & Clean Architecture Bootstrapper"
type: pattern
tags: [pattern, backend, architecture, golang, routing, clean-architecture]
created: 2026-08-26
updated: 2026-08-26
orchestrator_run: "harvest-1787715147310-7bd85203"
sources: ["[[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]"]
---

# Functional Router Registry & Clean Architecture Bootstrapper

Pola registrasi router modular fungsional yang memisahkan kernel server HTTP Gin dari inisialisasi modul bisnis dan adapter.

## 1. Overview & Architecture

Pola ini memisahkan bootstrap micro-framework dengan registrasi modul aplikasi secara deklaratif. Engine Api menyediakan interface Router { Route(*gin.RouterGroup) } dan method Register(func(dbConn, port, ctrl) Router) yang memungkinkan penambahan modul baru di cmd/api/api.go tanpa mengubah logika inti framework.

## 2. Implementation & Code Structure

cmd/
└── api/
    └── api.go                    # App entrypoint & Controller registration
shared/
├── api/
│   ├── api.go                   # Api server engine & Router interface
│   ├── default.go               # Infrastructure initialization (Gin, DB, Cache, Sentry)
│   └── register_api.go          # Functional Register method implementation
└── base/
    └── port.go                  # Port & BaseController dependency contracts
internal/
└── adapter/
    └── controller/
        ├── product.go           # Controller implementing api.Router
        └── contact.go           # Controller implementing api.Router

## 3. Key Implementation Points

- Inisialisasi terpusat infrastruktur (DB, Redis, Sentry, MinIO) di dalam shared/api.Default().
- Penggunaan closure/factory function pada api.Register untuk menginjeksi dbConn, base.Port, dan base.BaseController secara konsisten ke setiap controller.
- Pemisahan router grouping (/api/v1) di level Api struct sehingga endpoint tiap modul tetap bersih dan terisolasi.

## 4. Code Examples

### Inisialisasi aplikasi di entrypoint cmd/api/api.go dengan pendaftaran controller secara deklaratif dan fungsional.

```go
package main

import (
	"base-be-golang/internal/adapter/controller"
	"base-be-golang/internal/core/domain"
	"base-be-golang/pkg/health"
	"base-be-golang/shared/api"
	"base-be-golang/shared/base"
	"flag"
	"log"

	"github.com/joho/godotenv"
	_ "github.com/joho/godotenv/autoload"
	"gorm.io/gorm"
)

func main() {
	var envFile string
	flag.StringVar(&envFile, "env", ".env.stag", "Provide env file path")
	flag.Parse()
	_ = godotenv.Load(envFile)

	start := api.Default()

	// ========================= REGISTER CONTROLLER =========================
	start.Register(func(dbConn *gorm.DB, port base.Port, ctrl base.BaseController) api.Router {
		return health.NewHealthController(port, ctrl)
	})
	start.Register(func(dbConn *gorm.DB, port base.Port, ctrl base.BaseController) api.Router {
		return controller.NewProductController(dbConn, port, ctrl)
	})
	start.Register(func(dbConn *gorm.DB, port base.Port, ctrl base.BaseController) api.Router {
		return controller.NewContactController(dbConn, port, ctrl)
	})

	start.DB().AutoMigrate(&domain.Product{}, &domain.Contact{})

	if err := start.Start(); err != nil {
		panic(err)
	}
}
```

### Engine kernel API yang mengelola agregasi router dan injeksi dependensi shared base.

```go
package api

import (
	"base-be-golang/pkg/cache"
	"base-be-golang/pkg/logger"
	"base-be-golang/pkg/miniostorage"
	"base-be-golang/shared/base"
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

func (a *Api) Register(r func(dbConn *gorm.DB, port base.Port, controller base.BaseController) Router) {
	a.routers = append(a.routers,
		r(a.db,
			base.NewPort(a.db, a.cache, a.reZero),
			base.NewBaseController(a.db, a.cache),
		))
}

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

- Pemisahan interface Router memastikan modul controller tidak perlu mengetahui detail siklus hidup Gin Engine utama.
- Semua middleware global (CORS, Sentry, Body limit) harus dipasang sebelum pemanggilan router.Route() di dalam start.Start().
- Registrasi fungsional via closure mempermudah pengujian mocking pada level integration test.

## 6. Related Knowledge

- Clean Architecture In Go
- Hexagonal Architecture

## 7. Source

- [[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]
