---
title: "Polymorphic DTO Serializer with Fallback Type Converters"
type: pattern
tags: [pattern, mobile, flutter, freezed, json-serializable, dto, type-safety]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787125853633-86cbed7a
sources: ["Harvest 1787125853633 86cbed7a.json"]
---

# Polymorphic DTO Serializer with Fallback Type Converters

Pola serialisasi DTO polimorfik dengan custom fallback JsonConverter untuk ketahanan parsing respons backend.

## 1. Overview & Architecture

Pola serialisasi DTO defensif yang menggabungkan Freezed dengan custom JsonConverter untuk menjamin parsing payload JSON backend tidak memicu runtime crash pada tipe data polimorfik.

## 2. Implementation & Code Structure

lib/
├── model/
│   └── converter.dart
├── data/api/
│   ├── base/base_response.dart
│   └── dto/

## 3. Key Implementation Points

- Custom JsonConverter untuk konversi String <-> int/num tanpa fatal parse error
- Penanganan anomali server yang mengembalikan array tunggal alih-alih nilai skalar via ArrayIntOrIntConverter
- Generic BaseResponse<T> untuk parsing aman single data, dataset list, dan pagination metadata

## 4. Code Examples

### Polymorphic JSON converters parsing numbers, strings, or arrays with zero crash guarantee

```dart
class StringOrIntConverterNum implements JsonConverter<num?, dynamic> {
  const StringOrIntConverterNum();

  @override
  num? fromJson(dynamic json) {
    if (json is int) return json;
    if (json is String) return int.tryParse(json);
    return null;
  }

  @override
  toJson(num? object) => object;
}

class ArrayIntOrIntConverter implements JsonConverter<num?, dynamic> {
  const ArrayIntOrIntConverter();

  @override
  num? fromJson(dynamic json) {
    if (json is num) return json;
    if (json is List) {
      final list = (json).map<int>((e) => e).toList();
      if (list.isEmpty) return null;
      return list[0];
    }
    return null;
  }

  @override
  toJson(num? object) => object;
}
```

## 5. Considerations & Best Practices

- Gunakan converter khusus pada field API yang memiliki riwayat inkonsistensi tipe data antar microservice backend

## 6. Related Knowledge

- [[01-Knowledge/patterns/mobile/generic-dto-envelope-domain-error-normalization.md]]
- [[01-Knowledge/patterns/mobile/unified-error-handling-baseresponse-mapping-pattern.md]]

## 7. Source

- Harvest 1787125853633 86cbed7a.json
