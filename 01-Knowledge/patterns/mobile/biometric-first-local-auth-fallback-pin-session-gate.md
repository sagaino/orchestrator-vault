---
title: "Biometric-First Local Auth & Fallback PIN Session Gate"
type: pattern
tags: [pattern, mobile, flutter, security, local-auth, biometrics, pin, session-management]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787110568549-448b547b
sources: ["Harvest 1787110568549 448b547b.json"]
---

# Biometric-First Local Auth & Fallback PIN Session Gate

Biometric-First Local Auth & Fallback PIN Session Gate pattern in Flutter.

## 1. Overview & Architecture

Pola autentikasi lokal multi-tier untuk aplikasi perbankan/fintech yang memprioritaskan autentikasi biometrik perangkat keras (FaceID / Fingerprint) dengan fallback keypad PIN 6-digit terenkripsi dan proteksi backgrounding.

```text
[App Launch / Protected Route]
             │
             ▼
   [Check Device Biometrics] ───(Supported & Enabled)───► [Prompt Biometric (Fingerprint/FaceID)]
             │                                                          │
      (Not Supported)                                      ┌────────────┴────────────┐
             │                                             ▼ (Success)               ▼ (Fail / Cancel)
             ▼                                      [Grant Session]           [Switch to PIN Fallback]
   [6-Digit PIN Keypad] ◄────────────────────────────────────────────────────────────────┘
```

## 2. Implementation & Code Structure

lib/features/auth/bindings/auth_binding.dart
lib/features/auth/data/datasource/auth_local_datasource.dart
lib/features/auth/presentation/controller/auth_controller.dart
lib/features/auth/presentation/pages/auth_page.dart
lib/features/auth/presentation/pages/pin_page.dart

## 3. Key Implementation Points

- Hardware biometric authentication via local_auth with biometricOnly policy
- Custom 6-digit numeric PIN fallback with creation and 2-step confirmation validation
- Persistent session authentication gate stored securely in local settings storage
- Navigation offAllNamed to prevent user from popping back into auth screens

## 4. Code Examples

### Biometric authentication controller with PIN verification and session gate

```dart
class AuthController extends GetxController {
  final AuthLocalDatasource _authStorage = Get.find<AuthLocalDatasource>();
  final LocalAuthentication _auth = LocalAuthentication();

  final pin = ''.obs;
  final confirmPin = ''.obs;
  final pinError = RxnString();
  final isCreating = false.obs;
  final isConfirmStep = false.obs;
  static const int pinLength = 6;

  Future<bool> authenticateWithBiometrics() async {
    try {
      return await _auth.authenticate(
        localizedReason: 'Verify your identity to access Mini Bank',
        biometricOnly: true,
        persistAcrossBackgrounding: true,
      );
    } catch (_) {
      return false;
    }
  }

  Future<void> _verifyPin() async {
    final valid = await _authStorage.verifyPin(pin.value);
    if (valid) {
      await onAuthSuccess();
    } else {
      pin.value = '';
      pinError.value = 'Wrong PIN';
    }
  }
}
```

## 5. Considerations & Best Practices

- Clear PIN buffer memory immediately after verification to avoid memory extraction risks
- Configure persistAcrossBackgrounding to prevent race conditions when OS biometric dialog pops up
- Ensure offAllNamed is used upon login to wipe the authentication route stack

## 6. Related Knowledge

- Mobile App Security
- Session Lifecycle

## 7. Source

- Harvest 1787110568549 448b547b.json
