---
supersession: "ACTIVE"
review_by: "2027-02-22"
owner: "knowledge-curator"
confidence: 0.5
provenance_schema: 1
title: "unified-error-normalization-and-localized-envelope-pipeline"
type: pattern
tags: [pattern, backend, golang, error-handling, i18n, dto, architecture]
created: 2026-08-26
updated: 2026-08-26
orchestrator_run: "harvest-1787715147310-7bd85203"
sources: ["[[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]"]
---

# unified-error-normalization-and-localized-envelope-pipeline

Arsitektur penanganan dan normalisasi error terpadu yang memetakan error database dan domain menjadi respon HTTP standar dengan lokalisasi pesan multi-bahasa dinamis dan pelaporan insiden otomatis.

## 1. Overview & Architecture

Pola Unified Error Normalization and Localized Envelope Pipeline menyediakan arsitektur penanganan error yang kohesif dari level database, domain, hingga level transport HTTP. Pola ini memisahkan secara tegas klasifikasi error internal dan representasi pesan untuk klien melalui enkapsulasi DTO response standar dan engine lokalisasi berbasis template.

## 2. Implementation & Code Structure

pkg/
├── localerror/
│   └── util.go             # Custom error taxonomy (InvalidDataError, AccessControlError, InternalError)
├── mapper/
│   ├── default.go          # Mapper instance and i18n bundle initialization
│   ├── dto.go              # Database error constants (pgconn codes)
│   └── errors.go           # Error mapping, DB translation, localization, HTTP status matching
├── localize/
│   └── localize.go         # i18n bundle management, template placeholder interpolation
shared/
└── payload/
    └── response.go         # Standard envelope definitions (ResponseMeta, ErrorResponse, Response)

## 3. Key Implementation Points

- Taksonomi error domain terisolasi: InvalidDataError untuk HTTP 400, AccessControlError untuk HTTP 401, InternalError untuk HTTP 500.
- Penerjemahan kode error database (PostgreSQL 23505 duplicate, 22001 truncate, 23503 foreign key) menjadi domain error.
- Normalisasi respon API dalam generic envelope (ResponseMeta) yang menjamin konsistensi payload sukses dan gagal.
- Propagasi bahasa klien dari UserData di context untuk resolusi pesan terjemahan otomatis.

## 4. Code Examples

### Domain error taxonomy classification and HTTP envelope mapping with localization and Sentry error capturing

```go
type InvalidDataError struct {
	Msg             string
	DataToTemplated map[string]string
}

func (e InvalidDataError) Error() string {
	return e.Msg
}

func (e InvalidDataError) Is(target error) bool {
	if t, ok := target.(InvalidDataError); ok {
		return e.Msg == t.Msg
	}
	if t, ok := target.(*InvalidDataError); ok && t != nil {
		return e.Msg == t.Msg
	}
	return false
}

// Standard payload envelope definition
type Response struct {
	Code    int    `json:"code"`
	Message string `json:"message"`
	Result  any    `json:"result"`
}

type ErrorResponse struct {
	Code        int    `json:"code"`
	Message     string `json:"message"`
	Result      any    `json:"result"`
	ErrorServer string `json:"errorServer,omitempty"`
	Errors      any    `json:"errors,omitempty"`
}

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
				payload.DefaultErrorInvalidDataWithCode(http.StatusBadRequest, m.localizer.GetLocalized(userData.Lang, err.Error(), templates...)),
			)
			return
		}
		if m.IsAccessControlError(err) {
			c.JSON(
				http.StatusUnauthorized,
				payload.DefaultErrorInvalidDataWithCode(http.StatusUnauthorized, m.localizer.GetLocalized(userData.Lang, err.Error())),
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
		if res.Code == 0 {
			res.Code = http.StatusOK
		}
		res.Message = m.localizer.GetLocalized(userData.Lang, res.Message)
		c.JSON(res.Code, res)
		return
	}
	c.Status(http.StatusOK)
}
```

## 5. Considerations & Best Practices

- Pemisahan jelas antara internal error message (untuk developer/Sentry) dan user-facing localized message (i18n).
- Penggunaan errors.Is dan errors.As pada struct InvalidDataError untuk mendukung custom error matching di Go.
- Placeholder replacement dinamis pada pesan terjemahan menggunakan TemplatingData key-value pairs.

## 6. Related Knowledge

- Cross Cutting Port Facade And Context Propagation Pattern
- Http Response Envelopes

## 7. Source

- [[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]
