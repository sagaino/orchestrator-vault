---
title: "Fluent Typed HTTP Pipeline Builder with Reflection-Driven Form/Multipart Serialization"
type: pattern
tags: [pattern, backend, http-client, fluent-builder, reflection, rest-repo, golang]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787128418632-596cfbe7
sources: ["Harvest 1787128418632 596cfbe7.json"]
---

# Fluent Typed HTTP Pipeline Builder with Reflection-Driven Form/Multipart Serialization

Fluent Typed HTTP Pipeline Builder with Reflection-Driven Form/Multipart Serialization providing expressive API integration tooling.

## 1. Overview & Architecture

This pattern provides an expressive, type-safe HTTP client builder architecture for external service integrations. It encapsulates standard HTTP client boilerplate, context deadline propagation, header injection, reflection-based struct serialization for JSON/Form/Multipart payloads, and typed response unmarshaling.

## 2. Implementation & Code Structure

pkg/
└── inetproto/
    ├── builder.go               # RestRepoBuilder & Statement fluent builder
    ├── http.go                  # HttpServer, HttpInterface, and reflection serializers
    ├── reflects.go              # Type definitions and reflection slices
    └── repo.go                  # RestRepoTemplate utility for REST integrations

## 3. Key Implementation Points

- Fluent Interface API: Allows expressive chaining of WithContext, Post, Header, BodyJSON, and BodyResponse before firing Send().
- Reflection Serialization: Dynamically inspects struct tags (json) and converts string, int, uint, and float types to url.Values encoded streams.
- Context Cancellation Propagation: Propagates parent context cancellation down to http.NewRequestWithContext for deadline enforcement.
- Encapsulated Response Binding: Handles reading res.Body and unmarshaling directly into target structs with status code inspection.

## 4. Code Examples

### Fluent Statement builder managing context, headers, URL query concatenation, HTTP execution, and response unmarshaling.

```go
type RestRepoBuilder struct {
	Builder Statement
}

type Statement struct {
	client       HttpServer
	ctx          context.Context
	baseUrl      string
	body         interface{}
	bodyResponse interface{}
	method       string
	baseHeader   []RequestHeader
}

func (r *Statement) WithContext(ctx context.Context) *Statement {
	r.ctx = ctx
	return r
}

func (r *Statement) Post(path string, args ...QueryKeyVal) *Statement {
	var url = r.baseUrl + path
	query := "?"
	for _, arg := range args {
		query += fmt.Sprintf("&%s=%s", arg.Key, arg.Val)
	}
	if len(args) > 0 {
		url += query
	}
	r.baseUrl = url
	r.method = http.MethodPost
	return r
}

func (r *Statement) BodyJSON(body interface{}) *Statement {
	r.baseHeader = append(r.baseHeader, RequestHeader{
		Key:   "Content-Type",
		Value: gin.MIMEJSON,
	})
	r.body = body
	return r
}

func (r *Statement) Send() error {
	if reflect.TypeOf(r.bodyResponse).Kind() != reflect.Pointer {
		return fmt.Errorf("body Response struct should be Pointer of struct")
	}

	if r.ctx == nil {
		r.ctx = context.Background()
	}

	request, err := r.client.CreateRequest(
		r.ctx,
		r.baseHeader,
		r.method,
		r.baseUrl,
		r.body,
	)
	if err != nil {
		return err
	}

	res, err := r.client.Do(request)
	if err != nil {
		return err
	}

	err = r.client.BindResponse(res, r.bodyResponse)
	if err != nil {
		return err
	}

	return nil
}
```

### Reflection-based struct serializer converting Go structs into url.Values form data across multiple primitive data types.

```go
func (h HttpServer) convertToUrlFormData(data interface{}) (io.Reader, error) {
	forms := url.Values{}
	dType := reflect.TypeOf(data)
	dValue := reflect.ValueOf(data)

	if data == nil {
		return nil, nil
	}

	if dType.Kind() == reflect.Struct {
		for i := 0; i < dType.NumField(); i++ {
			key := dType.Field(i).Tag.Get("json")
			stval := dValue.Field(i)

			if str, ok := stval.Interface().(string); ok {
				forms.Set(key, str)
				continue
			}

			kind := dValue.Field(i).Kind()
			switch {
			case slices.Contains(TypeIsInt, kind):
				forms.Set(key, strconv.FormatInt(stval.Int(), 10))
				continue
			case slices.Contains(TypeIsUint, kind):
				forms.Set(key, strconv.FormatUint(stval.Uint(), 10))
				continue
			case slices.Contains(TypeIsFloat, kind):
				forms.Set(key, strconv.FormatFloat(stval.Float(), 'f', -1, 32))
				continue
			default:
				return nil, fmt.Errorf("invalid data type")
			}
		}

		p := strings.NewReader(forms.Encode())
		return p, nil
	}

	return nil, fmt.Errorf("only accept struct type")
}
```

## 5. Considerations & Best Practices

- Response targets must always be passed as pointers to allow json.Unmarshal mutation.
- Reusing builder instances across concurrent goroutines requires resetting internal state or creating independent statement instances.
- Reflection overhead is negligible for network I/O operations but should be kept organized within dedicated conversion routines.

## 6. Related Knowledge

- Fluent Rest Client Pipeline
- Struct To Url Form Reflection

## 7. Source

- Harvest 1787128418632 596cfbe7.json
