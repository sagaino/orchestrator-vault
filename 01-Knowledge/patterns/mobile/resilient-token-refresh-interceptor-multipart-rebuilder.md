---
title: "Resilient Token Refresh Interceptor & Multipart Rebuilder"
type: pattern
tags: [pattern, mobile, flutter, dio, networking, jwt, authentication, interceptor]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787116045805-ce1291de
sources: ["Harvest 1787116045805 Ce1291de.json"]
---

# Resilient Token Refresh Interceptor & Multipart Rebuilder

Pola interceptor HTTP untuk auto-refresh JWT token yang mendukung request replay transparan dan rekonstruksi payload multipart FormData.

## 1. Overview & Architecture

Interceptor Dio cerdas yang mendeteksi kedaluwarsa token (HTTP 401 / expired code), mengeksekusi refresh token di background, merekonstruksi payload request (termasuk multipart file upload FormData), dan melakukan retry request asli secara transparan.

## 2. Implementation & Code Structure

lib/
└── data/
    └── api/
        ├── default_interceptor.dart
        ├── http_client.dart
        ├── http_routes.dart
        └── refresh_token_interceptor.dart

## 3. Key Implementation Points

- Intercept response HTTP 401 dan periksa apakah error dapat dipulihkan via refresh token.
- Clone dan rekonstruksi binary stream / multipart FormData sebelum melakukan request replay.
- Simpan token baru ke secure cache dan perbarui header Authorization secara atomik.
- Picu sinkronisasi FCM token baru jika terdeteksi token device belum terdaftar.

## 4. Code Examples

### Dio interceptor for automatic token refresh, recursive replay, and multipart FormData rebuilding.

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
        final baseResponse = BaseResponse.fromJson(json: response.data);
        if (baseResponse.status == ErrorUtil.codeRefreshTokenExpiredError ||
            baseResponse.status == ErrorUtil.codeTokenInvalidError ||
            baseResponse.status == ErrorUtil.codeUserNotFoundError) {
          Util.logout(context: appRouter.navigatorKey.currentContext);
          return;
        }
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

  Future<FormData> _remakeFormData(
      FormData oldData, Map<String, dynamic> requestExtra) async {
    final Map<String, dynamic> newFormData = Map.fromEntries(oldData.fields);
    for (var file in oldData.files) {
      newFormData.putIfAbsent(file.key, () => file.value.clone());
    }
    return FormData.fromMap(newFormData);
  }
}
```

## 5. Considerations & Best Practices

- FormData Dio yang sudah dikonsumsi stream-nya tidak dapat digunakan ulang tanpa cloning eksplisit via _remakeFormData().
- Terminal authentication error codes harus langsung memicu centralized logout flow untuk mencegah infinite refresh loop.

## 6. Related Knowledge

- Jwt Authentication
- Dio Interceptor

## 7. Source

- Harvest 1787116045805 Ce1291de.json
