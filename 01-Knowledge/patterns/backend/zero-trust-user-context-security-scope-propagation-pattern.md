---
supersession: "ACTIVE"
review_by: "2027-02-22"
owner: "knowledge-curator"
confidence: 0.5
provenance_schema: 1
title: "Zero-Trust User Context & Security Scope Propagation Pattern"
type: pattern
tags: [pattern, backend, security, observability, auth]
created: 2026-08-26
updated: 2026-08-26
orchestrator_run: "harvest-1787715147310-7bd85203"
sources: ["[[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]"]
---

# Zero-Trust User Context & Security Scope Propagation Pattern

Zero-Trust User Context & Security Scope Propagation Pattern yang mengisolasi hub tracing, menyaring header sensitif, dan mempropagasi session identity antar layer.

## 1. Overview & Architecture

Pola isolasi konteks keamanan dan propagasi metadata user/session yang terhubung dengan middleware observability. Pola ini memastikan kredensial sensitif tersanitasi secara otomatis saat logging/tracing sambil tetap mempertahankan jejak identitas user (role, ID, timezone) di seluruh lapisan arsitektur.

## 2. Implementation & Code Structure

pkg/
├── middleware/
│   ├── dto.go
│   ├── sentry.go
│   └── session-user.go
shared/
└── payload/
    └── user_context.go

## 3. Key Implementation Points

- Isolation per-request dengan cloning Sentry Hub ke context.Context.
- Sanitasi otomatis untuk token Authorization, Cookie, dan X-Api-Key.
- Two-stage user enrichment (Gin context fallback ke context.Context value).

## 4. Code Examples

### Cloning per-request Sentry Hub, buffering request stream for observability, and redacting sensitive credentials

```go
// middleware/sentry.go & shared/payload/user_context.go
package middleware

import (
	"bytes"
	"context"
	"io"
	"net/http"

	"github.com/getsentry/sentry-go"
	"github.com/gin-gonic/gin"
)

type ctxKey string
const AuthCodeContext = ctxKey("authCode")

func SentryMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {
		hub := sentry.GetHubFromContext(c.Request.Context())
		if hub == nil {
			hub = sentry.CurrentHub().Clone()
		}

		ctx := sentry.SetHubOnContext(c.Request.Context(), hub)
		c.Request = c.Request.WithContext(ctx)

		var requestBody []byte
		if c.Request.Body != nil {
			requestBody, _ = io.ReadAll(c.Request.Body)
			c.Request.Body = io.NopCloser(bytes.NewBuffer(requestBody))
		}

		hub.ConfigureScope(func(scope *sentry.Scope) {
			scope.SetRequest(c.Request)
			scope.SetTag("request.path", c.Request.URL.Path)
			scope.SetTag("request.method", c.Request.Method)
			scope.SetTag("request.remote_addr", c.ClientIP())
			scope.SetExtra("request.headers", convertHeaders(c.Request.Header))
		})

		c.Next()
		enrichSentryWithUserData(hub, c)
	}
}

func convertHeaders(headers http.Header) map[string]string {
	result := make(map[string]string)
	for key, values := range headers {
		if len(values) > 0 {
			if key == "Authorization" || key == "Cookie" || key == "X-Api-Key" {
				result[key] = "[Filtered]"
			} else {
				result[key] = values[0]
			}
		}
	}
	return result
}
```

## 5. Considerations & Best Practices

- Pastikan buffer request body di-reset menggunakan io.NopCloser agar handler downstream dapat membaca body kembali.
- Sensitive header filtering harus case-insensitive atau dinormalisasi sebelum dikirim ke observability tools.
- Sentry Hub harus di-clone per-request agar state concurrency goroutine tidak mengalami race condition.

## 6. Related Knowledge

- Context Propagation
- Sentry Scope Enrichment

## 7. Source

- [[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]
