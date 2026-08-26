---
supersession: "ACTIVE"
review_by: "2027-02-22"
owner: "knowledge-curator"
confidence: 0.5
provenance_schema: 1
title: "context-aware-temporal-virtualization-and-timezone-normalization-engine"
type: pattern
tags: [pattern, backend, golang, time-service, temporal, formatting, utility]
created: 2026-08-26
updated: 2026-08-26
orchestrator_run: "harvest-1787715147310-7bd85203"
sources: ["[[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]"]
---

# context-aware-temporal-virtualization-and-timezone-normalization-engine

Mesin virtualisasi waktu berbasis konteks dan utilitas transformasi data yang menormalisasi operasi tanggal/waktu multi-zona serta menyediakan formatting memori reflektif.

## 1. Overview & Architecture

Pola Context-Aware Temporal Virtualization and Timezone Normalization Engine menyediakan abstraksi waktu deterministik dan modul transformasi data serbaguna. Pola ini memvirtualisasikan instan waktu berdasarkan zona waktu pengguna yang diinjeksi ke context dan menyediakan suite utilitas formatting/parsing data in-memory.

## 2. Implementation & Code Structure

pkg/
├── clock/
│   └── time.go             # Legacy context-aware clock abstraction with payload fallback
├── time/
│   └── time.go             # Clock interface, context timezone injection, UTC defaults
└── mapper/
    ├── formatter.go        # Reflection sorting (ASC/DESC), slice deduplication, duration parsing
    └── reader.go           # Float precision rounding, Base64 MIME reader decoding

## 3. Key Implementation Points

- Kontrak antarmuka Clock yang mengabstraksikan fungsi temporal inti (Now, NowUnix, ParseInLocation).
- Propagasi zona waktu pengguna (time.Location) secara transparan melalui context.Context.
- Helper transformasi data terisolasi: dynamic duration string builder (ParseServiceDurationFormat) dan MIME-stripped Base64 to io.Reader stream converter.
- Generic reflection-based in-memory sorting untuk tipe data String, Int, Uint, dan time.Time.

## 4. Code Examples

### Context-aware temporal clock provider and reflection-based slice transformer

```go
type Clock interface {
	ParseWithTzFromCtx(ctx context.Context, value string, format string) time.Time
	Now(ctx context.Context) time.Time
	NowUnix() int64
	GetTimeZoneByName(name string) *time.Location
	SetTimezoneToContext(ctx context.Context, val string) context.Context
	GetTimezoneFromContext(ctx context.Context) *time.Location
}

func (t clock) Now(ctx context.Context) time.Time {
	lz := t.GetTimezoneFromContext(ctx)
	return time.Now().In(lz)
}

func (t clock) GetTimezoneFromContext(ctx context.Context) *time.Location {
	lz := time.UTC
	if ct, ok := ctx.Value(CtxKeyTimezone).(time.Location); ok {
		lz = &ct
	}
	return lz
}

func (m Mapper) SortingByStructField(vals interface{}, fieldName string, sorting SortingDirection) interface{} {
	pVal := reflect.ValueOf(vals)
	if !m.typeIsArray(vals) {
		return vals
	}

	var newArr = make([]interface{}, pVal.Len())
	for i := 0; i < pVal.Len(); i++ {
		newArr[i] = pVal.Index(i).Interface()
	}

	for i := 0; i < len(newArr)-1; i++ {
		currentVal := pVal.Index(i).FieldByName(fieldName)
		kind := currentVal.Kind()
		if currentVal.Kind() == reflect.Struct {
			pType := reflect.TypeOf(currentVal.Interface())
			if pType.Name() == "Time" {
				kind = KindIsTime
			}
		}
		fn := sortingFunc[sorting][kind]
		prev, next := fn(reflect.ValueOf(newArr[i]), reflect.ValueOf(newArr[i+1]), fieldName)
		newArr[i] = prev
		newArr[i+1] = next
	}
	return newArr
}
```

## 5. Considerations & Best Practices

- Injeksi Clock interface memfasilitasi deterministic unit testing tanpa monkey patching time.Now().
- Fallback selalu ke time.UTC saat header atau klaim timezone tidak ditemukan di context.
- Operasi refleksi pada slice formatting (sorting/dedup) ditujukan untuk response collection ukuran moderat di memory.

## 6. Related Knowledge

- [[01-Knowledge/patterns/backend/generic-type-safe-repository-with-atomic-transaction-coordinator.md]]
- Deterministic Temporal Testing

## 7. Source

- [[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]
