---
title: "Unified Error Handling & BaseResponse Mapping Pattern"
type: pattern
tags: [pattern, mobile, flutter, error-handling, network, clean-architecture]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787116476503-650c4b2e
sources: ["Harvest 1787116476503 650c4b2e.json"]
---

# Unified Error Handling & BaseResponse Mapping Pattern

Standarisasi parsing error response backend menjadi typed ErrorData domain model dengan centralized user feedback.

## 1. Overview & Architecture

Arsitektur penanganan error terpadu yang memetakan response network dan exception server menjadi format domain model ErrorData yang konsisten untuk konsumsi UI.

## 2. Implementation & Code Structure

lib/
├── common/
│   └── util/
│       └── error_util.dart
├── data/
│   └── api/
│       └── base/
│           ├── base_data.dart
│           └── base_response.dart
└── model/
    └── error_data.dart

## 3. Key Implementation Points

- Generic response envelope BaseResponse<T> untuk deserialisasi JSON response backend yang konsisten.
- Typed exception class ErrorData pembungkus kode error dan pesan.
- Sentralisasi penanganan error di repository layer dan UI feedback mapping di ErrorUtil.

## 4. Code Examples

### Definisi model exception terstandar dan mapping respon error server ke notifikasi UI user-friendly.

```dart
class ErrorData implements Exception {
  final int? code;
  final String? message;

  ErrorData({this.code, this.message});
}

class ErrorUtil {
  static const int codeRefreshTokenExpiredError = 401;
  static const int codeTokenInvalidError = 403;
  static const int codeUserNotFoundError = 404;

  static void showToast(ErrorData error) {
    if ((error.code ?? 0) < 500) {
      toast(error.message ?? 'Terjadi kesalahan');
    } else {
      toast('Terjadi gangguan pada sistem. Silakan coba lagi.');
    }
  }
}
```

## 5. Considerations & Best Practices

- Repository mengubah DioException / HttpError menjadi domain-level ErrorData sebelum diteruskan ke layer UI/Presenter.
- Error dengan kode status >= 500 disamarkan dengan pesan generik untuk mencegah kebocoran informasi teknis server ke user.

## 6. Related Knowledge

- Error Handling Architecture
- Api Envelope Parsing

## 7. Source

- Harvest 1787116476503 650c4b2e.json
