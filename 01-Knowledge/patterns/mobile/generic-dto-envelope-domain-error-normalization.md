---
title: "Generic DTO Envelope & Domain Error Normalization"
type: pattern
tags: [pattern, mobile, flutter, clean-architecture, networking, error-handling, dto]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787116045805-ce1291de
sources: ["Harvest 1787116045805 Ce1291de.json"]
---

# Generic DTO Envelope & Domain Error Normalization

Pola generic envelope DTO dan normalisasi exception jaringan menjadi domain error terstruktur di repository layer.

## 1. Overview & Architecture

Pola deserialisasi respons API generik menggunakan Generic Envelope DTO yang dipadukan dengan pemetaan exception jaringan (DioException) ke dalam Domain ErrorData yang konsisten di lapisan Repository.

## 2. Implementation & Code Structure

lib/
├── data/
│   ├── api/
│   │   ├── base/
│   │   │   └── base_response.dart
│   │   └── dto/
│   │       ├── request/
│   │       └── response/
│   └── repository/
└── model/
    └── error_data.dart

## 3. Key Implementation Points

- Envelope generic BaseResponse<T> membungkus metadata server (status, code, message, error).
- Factory deserializer menerima callback parser dataFromJson khusus untuk generic T.
- Repository layer mengubah kegagalan API dan koneksi menjadi objek domain ErrorData yang seragam.

## 4. Code Examples

### Generic BaseResponse parser with nested DTO deserializer closure and typed ErrorData normalization.

```dart
class BaseResponse<T> {
  final int? status;
  final int? code;
  final String? message;
  final bool? error;
  final T? data;

  BaseResponse({
    this.status,
    this.code,
    this.message,
    this.error,
    this.data,
  });

  factory BaseResponse.fromJson({
    required Map<String, dynamic> json,
    T Function(Map<String, dynamic>)? dataFromJson,
  }) {
    return BaseResponse<T>(
      status: json['status'] as int?,
      code: json['code'] as int?,
      message: json['message'] as String?,
      error: json['error'] as bool?,
      data: json['data'] != null && dataFromJson != null
          ? dataFromJson(json['data'] as Map<String, dynamic>)
          : null,
    );
  }
}

// Penggunaan pada Repository:
@override
Future<SendOtpData?> sendOtp({required OtpRequest req}) async {
  try {
    final response = await service.sendOtp(request: req);
    final baseResponse = BaseResponse<SendOtpData>.fromJson(
        json: response.data, dataFromJson: SendOtpData.fromJson);
    if ((baseResponse.error ?? false)) {
      throw ErrorData(code: baseResponse.status, message: baseResponse.message);
    } else {
      return baseResponse.data;
    }
  } on DioException {
    throw ErrorData(code: 500, message: "System error :: network connection");
  }
}
```

## 5. Considerations & Best Practices

- Gunakan closure function dataFromJson untuk menghindari refleksi runtime pada parsing generik Dart.
- Selalu tangkap DioException di repository boundary dan konversi ke domain-level ErrorData.

## 6. Related Knowledge

- Api Contract Envelope
- Clean Architecture Flutter

## 7. Source

- Harvest 1787116045805 Ce1291de.json
