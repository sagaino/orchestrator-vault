---
title: "Dual BaseHttpClient Architecture with Alice & Logger Interception"
type: pattern
tags: [pattern, mobile, flutter, dio, networking, alice, logging]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787125853633-86cbed7a
sources: ["Harvest 1787125853633 86cbed7a.json"]
---

# Dual BaseHttpClient Architecture with Alice & Logger Interception

Pola arsitektur dual HTTP client Dio dengan environment-aware interceptor untuk logging, inspeksi Alice, dan validasi status response.

## 1. Overview & Architecture

Pola konfigurasi networking terpusat dengan segregasi endpoint Core API dan External AI API, dilengkapi interceptor inspeksi Alice dan Pretty Logging terkondisi.

## 2. Implementation & Code Structure

lib/
└── data/
    └── api/
        ├── api.dart
        ├── http_client.dart
        ├── default_interceptor.dart
        └── refresh_token_interceptor.dart

## 3. Key Implementation Points

- Pemisahan httpProvider (Core API) dan httpExternalProvider (AI/LLM microservice)
- Injeksi Alice Inspector dengan shake-to-inspect di debug/non-production
- Custom validateStatus < 502 agar error 4xx/500 dapat diproses interceptor tanpa uncaught exception awal

## 4. Code Examples

### Dual BaseHttpClient configuration with environment conditional logging and Alice inspector

```dart
final httpProvider = Provider<BaseHttpClient>((ref) {
  final httpClient = BaseHttpClient(
    interceptor: <Interceptor>[
      DefaultInterceptor(),
      if (kDebugMode || !EnvConfig.getInstance().isProduction())
        HttpFormatter(
          logger: Logger(
            printer: PrettyPrinter(
              methodCount: 0,
              colors: false,
              printTime: false,
              printEmojis: false,
            ),
          ),
        ),
    ],
    option: BaseOptions(
      baseUrl: EnvValue.read().baseUrl,
      followRedirects: true,
      validateStatus: (int? status) => status == null ? false : status < 502,
      sendTimeout: const Duration(seconds: 60),
      contentType: Headers.formUrlEncodedContentType,
      connectTimeout: const Duration(seconds: 60),
      receiveTimeout: const Duration(seconds: 60),
    ),
  );

  final RefreshTokenInterceptor refreshTokenInterceptor =
      RefreshTokenInterceptor(httpClient, ref.container);
  httpClient.addInterceptor(refreshTokenInterceptor);
  if (kDebugMode || !EnvConfig.getInstance().isProduction()) {
    httpClient.addInterceptor(ref.read(aliceProvider).getDioInterceptor());
  }

  return httpClient;
});
```

## 5. Considerations & Best Practices

- Pastikan Alice Inspector dan HttpFormatter tidak aktif pada build release production untuk keamanan dan performa
- Gunakan BaseHttpClient wrapper untuk sentralisasi konfigurasi timeout dan SSL certificate bypass pada dev flavor

## 6. Related Knowledge

- [[01-Knowledge/patterns/mobile/dio-interceptor-token-refresh-request-replay-pattern.md]]

## 7. Source

- Harvest 1787125853633 86cbed7a.json
