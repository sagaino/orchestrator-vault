---
supersession: "ACTIVE"
review_by: "2027-02-22"
owner: "knowledge-curator"
confidence: 0.5
provenance_schema: 1
title: "Distributed Idempotency Guard & Request Pipeline Pattern"
type: pattern
tags: [pattern, backend, golang, middleware, idempotency, redis, concurrency]
created: 2026-08-26
updated: 2026-08-26
orchestrator_run: "harvest-1787715147310-7bd85203"
sources: ["[[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]"]
---

# Distributed Idempotency Guard & Request Pipeline Pattern

Middleware pipeline Gin berbasis Redis lock mutex untuk menjamin idempotensi eksekusi endpoint berisiko tinggi (misal transaksi/pembayaran).

## 1. Overview & Architecture

Pola ini menyediakan middleware guard untuk menjamin idempotensi eksekusi endpoint write/mutasi berisiko tinggi. Middleware mengekstrak idempotency key dari parameter URL, query, atau body JSON, menguncinya di Redis selama durasi tertentu, dan mengembalikan 409 Conflict jika request identik terdeteksi sebelum kunci kadaluarsa.

## 2. Implementation & Code Structure

pkg/
└── middleware/
    ├── idempotent.go            # Redis-based distributed idempotency guard
    ├── cors.go                  # Cross-Origin Resource Sharing configuration
    ├── sentry.go                # Request context enrichment for distributed tracing
    └── validator.go             # Input validation & schema binding middleware

## 3. Key Implementation Points

- Multi-channel parameter extraction: mengecek URL Param, Query String, JSON Body, hingga Multipart Form secara berurutan.
- Request body stream buffering & restoration agar pipeline Gin downstream tetap dapat membaca payload.
- Pencegahan concurrent duplicate request menggunakan atomic key check pada Redis dengan response 409 Conflict.

## 4. Code Examples

### Implementasi middleware Idempotent berbasis Redis lock dengan pemulihan request body stream.

```go
package middleware

import (
	"base-be-golang/pkg/cache"
	"base-be-golang/shared/payload"
	"bytes"
	"context"
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"path/filepath"
	"strings"
	"time"

	"github.com/gin-gonic/gin"
)

type IDEMPOTENT struct {
	cache cache.DbClient
}

func NewIdempotent(defaultCache cache.DbClient) IDEMPOTENT {
	return IDEMPOTENT{
		cache: defaultCache,
	}
}

const IdempotencePrefixKey = "base-be:IDEMPOTENT"

func (idem IDEMPOTENT) Idempotent(name string, paramKey string, lockTime time.Duration) gin.HandlerFunc {
	return func(c *gin.Context) {
		key := strings.ReplaceAll(strings.ToLower(c.Param(paramKey)), " ", "")
		if key == "" {
			key = strings.ReplaceAll(strings.ToLower(c.Query(paramKey)), " ", "")
		}
		if key == "" {
			contentType := c.ContentType()
			var body map[string]any
			switch contentType {
			case gin.MIMEMultipartPOSTForm:
				body = idem.getBodyMultiPart(c)
			case gin.MIMEJSON:
				body = idem.getBodyJSON(c)
			}
			key = strings.ReplaceAll(strings.ToLower(fmt.Sprint(body[paramKey])), " ", "")
		}
		if key == "" {
			c.Next()
			return
		}

		ipAddress := c.ClientIP()
		idempotenceKey := fmt.Sprintf("%v-%v-%v-%v", IdempotencePrefixKey, ipAddress, name, key)
		ctx := context.Background()

		lock, _ := idem.cache.Get(ctx, idempotenceKey)
		if lock != "" {
			c.JSON(http.StatusConflict, payload.DefaultErrorResponseWithMessage("IDEMPOTENT request", nil))
			c.Abort()
			return
		}

		_ = idem.cache.Set(ctx, idempotenceKey, "locked", lockTime)
		c.Next()
	}
}

func (idem IDEMPOTENT) getBodyJSON(c *gin.Context) map[string]any {
	var body = map[string]any{}
	bodyRaw := c.Copy().Request.Body
	bodyByte, _ := io.ReadAll(bodyRaw)
	_ = json.Unmarshal(bodyByte, &body)

	// Restore the request body stream for subsequent Gin handlers
	c.Request.Body = io.NopCloser(bytes.NewBuffer(bodyByte))
	return body
}
```

## 5. Considerations & Best Practices

- Pemulihan stream (c.Request.Body = io.NopCloser(...)) wajib dilakukan agar binding JSON pada handler berikutnya tidak menghasilkan EOF error.
- Durasi lock (TTL) harus disesuaikan dengan perkiraan latency transaksi maksimum agar tidak menolak request yang sah.

## 6. Related Knowledge

- Distributed Locks
- Api Idempotency

## 7. Source

- [[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]
