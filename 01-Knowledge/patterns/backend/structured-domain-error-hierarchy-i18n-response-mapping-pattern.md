---
title: "Structured Domain Error Hierarchy & I18n Response Mapping Pattern"
type: pattern
tags: [pattern, backend, golang, error-handling, i18n, gin, middleware]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787108504660-b577a5b1
sources: ["Harvest 1787108504660 B577a5b1.json"]
---

# Structured Domain Error Hierarchy & I18n Response Mapping Pattern

Structured Domain Error Hierarchy & I18n Response Mapping Pattern separating domain errors from HTTP transport with localization.

## 1. Overview & Architecture

Hierarchical domain error architecture separating business errors from transport layer responses with multi-language i18n support.

## 2. Implementation & Code Structure

pkg/localerror/util.go defines domain errors and pkg/mapper/errors.go maps errors to localized JSON responses.

## 3. Key Implementation Points

- Pure domain error types (InvalidDataError, AccessControlError)
- I18n localized message lookup using user context language
- Decoupled HTTP status and payload formatting via Mapper

## 4. Code Examples

### Domain error hierarchy with Gin HTTP translation and localization mapping

```go
type InvalidDataError struct {
	Msg             string
	DataToTemplated map[string]string
}
func (e InvalidDataError) Error() string { return e.Msg }

func (m Mapper) NewResponse(c *gin.Context, res *payload.Response, err error) {
	userData := m.GetAuthDataFromContext(c)
	if err != nil {
		if ok, invErr := m.IsInvalidDataError(err); ok {
			c.JSON(http.StatusBadRequest, payload.DefaultErrorInvalidDataWithMessage(m.localizer.GetLocalized(userData.Lang, err.Error())))
			return
		}
		c.JSON(http.StatusInternalServerError, payload.DefaultErrorResponseWithMessage(m.localizer.GetLocalized(userData.Lang, "InternalError"), err))
		return
	}
	c.JSON(http.StatusOK, res)
}
```

## 5. Considerations & Best Practices

- Domain errors should never expose database driver errors directly to clients
- Internal server errors must be logged and replaced with a generic localized message

## 6. Related Knowledge

- Golang Modular Jwt Redis Session

## 7. Source

- Harvest 1787108504660 B577a5b1.json
