---
supersession: "ACTIVE"
review_by: "2027-02-22"
owner: "knowledge-curator"
confidence: 0.5
provenance_schema: 1
title: "declarative-struct-tag-reflection-query-binder-and-validation-matrix"
type: pattern
tags: [pattern, backend, golang, validation, reflection, middleware, data-binding]
created: 2026-08-26
updated: 2026-08-26
orchestrator_run: "harvest-1787715147310-7bd85203"
sources: ["[[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]"]
---

# declarative-struct-tag-reflection-query-binder-and-validation-matrix

Sistem binding dan validasi permintaan HTTP berbasis refleksi deklaratif yang mengekstrak nilai query, path, dan multipart ke DTO bertipe aman dengan matriks terjemahan pesan kesalahan terpadu.

## 1. Overview & Architecture

Pola Declarative Struct-Tag Reflection Query Binder and Validation Matrix menyediakan mekanisme otomatis untuk memetakan parameter HTTP (query params, path variable, multipart body) ke dalam struct DTO menggunakan metadata struct tag, sekaligus memvalidasi format data dan mengembalikan translasi pesan kesalahan yang terstandarisasi.

## 2. Implementation & Code Structure

pkg/
└── middleware/
    ├── validator.go                      # Enigma struct validator, reflection query binder, type converters
    ├── custom-translation-validator.go   # Custom validation rules (monthyearformat, trx-status) and translations
    ├── custome-validation.go             # Enum and pattern validation functions
    ├── shared-mapper.go                  # Non-destructive body extraction (JSON & Multipart restoration)
    └── dto.go                            # Context key definitions and validator test structures

## 3. Key Implementation Points

- Engine Enigma mengintegrasikan go-playground/validator/v10 dengan universal-translator untuk pesan validasi ramah pengguna.
- Custom struct tag bindQuery mendukung konfigurasi deklaratif sumber data (pathVariable / queryParams) dan konversi tipe data otomatis.
- Binding rules modular untuk timestamp (dengan timezone awareness), integer, bigint, float, boolean, regex string, dan UUID.
- Pemulihan body stream request (JSON & Multipart) setelah pembacaan di lapisan middleware.

## 4. Code Examples

### Declarative struct-tag reflection query binding with validation translation matrix

```go
type Enigma struct {
	engine *validator.Validate
	trans  ut.Translator
}

func (v Enigma) BindAndValidate(c *gin.Context, payload any) map[string][]string {
	var err error
	if c.ContentType() == gin.MIMEMultipartPOSTForm {
		err = c.ShouldBind(payload)
	} else {
		err = c.Bind(payload)
	}
	if err != nil {
		var errJSON *json.UnmarshalTypeError
		if errors.As(err, &errJSON) {
			return map[string][]string{
				errJSON.Field: {errJSON.Error()},
			}
		}
		return map[string][]string{"error": {err.Error()}}
	}
	return v.Validate(c, payload)
}

func (v Enigma) queryToFilter(c *gin.Context, payload interface{}, isDive bool) error {
	pVal, pType, vals, err := v.preparingReflection(payload, isDive)
	if err != nil {
		return err
	}

	for i := 0; i < vals.NumField(); i++ {
		tagKeyVal := strings.Split(pType.Elem().Field(i).Tag.Get("bindQuery"), ";")
		var tags = map[string]string{}
		for j := 0; j < len(tagKeyVal); j++ {
			if tagKeyVal[j] == "" {
				continue
			}
			keyVals := strings.Split(tagKeyVal[j], "=")
			if len(keyVals) >= 2 {
				tags[keyVals[0]] = keyVals[1]
			}
		}

		if _, ok := tags["ignore"]; ok {
			continue
		}

		reqField := pType.Elem().Field(i).Tag.Get("json")
		if rn, ok := tags["reqName"]; ok {
			reqField = rn
		}

		reqVal := getFromRequest(c, tags["reqPlace"], reqField)
		if binder, ok := bindingRules[tags["dataType"]]; ok {
			finalVal, err := binder(c, reqVal, tags)
			if err != nil {
				return err
			}
			vals.Field(i).Set(reflect.ValueOf(finalVal))
			continue
		}
		vals.Field(i).Set(reflect.ValueOf(reqVal))
	}
	return nil
}
```

## 5. Considerations & Best Practices

- Refleksi runtime memiliki overhead minimal dibandingkan manual parsing, namun memberikan fleksibilitas tinggi untuk complex query filtering.
- Penggunaan io.NopCloser pada request body mutlak diperlukan agar Gin context dan middleware lain tetap dapat membaca payload setelah inspeksi.
- Penanganan dive tag memungkinkan nested struct query parsing secara rekursif.

## 6. Related Knowledge

- [[01-Knowledge/patterns/backend/fluent-typed-http-request-pipeline-with-dynamic-mime-content-marshalling.md]]
- Reflection Based Data Binding

## 7. Source

- [[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]
