---
title: "Replay-Capable Refresh Token Interceptor with FormData Multipart Cloning"
type: pattern
tags: [pattern, mobile, flutter, jwt, refresh-token, dio, multipart-upload]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787125853633-86cbed7a
sources: ["Harvest 1787125853633 86cbed7a.json"]
---

# Replay-Capable Refresh Token Interceptor with FormData Multipart Cloning

Pola interceptor refresh token tangguh dengan kemampuan cloning multipart FormData untuk replay request tanpa kehilangan payload biner.

## 1. Overview & Architecture

Interceptor Dio yang secara transparan menangani perpanjangan token OAuth/JWT saat menerima response 401, serta melakukan cloning FormData untuk replay request upload.

## 2. Implementation & Code Structure

lib/
└── data/
    └── api/
        ├── default_interceptor.dart
        └── refresh_token_interceptor.dart

## 3. Key Implementation Points

- Intersepsi 401 dan pemanggilan silent refresh token menggunakan platform/device id
- Cloning stream multipart file pada FormData saat mereplay request upload yang gagal karena 401
- Pembersihan session dan navigasi logout otomatis saat refresh token expired

## 4. Code Examples

### Resilient refresh token interceptor with FormData multipart cloning and automatic logout on expired session

```dart
class RefreshTokenInterceptor extends Interceptor {
  RefreshTokenInterceptor(this.baseHttpClient, this.parentContainer);
  final BaseHttpClient baseHttpClient;
  final ProviderContainer parentContainer;

  @override
  Future<void> onResponse(Response<dynamic> response, ResponseInterceptorHandler handler) async {
    try {
      final header = response.requestOptions.headers;
      if ((response.statusCode ?? 0) >= 400 && (response.statusCode ?? 0) < 500) {
        final baseResponse = BaseResponse.fromJson(json: response.data);
        if (baseResponse.status == ErrorUtil.codeRefreshTokenExpiredError ||
            baseResponse.status == ErrorUtil.codeRefreshTokenCredentialError ||
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
            header["Authorization"] = "Bearer ${await CacheUtil.getString(Strings.keyAccessToken)}";
            dynamic newData = response.requestOptions.data;
            if (newData is FormData) {
              newData = await _remakeFormData(newData, response.requestOptions.extra);
            }
            final Response<dynamic> value = await baseHttpClient.client
                .fetch<dynamic>(response.requestOptions.copyWith(headers: header, data: newData));
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

  Future<FormData> _remakeFormData(FormData oldData, Map<String, dynamic> requestExtra) async {
    final Map<String, dynamic> newFormData = Map.fromEntries(oldData.fields);
    for (var file in oldData.files) {
      newFormData.putIfAbsent(file.key, () => file.value.clone());
    }
    return FormData.fromMap(newFormData);
  }
}
```

## 5. Considerations & Best Practices

- FormData yang sudah dikonsumsi pada request awal tidak dapat dibaca ulang tanpa diclone via _remakeFormData
- Deteksi error code kadaluarsa harus memicu auto-logout ke login page secara reaktif

## 6. Related Knowledge

- [[01-Knowledge/patterns/mobile/resilient-token-refresh-interceptor-multipart-rebuilder.md]]

## 7. Source

- Harvest 1787125853633 86cbed7a.json
