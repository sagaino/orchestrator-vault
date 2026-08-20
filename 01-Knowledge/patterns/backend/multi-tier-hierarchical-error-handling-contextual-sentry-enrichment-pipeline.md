---
title: "Multi-Tier Hierarchical Error Handling & Contextual Sentry Enrichment Pipeline"
type: pattern
tags: [pattern, backend, error-handling, sentry, telemetry, dto-envelope]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787128418632-596cfbe7
sources: ["Harvest 1787128418632 596cfbe7.json"]
---

# Multi-Tier Hierarchical Error Handling & Contextual Sentry Enrichment Pipeline

Multi-Tier Hierarchical Error Handling & Contextual Sentry Enrichment Pipeline memisahkan typed domain error dengan runtime panic/database error, serta mengintegrasikan pemformatan HTTP envelope, lokalisasi pesan, dan monitoring Sentry yang aman dari kebocoran credentials.

## 1. Overview & Architecture

Arsitektur manajemen error berlapis yang membedakan domain errors dengan runtime infrastructure errors, terintegrasi langsung dengan mapper response HTTP Gin, dynamic template regex error matching, message localization, serta context-aware Sentry hub injection dengan automated header filtering.

## 2. Implementation & Code Structure

pkg/
├── localerror/
│   └── util.go
├── mapper/
│   └── errors.go
├── middleware/
│   └── sentry.go
└── shared/payload/
    └── response.go

## 3. Key Implementation Points

- Pemisahan tegas domain error (InvalidDataError, AccessControlError) dan infrastructure error (InternalError).
- Dynamic regex error template matching (ErrorIs, ReplaceLabelErr) untuk memvalidasi error berparameter.
- Response envelope polimorfik terstandardisasi (ResponseMeta, ErrorResponse, Response).
- Sentry context enrichment dengan user ID, role, dan locale setelah pipeline auth.

## 4. Code Examples

### Definisi typed domain errors dan error handler wrapper.

```go
package localerror

import (
	"base-be-golang/pkg/logger"
	"errors"
	"gorm.io/gorm"
)

type InvalidDataError struct {
	Msg             string
	DataToTemplated map[string]string
}

func (e InvalidDataError) Error() string {
	return e.Msg
}

type AccessControlError struct {
	Msg string
}

func (e AccessControlError) Error() string {
	return e.Msg
}

func NotFound(err error, msg string) error {
	if err != nil && errors.Is(err, gorm.ErrRecordNotFound) {
		return InvalidDataError{Msg: msg}
	}
	return err
}

func InvalidDataWithData(msg string, data map[string]string) error {
	return InvalidDataError{Msg: msg, DataToTemplated: data}
}

type HandleError struct {
	logger *logger.ReZero
}

func (h HandleError) ErrorReturn(err error) error {
	if errors.As(err, &AccessControlError{}) || errors.As(err, &InvalidDataError{}) {
		return err
	}
	h.logger.Error(err)
	return errors.New("InternalError")
}
```

### Unified HTTP error mapper dengan integrasi localization dan Sentry capture.

```go
func (m Mapper) NewResponse(c *gin.Context, res *payload.Response, err error) {
	userData := m.GetAuthDataFromContext(c)
	if err != nil {
		if ok, invErr := m.IsInvalidDataError(err); ok {
			var templates = make([]localize.TemplatingData, 0)
			if invErr.DataToTemplated != nil {
				for key, val := range invErr.DataToTemplated {
					templates = append(templates, localize.TemplatingData{
						Name:  key,
						Value: val,
					})
				}
			}
			c.JSON(
				http.StatusBadRequest,
				payload.DefaultErrorInvalidDataWithMessage(m.localizer.GetLocalized(userData.Lang, err.Error(), templates...)),
			)
			return
		}
		if m.IsAccessControlError(err) {
			c.JSON(
				http.StatusUnauthorized,
				payload.DefaultErrorInvalidDataWithMessage(m.localizer.GetLocalized(userData.Lang, err.Error())),
			)
			return
		}
		middleware.CaptureError(c, err)
		c.JSON(
			http.StatusInternalServerError,
			payload.DefaultErrorResponseWithMessage(m.localizer.GetLocalized(userData.Lang, "InternalError"), err),
		)
		return
	}
	if res != nil {
		res.Message = m.localizer.GetLocalized(userData.Lang, res.Message)
		c.JSON(http.StatusOK, res)
		return
	}
	c.Status(http.StatusOK)
}
```

### Sentry middleware dengan request body preservation dan header sanitization.

```go
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
			scope.SetExtra("request.headers", convertHeaders(c.Request.Header))
			scope.SetExtra("request.body", string(requestBody))
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

- Typed domain errors mencegah kebocoran detail teknis database (GORM error) ke response publik.
- Sentry header filter wajib memfilter Authorization dan Cookie token agar tidak terekspos di dashboard log eksternal.
- Penyimpanan dan pembacaan ulang c.Request.Body menggunakan io.NopCloser penting agar downstream handlers tetap dapat membaca payload HTTP.

## 6. Related Knowledge

- Error Handling Architecture
- Sentry Telemetry Pipeline

## 7. Source

- Harvest 1787128418632 596cfbe7.json
