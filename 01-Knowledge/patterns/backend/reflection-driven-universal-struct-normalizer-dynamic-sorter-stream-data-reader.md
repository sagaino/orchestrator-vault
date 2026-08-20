---
title: "Reflection-Driven Universal Struct Normalizer, Dynamic Sorter & Stream Data Reader"
type: pattern
tags: [pattern, backend, reflection, data-normalization, stream-reader, in-memory-sorting]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787128418632-596cfbe7
sources: ["Harvest 1787128418632 596cfbe7.json"]
---

# Reflection-Driven Universal Struct Normalizer, Dynamic Sorter & Stream Data Reader

Reflection-Driven Universal Struct Normalizer & Stream Data Reader memungkinkan manipulasi slice data dinamis, deduplikasi entitas, serta konversi base64 payload menjadi streaming io.Reader secara efisien.

## 1. Overview & Architecture

Service normalization engine berbasis Go reflection untuk melakukan manipulasi in-memory, deduplikasi, dan dynamic sorting pada slice of struct heterogen tanpa boilerplate comparator, serta adapter stream data base64 ke io.Reader.

## 2. Implementation & Code Structure

pkg/
└── mapper/
    ├── default.go
    ├── formatter.go
    ├── reader.go
    └── slice.go

## 3. Key Implementation Points

- Dynamic dispatching sorting function berdasarkan reflect.Kind (string, int, uint, time.Time).
- Deduplikasi koleksi slice of struct via UniqueByStructField dan ContainByStructField.
- MIME prefix stripping aman untuk data URL base64 image/file uploads.
- Humanized duration formatting (HH:MM:SS -> Hari/Jam).

## 4. Code Examples

### Dynamic reflection struct sorting engine.

```go
var (
	stringAsc = func(prev reflect.Value, next reflect.Value, fieldName string) (interface{}, interface{}) {
		prevStr := strings.ToLower(prev.FieldByName(fieldName).String())
		nextStr := strings.ToLower(next.FieldByName(fieldName).String())

		baseRange := len(nextStr)
		if len(prevStr) < len(nextStr) {
			baseRange = len(prevStr)
		}

		for i := 0; i < baseRange; i++ {
			switch {
			case prevStr[i] < nextStr[i]:
				return prev.Interface(), next.Interface()
			case prevStr[i] > nextStr[i]:
				return next.Interface(), prev.Interface()
			}
		}

		if len(prevStr) > len(nextStr) {
			return next.Interface(), prev.Interface()
		}
		return prev.Interface(), next.Interface()
	}

	sortingFunc = map[SortingDirection]map[reflect.Kind]func(prev reflect.Value, next reflect.Value, fieldName string) (interface{}, interface{}){
		SortingAscending: {
			reflect.String: stringAsc,
			reflect.Int: func(prev, next reflect.Value, fieldName string) (interface{}, interface{}) {
				if prev.FieldByName(fieldName).Int() > next.FieldByName(fieldName).Int() {
					return next.Interface(), prev.Interface()
				}
				return prev.Interface(), next.Interface()
			},
			KindIsTime: func(prev, next reflect.Value, fieldName string) (interface{}, interface{}) {
				prevDate := prev.FieldByName(fieldName).Interface().(time.Time)
				nextDate := next.FieldByName(fieldName).Interface().(time.Time)
				if prevDate.After(nextDate) {
					return next.Interface(), prev.Interface()
				}
				return prev.Interface(), next.Interface()
			},
		},
	}
)

func (m Mapper) SortingByStructField(vals interface{}, fieldName string, sorting SortingDirection) interface{} {
	pVal := reflect.ValueOf(vals)
	if pVal.Kind() != reflect.Slice && pVal.Kind() != reflect.Array {
		return vals
	}

	newArr := make([]interface{}, pVal.Len())
	for i := 0; i < pVal.Len(); i++ {
		newArr[i] = pVal.Index(i).Interface()
	}

	for i := 0; i < len(newArr)-1; i++ {
		currentVal := pVal.Index(i).FieldByName(fieldName)
		kind := currentVal.Kind()
		if currentVal.Kind() == reflect.Struct && reflect.TypeOf(currentVal.Interface()).Name() == "Time" {
			kind = KindIsTime
		}
		fn := sortingFunc[sorting][kind]
		prev, next := fn(reflect.ValueOf(newArr[i]), reflect.ValueOf(newArr[i+1]), fieldName)
		newArr[i] = prev
		newArr[i+1] = next
	}

	return newArr
}
```

### Base64 data URL MIME stripper and stream reader adapter.

```go
func (m Mapper) Base64ToReader(base64String string) (io.Reader, int64, error) {
	// Remove data URL prefix if present (e.g., "data:image/png;base64,")
	if len(base64String) > 0 {
		if commaIndex := bytes.IndexByte([]byte(base64String), ','); commaIndex != -1 {
			base64String = base64String[commaIndex+1:]
		}
	}

	// Decode the base64 string
	decodedData, err := base64.StdEncoding.DecodeString(base64String)
	if err != nil {
		return nil, 0, fmt.Errorf("failed to decode base64: %w", err)
	}

	// Create a reader from the decoded data
	reader := bytes.NewReader(decodedData)
	return reader, int64(len(decodedData)), nil
}
```

## 5. Considerations & Best Practices

- Operasi reflection memiliki overhead komputasi CPU lebih tinggi dibandingkan direct type-safe slice sorting; gunakan untuk koleksi in-memory dinamis skala moderat.
- Base64ToReader mengembalikan panjang byte riil hasil dekode yang penting untuk Content-Length saat streaming ke S3/MinIO.

## 6. Related Knowledge

- Reflection Struct Mapper
- Base64 Stream Reader

## 7. Source

- Harvest 1787128418632 596cfbe7.json
