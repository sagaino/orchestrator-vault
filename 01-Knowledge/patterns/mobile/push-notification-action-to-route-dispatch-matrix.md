---
title: "Push Notification Action-to-Route Dispatch Matrix"
type: pattern
tags: [pattern, mobile, flutter, fcm, firebase, notifications, deep-linking]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787125853633-86cbed7a
sources: ["Harvest 1787125853633 86cbed7a.json"]
---

# Push Notification Action-to-Route Dispatch Matrix

Pola arsitektur push notification FCM dengan mapping aksi bertipe enum untuk deep-linking dan pergantian tab dashboard otomatis.

## 1. Overview & Architecture

Pola dispatch aksi notifikasi terpusat yang memetakan payload FCM ke enum tipe aksi dan menavigasikan user ke halaman spesifik atau tab dashboard secara mulus.

## 2. Implementation & Code Structure

lib/
├── common/
│   ├── service/notification_service.dart
│   └── notifier/notification_notifier.dart
└── model/notification_data_model.dart

## 3. Key Implementation Points

- Mapping payload JSON notificationData.clickAction ke typed enum _notificationType
- Direct deep-navigation menggunakan appRouter.navigate dan manipulasi index dashboardNotifierProvider
- Sentralisasi penanganan notifikasi foreground, background, dan terminated app state

## 4. Code Examples

### Push notification click action resolver routing to nested AutoRoute screens and changing dashboard tabs

```dart
static void _handleClickAction(_notificationType notificationType, NotificationDataModel notificationData) {
  switch (notificationType) {
    case _notificationType.FLUTTER_NOTIFICATION_CLICK_CHAT:
      {
        final roomId = notificationData.additionalData?.roomId;
        appRouter.navigate(RouteMessage(
            userData: MessageUserModel(
                targetUserId: notificationData.actorId,
                roomId: roomId,
                isFetchRoom: true)));
      }
    case _notificationType.FLUTTER_NOTIFICATION_CLICK_DAILY_LIKE:
      {
        appRouter.navigate(RouteDashboard(initialIndex: 1));
        ProviderScope.containerOf(appRouter.navigatorKey.currentContext!)
            .read(dashboardNotifierProvider.notifier)
            .changeIndexPage(1);
      }
    case _notificationType.FLUTTER_NOTIFICATION_CLICK_MAILBOX:
      {
        appRouter.navigate(RouteDashboard(initialIndex: 0));
        ProviderScope.containerOf(appRouter.navigatorKey.currentContext!)
            .read(dashboardNotifierProvider.notifier)
            .changeIndexPage(0);
      }
    default: {}
  }
}
```

## 5. Considerations & Best Practices

- Background handler memerlukan anotasi @pragma('vm:entry-point') agar dikenali engine Flutter saat app terminated
- Cold-start navigation membutuhkan slight delay agar navigator stack terpasang sempurna

## 6. Related Knowledge

- [[01-Knowledge/patterns/mobile/decoupled-native-push-notification-in-app-purchase-pattern.md]]

## 7. Source

- Harvest 1787125853633 86cbed7a.json
