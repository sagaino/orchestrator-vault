---
title: "Cross-Platform Native Billing & StoreKit Queue Delegate"
type: pattern
tags: [pattern, mobile, flutter, iap, in-app-purchase, storekit, google-play-billing]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787125853633-86cbed7a
sources: ["Harvest 1787125853633 86cbed7a.json"]
---

# Cross-Platform Native Billing & StoreKit Queue Delegate

Pola integrasi in-app purchase lintas platform dengan manajemen antrean transaksi StoreKit iOS dan siklus konfirmasi pembelian.

## 1. Overview & Architecture

Pola integrasi In-App Purchase terpadu untuk iOS dan Android dengan penanganan reactive stream, delegate antrean StoreKit, dan verifikasi receipt backend.

## 2. Implementation & Code Structure

lib/
├── common/service/billing_service.dart
├── presenter/priority_pass/
│   ├── notifier/ios_payment_queue_delegate.dart
│   └── notifier/priority_pass_notifier.dart
└── model/purchasable_product_model.dart

## 3. Key Implementation Points

- Wrapper stream-based lintas platform Google Play Billing dan Apple StoreKit
- Implementasi IosPaymentQueueDelegate untuk mengelola antrean transaksi pembayaran iOS StoreKit
- Siklus lengkap verifikasi status (pending, purchased, error, restored, canceled) dan acknowledgment transaksi

## 4. Code Examples

### Unified In-App Purchase service with iOS StoreKit payment queue delegate and purchase stream lifecycle

```dart
class BillingService {
  late final InAppPurchase _inAppPurchase;
  late final Stream<List<PurchaseDetails>> _stream;

  BillingService() {
    _inAppPurchase = InAppPurchase.instance;
    _stream = _inAppPurchase.purchaseStream;
  }

  void initPayment({
    required Future<void> Function(PurchaseDetails) onVerifyPurchase,
    required Future<void> Function() onPendingCompletePurchase,
    required Future<void> Function(IAPError, PurchaseDetails) onStatusError,
  }) async {
    _stream.listen((purchaseDetailsList) async {
      for (var purchaseDetails in purchaseDetailsList) {
        if (purchaseDetails.status == PurchaseStatus.purchased) {
          await onVerifyPurchase(purchaseDetails);
        } else if (purchaseDetails.status == PurchaseStatus.error) {
          onStatusError(purchaseDetails.error!, purchaseDetails);
        }
        if (purchaseDetails.pendingCompletePurchase) {
          onPendingCompletePurchase();
          _inAppPurchase.completePurchase(purchaseDetails);
        }
      }
    });

    if (Platform.isIOS) {
      var iosAddition = _inAppPurchase
          .getPlatformAddition<InAppPurchaseStoreKitPlatformAddition>();
      await iosAddition.setDelegate(IosPaymentQueueDelegate());
    }
  }
}
```

## 5. Considerations & Best Practices

- Panggilan completePurchase() wajib dilakukan setelah verifikasi backend berhasil untuk menghindari auto-refund oleh Google Play / App Store
- Bersihkan antrean transaksi iOS dengan finishTransaction saat dispose service

## 6. Related Knowledge

- [[01-Knowledge/patterns/mobile/decoupled-native-push-notification-in-app-purchase-pattern.md]]

## 7. Source

- Harvest 1787125853633 86cbed7a.json
