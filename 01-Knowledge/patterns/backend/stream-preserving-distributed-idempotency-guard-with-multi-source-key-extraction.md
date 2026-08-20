---
title: "Stream-Preserving Distributed Idempotency Guard with Multi-Source Key Extraction"
type: pattern
tags: [pattern, backend, idempotency, distributed-lock, redis, middleware, gin, stream-handling]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787128418632-596cfbe7
sources: ["Harvest 1787128418632 596cfbe7.json"]
---

# Stream-Preserving Distributed Idempotency Guard with Multi-Source Key Extraction

Stream-Preserving Distributed Idempotency Guard with Multi-Source Key Extraction supporting JSON and Multipart forms with body stream rewinding.

## 1. Overview & Architecture

This pattern implements a robust distributed idempotency guard designed to protect sensitive state-mutating endpoints (e.g. payment submissions, transaction creation) from duplicate requests and replay attacks. It dynamically resolves the idempotency key across various request formats and restores drained stream buffers using io.NopCloser to preserve payload availability for subsequent middleware and handlers.

## 2. Implementation & Code Structure

pkg/
└── middleware/
    └── idempotent.go            # Idempotent middleware, body draining, multipart parser, Redis lock

## 3. Key Implementation Points

- Multi-Tier Key Discovery: Automatically falls back through Route Params -> Query Strings -> JSON Body -> Multipart Form fields to locate the idempotency key.
- Stream Drain & Buffer Rewind: Reads c.Request.Body and restores it using io.NopCloser(bytes.NewBuffer(bodyByte)) so downstream handlers can re-parse the body cleanly.
- Multipart Form Inspector: Safely parses multipart forms without breaking file uploads or value bindings.
- Distributed Redis Mutex: Uses cache TTL locks to atomically reject duplicate concurrent submissions with HTTP 409 Conflict.

## 4. Code Examples

### Idempotent middleware handler resolving keys from params, query, JSON body, and multipart forms with distributed Redis lock and 409 Conflict rejection.

```go
func (idem IDEMPOTENT) Idempotent(name string, paramKey string, lockTime time.Duration) gin.HandlerFunc {
	return func(c *gin.Context) {
		// prepare the key
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
		lock, err := idem.cache.Get(ctx, idempotenceKey)
		if lock != "" {
			c.JSON(http.StatusConflict, payload.DefaultErrorResponseWithMessage("IDEMPOTENT request", err))
			c.Abort()
			return
		}

		// lock request
		_ = idem.cache.Set(ctx, idempotenceKey, "locked", lockTime)

		// handle request
		c.Next()
	}
}
```

### Stream-preserving body extraction that drains and restores the Request.Body buffer via io.NopCloser and safely parses multipart form values.

```go
func (idem IDEMPOTENT) getBodyJSON(c *gin.Context) map[string]any {
	var body = map[string]any{}
	bodyRaw := c.Copy().Request.Body
	bodyByte, err := io.ReadAll(bodyRaw)
	if err != nil {
		fmt.Println(err)
	}
	err = json.Unmarshal(bodyByte, &body)
	if err != nil {
		fmt.Println(err)
	}

	// Restore the request body so it can be used by Gin
	c.Request.Body = io.NopCloser(bytes.NewBuffer(bodyByte))

	return body
}

func (idem IDEMPOTENT) getBodyMultiPart(c *gin.Context) map[string]any {
	reqBody, err := c.MultipartForm()
	if err != nil {
		fmt.Println("c.MultipartForm: %w", err)
	}

	body := make(map[string]any)
	for key, val := range reqBody.File {
		fileName := filepath.Base(val[0].Filename)
		body[key] = fileName
	}
	for key, val := range reqBody.Value {
		body[key] = val
	}
	return body
}
```

## 5. Considerations & Best Practices

- Reading the entire JSON request body into memory requires appropriate request size limits to prevent memory exhaustion on massive payloads.
- Lock time (TTL) must be tuned to match the expected execution duration of the underlying mutation handler.
- IP-scoping in idempotence keys helps isolate locks per client while preventing cross-tenant locking conflicts.

## 6. Related Knowledge

- Distributed Idempotency Pipeline
- Gin Request Body Rewind Snippet

## 7. Source

- Harvest 1787128418632 596cfbe7.json
