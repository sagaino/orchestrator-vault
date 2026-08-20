---
title: "Declarative Route Registration & Context-Enriching Guard Pipeline"
type: pattern
tags: [pattern, backend, golang, routing, middleware, jwt, rbac, security]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787128418632-596cfbe7
sources: ["Harvest 1787128418632 596cfbe7.json"]
---

# Declarative Route Registration & Context-Enriching Guard Pipeline

Declarative Route Registration & Context-Enriching Guard Pipeline untuk proteksi endpoint API dan propagasi context di Go.

## 1. Overview & Architecture

Pola Declarative Route Registration & Context-Enriching Guard Pipeline mengatur deklarasi rute modular dan mengamankan endpoint API melalui middleware chain terstandarisasi. Middleware memvalidasi token JWT, menginjeksi informasi user ke standard context, mencatat aktivitas user, dan mengevaluasi Role-Based Access Control (RBAC).

## 2. Implementation & Code Structure

iam_module/
  └── pkg/
      ├── middleware/
      │   ├── session.go             <-- Validate() & Authorize() Guard Implementations
      │   └── dto.go                 <-- User context & session payloads
      └── security/
          └── authenticate.go        <-- Token generator & parser
shared/
  └── api/
      └── register_api.go            <-- Router registration pipeline

## 3. Key Implementation Points

- Hierarchical Route Grouping mengisolasi endpoint publik dari endpoint yang membutuhkan otentikasi/otorisasi.
- `Validate()` memverifikasi JWT claim, mengecek masa kedaluwarsa, memperbarui aktivitas pengguna, dan menginjeksi struct `UserData` ke dalam Gin context serta `c.Request.Context()`.
- `Authorize(roles...)` melakukan evaluasi role granular (RBAC) dan menghentikan eksekusi rute (`c.Abort()`) jika role tidak sesuai.

## 4. Code Examples

### JWT Session Validation, Context Enrichment, and RBAC Guard Middleware

```go
// iam_module/pkg/middleware/session.go
package middleware

import (
	"context"
	"net/http"
	"strings"
	"time"

	"base-be-golang/pkg/logger"
	"base-be-golang/shared/payload"
	"github.com/gin-gonic/gin"
	"golang.org/x/exp/slices"
)

const AuthCodeContext = "authData"

func (receiver Auth) Validate() gin.HandlerFunc {
	return func(c *gin.Context) {
		tokenStr := strings.Replace(c.GetHeader("Authorization"), "Bearer ", "", -1)
		secret := []byte(os.Getenv("SECRET"))
		token, err := receiver.parseToken(tokenStr, secret)
		if err != nil {
			c.JSON(http.StatusUnauthorized, payload.DefaultErrorResponseWithMessage(err.Error(), err))
			c.Abort()
			return
		}

		authData, valid := receiver.getAuthData(token)
		userDataStruct := payload.UserData{}
		if err := userDataStruct.LoadFromMap(authData); err != nil || !valid {
			c.JSON(http.StatusUnauthorized, payload.DefaultErrorResponse(err))
			c.Abort()
			return
		}

		// Update activity & enrich standard context
		receiver.setUserActivity(userDataStruct)
		c.Set(string(AuthCodeContext), userDataStruct)
		newCtx := context.WithValue(c.Request.Context(), AuthCodeContext, userDataStruct)
		c.Request = c.Request.WithContext(newCtx)
		c.Next()
	}
}

func (receiver Auth) Authorize(roles ...string) gin.HandlerFunc {
	return func(c *gin.Context) {
		var authData payload.UserData
		if val, ok := c.Get(string(AuthCodeContext)); ok {
			authData = val.(payload.UserData)
		}

		if slices.Contains(roles, authData.RoleName) {
			c.Next()
			return
		}

		c.JSON(http.StatusUnauthorized, payload.DefaultBadRequestResponse())
		c.Abort()
	}
}
```

### Deklarasi rute hierarkis dengan pipeline middleware guard terisolasi

```go
// internal/adapter/controller/product.go
func (ctrl ProductController) Route(router *gin.RouterGroup) {
	products := router.Group("/products")
	
	// Public routes
	products.GET("", ctrl.Get)
	
	// Protected routes with Session Validation & Role Authorization
	authorized := products.Group("")
	authorized.Use(ctrl.Security.Validate())
	{
		authorized.POST("", ctrl.Security.Authorize("ADMIN", "MANAGER"), ctrl.Create)
		authorized.PUT("/:id", ctrl.Security.Authorize("ADMIN"), ctrl.Edit)
		authorized.PATCH("/:id", ctrl.Security.Authorize("ADMIN"), ctrl.EditPartial)
		authorized.DELETE("/:id", ctrl.Security.Authorize("ADMIN"), ctrl.Delete)
	}
}
```

## 5. Considerations & Best Practices

- Context injection memastikan use case tidak bergantung pada HTTP Gin context melainkan pada standard library `context.Context`.
- Pemisahan guard ke dalam dua tahap (`Validate` untuk autentikasi dan `Authorize` untuk otorisasi granular) memberikan fleksibilitas rute publik, authenticated-only, dan role-restricted.
- Automatic last-active update langsung dieksekusi di middleware tanpa mengotori business logic di use case.

## 6. Related Knowledge

- Jwt Authentication Middleware
- Rbac Authorization

## 7. Source

- Harvest 1787128418632 596cfbe7.json
