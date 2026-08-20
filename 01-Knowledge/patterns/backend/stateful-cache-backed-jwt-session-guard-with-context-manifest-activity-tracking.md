---
title: "Stateful Cache-Backed JWT Session Guard with Context Manifest & Activity Tracking"
type: pattern
tags: [pattern, backend, authentication, jwt, session-management, redis, security, golang]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787128418632-596cfbe7
sources: ["Harvest 1787128418632 596cfbe7.json"]
---

# Stateful Cache-Backed JWT Session Guard with Context Manifest & Activity Tracking

Stateful Cache-Backed JWT Session Guard with Context Manifest & Activity Tracking combining stateless JWT claims with Redis session storage and context enrichment.

## 1. Overview & Architecture

This pattern combines stateless JWT verification with stateful Redis session caching to achieve the scalability of JWTs along with immediate revocation and single-login enforcement capabilities. Upon successful signature and expiration verification, the token claims are converted into a rich UserManifest struct, enriched with the user's localized timezone, injected into both Gin Context and Go standard context.Context, and logged for user activity tracking.

## 2. Implementation & Code Structure

iam_module/
├── pkg/
│   ├── security/
│   │   ├── authenticate.go      # JWT generation & signing
│   │   └── dto.go               # SingleTokenClaim, UserData DTOs
│   └── middleware/
│       ├── session.go           # Validate(), Authorize(), SetSession(), GetSession()
│       └── dto.go               # Context keys & payload mappings
└── internal/core/domain/
    └── user_interface.go        # UserEntityInterface for last_active touch

## 3. Key Implementation Points

- Dual Context Enrichment: Sets user manifest both into gin.Context and standard Go context.Context for seamless downstream use case propagation.
- Timezone Dynamic Location: Parses user timezone on-the-fly with fallback to UTC, attaching the loaded time.Location to the request context.
- Hybrid Revocation via Cache: Combines cryptographic JWT signature checks with optional Redis-backed session lookup for immediate token invalidation.
- Role-Based Authorization Interceptor: Authorize(roles...) middleware verifies claims against permitted roles before executing handlers.

## 4. Code Examples

### JWT Validation middleware extracting token claims, resolving timezone location, enriching Gin & Request Context, and triggering activity logging.

```go
func (receiver Auth) Validate() gin.HandlerFunc {
	return func(c *gin.Context) {
		tokenStr := strings.Replace(c.GetHeader("Authorization"), "Bearer ", "", -1)
		secret := os.Getenv("SECRET")
		token, err := receiver.parseToken(tokenStr, []byte(secret))
		if err != nil {
			response := payload.DefaultErrorResponseWithMessage(err.Error(), err)
			c.JSON(http.StatusUnauthorized, response)
			c.Abort()
			return
		}

		authData, valid := receiver.getAuthData(token)
		userDataStruct := payload.UserData{}
		err = userDataStruct.LoadFromMap(authData)
		if err != nil {
			response := payload.DefaultErrorResponse(err)
			response.Message = receiver.localize.GetLocalized(userDataStruct.Lang, constant2.SessionExpired.String())
			c.JSON(http.StatusUnauthorized, response)
			c.Abort()
			return
		}

		if valid {
			receiver.setUserActivity(userDataStruct)
			tz := time.UTC
			if userDataStruct.Timezone != "" {
				tz, err = time.LoadLocation(userDataStruct.Timezone)
				if err != nil {
					logger.Error(err)
				}
			}
			userDataStruct.Tz = tz
			c.Set(string(AuthCodeContext), userDataStruct)
			olCtx := c.Request.Context()
			newCtx := context.WithValue(olCtx, AuthCodeContext, userDataStruct)
			c.Request = c.Request.WithContext(newCtx)
			c.Next()
			return
		}

		response := payload.DefaultErrorResponse(err)
		c.JSON(http.StatusUnauthorized, response)
		c.Abort()
	}
}
```

### Redis-backed session synchronization enabling immediate session invalidation and single-user policies.

```go
func (receiver Auth) SetSession(ctx context.Context, user payload.SessionDataUser) error {
	loginCacheKey := "LOGIN_KEY_"

	marshal, err := json.Marshal(user)
	if err != nil {
		return err
	}

	return receiver.cache.Set(
		ctx,
		loginCacheKey+user.UserReference,
		string(marshal),
		time.Hour*time.Duration(receiver.env.GetInt("EXPIRED_TOKEN_JWT", 1)),
	)
}

func (receiver Auth) GetSession(ctx context.Context, authCode string, sessionData *payload.SessionDataUser) error {
	loginCacheKey := "LOGIN_KEY_"
	sessionStr, err := receiver.cache.Get(ctx, loginCacheKey+authCode)
	if err != nil {
		if errors.Is(redis.Nil, err) {
			return localerror.AccessControlError{Msg: constant2.AccessNotAllowed.String()}
		}
		return err
	}

	err = json.Unmarshal([]byte(sessionStr), sessionData)
	if err != nil {
		return err
	}

	return nil
}
```

## 5. Considerations & Best Practices

- JWT secret must be securely injected via environment variables and rotated periodically.
- Timezone parsing fallback to time.UTC prevents unexpected crashes when clients submit malformed IANA timezone strings.
- Activity timestamp updates should be executed via lightweight partial column updates to minimize database load.

## 6. Related Knowledge

- Hybrid Token Session Architecture
- Gin Context User Manifest

## 7. Source

- Harvest 1787128418632 596cfbe7.json
