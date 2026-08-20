---
title: "Dio Interceptor Token Refresh & Request Replay Pattern"
type: pattern
tags: [pattern, mobile, flutter, dio, network, jwt, interceptor]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787116476503-650c4b2e
sources: ["Harvest 1787116476503 650c4b2e.json"]
---

# Dio Interceptor Token Refresh & Request Replay Pattern

Otomatisasi refresh token JWT dan request replay transparan pada Dio HTTP Client termasuk kloning multipart FormData.

## 1. Overview & Architecture

Pola otomatisasi pembaruan token otentikasi di lapisan HTTP client Dio. Mengeliminasi kegagalan request user saat access token expired dengan me-refresh token di background dan me-replay request original secara transparan.

## 2. Implementation & Code Structure

lib/
└── data/
    └── api/
        ├── api.dart
        ├── base/
        │   └── base_response.dart
        ├── default_interceptor.dart
        ├── http_client.dart
        ├── http_routes.dart
        └── refresh_token_interceptor.dart

## 3. Key Implementation Points

- Intersepsi onResponse untuk mendeteksi status code 401 atau token invalid.
- Pengambilan token baru menggunakan RefreshTokenRequest yang disimpan di Secure Storage.
- Rekonstruksi data payload request (termasuk FormData clone) dan eksekusi ulang (replay) fetch request dengan header Bearer baru.

## 4. Code Examples

### Interceptor Dio untuk mendeteksi HTTP 401, mengambil token baru via AuthRepository, me-remake FormData, dan me-replay request original secara otomatis.

```dart
class RefreshTokenInterceptor extends Interceptor {
  RefreshTokenInterceptor(this.baseHttpClient, this.parentContainer);
  final BaseHttpClient baseHttpClient;
  final ProviderContainer parentContainer;

  @override
  Future<void> onResponse(
    Response<dynamic> response,
    ResponseInterceptorHandler handler,
  ) async {
    try {
      final header = response.requestOptions.headers;
      if ((response.statusCode ?? 0) >= 400 && (response.statusCode ?? 0) < 500) {
        if ((response.statusCode ?? 0) == 401) {
          final SignUpData? value = await getRefreshToken();
          if (value == null) {
            handler.resolve(response);
          } else {
            header["Authorization"] =
                "Bearer ${await CacheUtil.getString(Strings.keyAccessToken)}";
            dynamic newData = response.requestOptions.data;
            if (newData is FormData) {
              newData = await _remakeFormData(newData, response.requestOptions.extra);
            }
            final Response<dynamic> value = await baseHttpClient.client
                .fetch<dynamic>(response.requestOptions
                .copyWith(headers: header, data: newData));
            handler.resolve(value);
            return;
          }
        }
      }
      super.onResponse(response, handler);
    } catch (e) {
      return;
    }
  }
}
```

## 5. Considerations & Best Practices

- Perlu me-reconstruct FormData saat me-replay request multipart (file/image upload) karena stream/buffer request original sudah terkonsumsi.
- Gunakan session clearing dan trigger navigasi logout global (Util.logout) jika refresh token kadaluarsa atau invalid.

## 6. Related Knowledge

- Dio Interceptors
- Token Refresh Replay

## 7. Source

- Harvest 1787116476503 650c4b2e.json
