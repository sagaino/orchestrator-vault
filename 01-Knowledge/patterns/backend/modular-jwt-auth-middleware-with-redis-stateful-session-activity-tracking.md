---
supersession: ACTIVE
review_by: 2027-02-23
owner: "local:sagaino"
confidence: 0.95
provenance_schema: 1
title: "Modular JWT Auth Middleware with Redis Stateful Session & Activity Tracking"
type: pattern
tags: [pattern, backend, golang, auth, jwt, redis, session, middleware, rbac]
created: 2026-08-19
updated: 2026-08-27
orchestrator_run: task-014-20260827T080907Z-618ab2c0
sources: ["Harvest 1787108504660 B577a5b1.json", "[[03-Sources/other/orchestrator-runs/task-013-20260827T080549Z-96fe5437.json]]"]
---

# Modular JWT Auth Middleware with Redis Stateful Session & Activity Tracking

Modular JWT Auth Middleware with Redis Stateful Session & Activity Tracking in Go Clean Architecture.

## 1. Overview & Architecture

Pluggable IAM authentication and authorization middleware combining stateless JWT, stateful Redis session validation, and RBAC.

## 2. Implementation & Code Structure

iam_module/pkg/middleware/session.go and iam_module/pkg/security/authenticate.go provide authentication and authorization.

## 3. Key Implementation Points

- Stateless JWT token validation with algorithm checks
- Stateful Redis session validation for revocable tokens
- RBAC role-based authorization slice checks

## 4. Code Examples

### Modular JWT auth middleware with RBAC and Redis session tracking

```go
func (receiver Auth) Authorize(roles ...string) gin.HandlerFunc {
	return func(c *gin.Context) {
		var authData = payload.UserData{}
		if authDataStr, ok := c.Get("authData"); ok {
			authData = authDataStr.(payload.UserData)
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

## 5. Considerations & Best Practices

- HMAC signing method verification prevents algorithm confusion attacks
- User activity updates should be handled efficiently to avoid slowing down requests

## 6. Related Knowledge

- Golang Structured Domain Error I18n

## 7. Source

- Harvest 1787108504660 B577a5b1.json

## Update from TASK-013 — 2026-08-27

<!-- orchestrator-run:task-013-20260827T080549Z-96fe5437 -->
Pembaruan implementasi Modular JWT Auth Middleware dengan validasi session stateful di Redis (session:user:<userId>), propagasi context Gin (currentUser, userId), dan flow logout controller untuk revokasi token instan.

- Rationale: Task TASK-013 mengimplementasikan validasi session Redis pada auth middleware Gin serta endpoint logout controller. Pengetahuan ini secara langsung memperkaya dan mengkonkretkan implementasi pada halaman pattern existing 'Modular JWT Auth Middleware with Redis Stateful Session & Activity Tracking'.
- Source: [[03-Sources/other/orchestrator-runs/task-013-20260827T080549Z-96fe5437.json]]

## Update from TASK-014 — 2026-08-27

<!-- orchestrator-run:task-014-20260827T080907Z-618ab2c0 -->
Pembaruan unit testing suite dan unmarshaling contract untuk validasi session Redis pada auth middleware: menangani custom unmarshaler validation pada session snapshot DTO untuk mencegah silent zero-value struct deserialization serta memastikan fallback map/raw context parsing.

- Rationale: Task TASK-014 menambahkan unit testing menyeluruh untuk verifikasi login session creation, middleware session hit/miss (401), logout session deletion, serta perbaikan custom UnmarshalJSON pada UserSessionSnapshot DTO agar json.Unmarshal tidak mengabaikan non-matching payload. Insight ini secara langsung memperkaya knowledge page existing 'Modular JWT Auth Middleware with Redis Stateful Session & Activity Tracking'.
- Source: [[03-Sources/other/orchestrator-runs/task-014-20260827T080907Z-618ab2c0.json]]
