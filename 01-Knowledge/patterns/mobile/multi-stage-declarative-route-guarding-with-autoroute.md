---
title: "Multi-Stage Declarative Route Guarding with AutoRoute"
type: pattern
tags: [pattern, mobile, flutter, autoroute, navigation, route-guard, onboarding]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787125853633-86cbed7a
sources: ["Harvest 1787125853633 86cbed7a.json"]
---

# Multi-Stage Declarative Route Guarding with AutoRoute

Pola navigasi berjenjang menggunakan AutoRoute Guard untuk validasi autentikasi, onboarding, dan verifikasi biometrik.

## 1. Overview & Architecture

Pola penjagaan rute bertingkat (multi-stage navigation guard) yang memvalidasi otentikasi, status new user onboarding, dan status verifikasi profil sebelum user masuk ke dashboard.

## 2. Implementation & Code Structure

lib/
└── router/
    ├── app_route.dart
    ├── app_route.gr.dart
    ├── login_route_guard.dart
    └── dashboard_route_guard.dart

## 3. Key Implementation Points

- Evaluasi berjenjang: isLogin -> isNewUser -> isVerified -> RouteDashboard / RouteEditProfile / RouteNewAccount
- Penyederhanaan routing deklaratif AutoRoute 8 dengan integrasi resolver.next(true) dan router.push()

## 4. Code Examples

### AutoRoute multi-stage onboarding and authentication guard

```dart
class LoginRouteGuard extends AutoRouteGuard {
  @override
  void onNavigation(NavigationResolver resolver, StackRouter router) async {
    CacheUtil.getBoolean(Strings.keyIsLogin).then(
      (isLogin) {
        if (isLogin) {
          CacheUtil.getBoolean(Strings.keyIsNewUser).then((isNewUser) {
            if (isNewUser) {
              router.push(const RouteNewAccount());
            } else {
              CacheUtil.getBoolean(Strings.keyIsVerified).then(
                (isVerified) {
                  if (isVerified) {
                    router.push(RouteDashboard());
                  } else {
                    router.push(RouteEditProfile(isInitialPage: true));
                  }
                },
              );
            }
          });
        } else {
          resolver.next(true);
        }
      },
    );
  }
}
```

## 5. Considerations & Best Practices

- Guard async harus cepat agar tidak menghambat initial render aplikasi pada cold start
- Pastikan state keychain diupdate secara konsisten saat user menyelesaikan tahapan registrasi/verifikasi

## 6. Related Knowledge

- [[01-Knowledge/patterns/mobile/autoroute-multi-stage-navigation-onboarding-guard.md]]
- [[01-Knowledge/patterns/mobile/declarative-multi-tier-route-guarding.md]]

## 7. Source

- Harvest 1787125853633 86cbed7a.json
