---
supersession: "ACTIVE"
review_by: "2027-02-22"
owner: "knowledge-curator"
confidence: 0.5
provenance_schema: 1
title: "Fluent Typed HTTP Request Pipeline with Dynamic MIME Content Marshalling"
type: pattern
tags: [pattern, backend, http-client, resilience, fluent-builder]
created: 2026-08-26
updated: 2026-08-26
orchestrator_run: "harvest-1787715147310-7bd85203"
sources: ["[[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]"]
---

# Fluent Typed HTTP Request Pipeline with Dynamic MIME Content Marshalling

Fluent Typed HTTP Request Pipeline with Dynamic MIME Content Marshalling untuk komunikasi antar-service yang konsisten dan type-safe.

## 1. Overview & Architecture

Pola HTTP client pipeline dengan antarmuka fluent builder yang secara dinamis memetakan struct Go ke berbagai format payload (JSON, Form-Data, Multipart) dengan integrasi context propagation.

## 2. Implementation & Code Structure

pkg/
└── inetproto/
    ├── builder.go
    ├── http.go
    ├── reflects.go
    └── repo.go

## 3. Key Implementation Points

- Fluent Method Chaining untuk menyusun request HTTP secara deklaratif.
- Dynamic MIME Content Negotiation (JSON, Form URL-encoded, Multipart).
- Decoupled HTTP client instance yang memfasilitasi mocking dan testing.

## 4. Code Examples

### Fluent request builder with automatic struct-to-MIME payload serialization and context enforcement

```go
// pkg/inetproto/builder.go & http.go
package inetproto

import (
	"bytes"
	"context"
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"net/url"
	"reflect"
	"strings"
	"github.com/gin-gonic/gin"
)

type Statement struct {
	client       HttpServer
	ctx          context.Context
	baseUrl      string
	body         interface{}
	bodyResponse interface{}
	method       string
	baseHeader   []RequestHeader
}

func (r *Statement) BodyJSON(body interface{}) *Statement {
	r.baseHeader = append(r.baseHeader, RequestHeader{Key: "Content-Type", Value: gin.MIMEJSON})
	r.body = body
	return r
}

func (h HttpServer) CreateRequest(ctx context.Context, header []RequestHeader, method, urlStr string, body any) (*http.Request, error) {
	var bodyReq io.Reader
	var contentType string
	for _, hd := range header {
		if hd.Key == "Content-Type" {
			contentType = hd.Value
		}
	}

	if body != nil {
		switch contentType {
		case gin.MIMEPOSTForm, gin.MIMEMultipartPOSTForm:
			var err error
			bodyReq, err = h.convertToUrlFormData(body)
			if err != nil {
				return nil, err
			}
		case gin.MIMEJSON:
			payloadBytes, _ := json.Marshal(body)
			bodyReq = bytes.NewBuffer(payloadBytes)
		default:
			if rc, ok := body.(io.ReadCloser); ok {
				bodyReq = rc
			}
		}
	}

	req, err := http.NewRequestWithContext(ctx, method, urlStr, bodyReq)
	if err != nil {
		return nil, err
	}
	for _, h := range header {
		req.Header.Set(h.Key, h.Value)
	}
	return req, nil
}
```

## 5. Considerations & Best Practices

- Pastikan pointer passed ke BodyResponse adalah valid struct pointer untuk menghindari reflect panic saat decoding.
- Gunakan context.WithTimeout untuk mencegah koneksi hanging pada external integration.

## 6. Related Knowledge

- Resilient Http Client
- Fluent Request Builder

## 7. Source

- [[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]
