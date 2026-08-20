---
title: "Declarative Multi-Tier Route Guarding"
type: pattern
tags: [pattern, mobile, flutter, auto-route, navigation, route-guard, security]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787116045805-ce1291de
sources: ["Harvest 1787116045805 Ce1291de.json"]
---

# Declarative Multi-Tier Route Guarding

Pola routing guard berantai deklaratif dengan AutoRoute untuk validasi sesi, onboarding, dan verifikasi akun.

## 1. Overview & Architecture

Pola guard navigasi bertingkat menggunakan pustaka AutoRoute untuk memvalidasi status autentikasi, status onboarding akun baru, dan kelengkapan verifikasi profil sebelum mengizinkan transisi ke layar tertentu.

## 2. Implementation & Code Structure

lib/
└── router/
    ├── app_route.dart
    ├── app_route.gr.dart
    ├── dashboard_route_guard.dart
    └── login_route_guard.dart

## 3. Key Implementation Points

- AutoRouteGuard memotong siklus navigasi sebelum rute target ditampilkan.
- Evaluasi bertingkat status sesi: Login -> New User -> Profile Verification -> Main Dashboard.
- Pemisahan logika otorisasi rute dari lapisan UI view/screen.

## 4. Code Examples

### AutoRouteGuard implementation checking authentication, onboarding, and verification state before resolving navigation.

```dart
import 'package:auto_route/auto_route.dart';
import 'package:lovelet_lookin/common/resource/strings.dart';
import 'package:lovelet_lookin/common/util/cache_util.dart';
import 'package:lovelet_lookin/router/app_route.dart';

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

- Pastikan pengecekan cache/persisted session dilakukan secara non-blocking atau dengan splash screen fallback.
- Gunakan resolver.next(false) atau route push eksplisit untuk mencegah navigation loop.

## 6. Related Knowledge

- Routing Guards
- Auto Route Flutter

## 7. Source

- Harvest 1787116045805 Ce1291de.json
