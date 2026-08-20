---
title: "Decoupled Native Push Notification & In-App Purchase Pattern"
type: pattern
tags: [pattern, mobile, flutter, fcm, notifications, in-app-purchase, native-integration]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787116476503-650c4b2e
sources: ["Harvest 1787116476503 650c4b2e.json"]
---

# Decoupled Native Push Notification & In-App Purchase Pattern

Integrasi modular untuk Firebase Push Notifications dan In-App Purchase dengan decoupled singleton services.

## 1. Overview & Architecture

Pola modularisasi integrasi layanan native (Firebase Cloud Messaging dan In-App Purchases) dalam service class mandiri yang terpisah dari logic presentation.

## 2. Implementation & Code Structure

lib/
└── common/
    ├── notifier/
    │   ├── notification_notifier.dart
    │   └── premium_user_notifier.dart
    └── service/
        ├── billing_service.dart
        └── notification_service.dart

## 3. Key Implementation Points

- Singleton pattern untuk manajemen lifecycle push notification dan in-app purchase.
- Penanganan notifikasi baik dalam kondisi foreground, background, maupun terminated state.
- Integrasi bridge antara service layer dan Riverpod Notifier untuk reaktivitas UI.

## 4. Code Examples

### Singleton service untuk mengelola inisialisasi FCM, listening event foreground/background, dan deep link routing.

```dart
class NotificationService {
  static final NotificationService _instance = NotificationService._internal();
  factory NotificationService() => _instance;
  NotificationService._internal();

  final FirebaseMessaging _fcm = FirebaseMessaging.instance;
  final FlutterLocalNotificationsPlugin _localNotification = FlutterLocalNotificationsPlugin();

  Future<void> initialize() async {
    await _fcm.requestPermission(alert: true, badge: true, sound: true);
    FirebaseMessaging.onMessage.listen((RemoteMessage message) {
      _showLocalNotification(message);
    });
    FirebaseMessaging.onMessageOpenedApp.listen((RemoteMessage message) {
      _handleNotificationClick(message);
    });
  }

  void _showLocalNotification(RemoteMessage message) {
    // Tampilkan local notification banner saat aplikasi di foreground
  }
  
  void _handleNotificationClick(RemoteMessage message) {
    // Deep linking routing berdasarkan payload data notification
  }
}
```

## 5. Considerations & Best Practices

- Inisialisasi permission FCM dan notification channel Android harus dikonfigurasi saat bootstrap aplikasi di main.dart.
- Integrasikan push token lifecycle dengan user session (sinkronisasi FCM token ke backend via AuthRepository saat login / refresh token).

## 6. Related Knowledge

- Fcm Push Notifications
- In App Billing

## 7. Source

- Harvest 1787116476503 650c4b2e.json
