---
title: "AutoRoute Multi-Stage Navigation & Onboarding Guard"
type: pattern
tags: [pattern, mobile, flutter, navigation, autoroute, guards, security]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787116476503-650c4b2e
sources: ["Harvest 1787116476503 650c4b2e.json"]
---

# AutoRoute Multi-Stage Navigation & Onboarding Guard

Proteksi navigasi deklaratif dengan AutoRouteGuard untuk routing bertingkat berdasarkan status autentikasi, akun baru, dan verifikasi user.

## 1. Overview & Architecture

Pola proteksi rute navigasi bertingkat menggunakan AutoRouteGuard. Guard mengevaluasi state session, kelengkapan registrasi, dan verifikasi biometrik pengguna sebelum mengizinkan transisi antar layar.

## 2. Implementation & Code Structure

lib/
└── router/
    ├── app_route.dart
    ├── app_route.gr.dart
    ├── dashboard_route_guard.dart
    └── login_route_guard.dart

## 3. Key Implementation Points

- Implementasi subclass AutoRouteGuard yang meng-override method onNavigation.
- Multi-level branching evaluation: isLogin -> isNewUser -> isVerified -> Destination.
- Pengalihan navigasi otomatis ke flow registrasi/verifikasi profil bila syarat belum terpenuhi.

## 4. Code Examples

### Implementasi AutoRouteGuard bertingkat untuk mengevaluasi status onboarding dan autentikasi pengguna secara deklaratif.

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

- Semua pengecekan status kredensial session diambil secara asinkron dari Secure Storage/CacheUtil sebelum route resolver diizinkan lanjut.
- Route guard terpasang secara deklaratif pada router configuration (@AutoRouterConfig) sehingga tidak ada bypass navigasi di tingkat widget.

## 6. Related Knowledge

- Autoroute Guards
- Declarative Navigation

## 7. Source

- Harvest 1787116476503 650c4b2e.json
