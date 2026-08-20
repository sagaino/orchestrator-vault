---
title: "Keychain-Backed Secure Session Persistence"
type: pattern
tags: [pattern, mobile, flutter, security, keychain, keystore, storage, persistence]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787116045805-ce1291de
sources: ["Harvest 1787116045805 Ce1291de.json"]
---

# Keychain-Backed Secure Session Persistence

Abstraksi secure storage berbasis hardware-backed keychain untuk manajemen kredensial dan sesi mobile.

## 1. Overview & Architecture

Pola penyimpanan data persisten lokal berorientasi keamanan tinggi menggunakan iOS Keychain dan Android KeyStore melalui wrapper CacheUtil yang seragam dan type-safe.

## 2. Implementation & Code Structure

lib/
└── common/
    ├── resource/
    │   └── strings.dart
    └── util/
        └── cache_util.dart

## 3. Key Implementation Points

- Abstraksi statis CacheUtil membungkus flutter_keychain untuk platform iOS Keychain dan Android KeyStore.
- Menyediakan serializer type-safe untuk String, Boolean, Integer, dan Uint8List binary.
- Sentralisasi key string konstan untuk mencegah typo pada key storage.

## 4. Code Examples

### Static helper encapsulating hardware-backed iOS Keychain and Android Keystore with type-safe operations.

```dart
import 'dart:typed_data';
import 'package:flutter_keychain/flutter_keychain.dart';

class CacheUtil {
  CacheUtil._();

  static Future<void> putString(String key, String value) async {
    await FlutterKeychain.put(key: key, value: value);
  }

  static Future<void> putBoolean(String key, bool value) async {
    await FlutterKeychain.put(key: key, value: value.toString());
  }

  static Future<String> getString(String key) async {
    final value = await FlutterKeychain.get(key: key);
    return value ?? '';
  }

  static Future<bool> getBoolean(String key) async {
    final value = await FlutterKeychain.get(key: key);
    return value == 'true';
  }

  static Future<Uint8List?> getUint8List(String key) async {
    final value = await FlutterKeychain.get(key: key);
    if (value == null) return null;
    List<String> stringList =
        value.replaceAll('[', '').replaceAll(']', '').split(', ');
    List<int> intList = stringList.map(int.parse).toList();
    return Uint8List.fromList(intList);
  }

  static Future<void> remove(String key) async {
    await FlutterKeychain.remove(key: key);
  }

  static Future<void> clear() async {
    FlutterKeychain.clear();
  }
}
```

## 5. Considerations & Best Practices

- Keychain I/O bersifat asynchronous; hindari pemanggilan berulang dalam synchronous build methods.
- Pada reset sesi/logout, panggil CacheUtil.clear() untuk menjamin data sensitif terhapus dari hardware secure storage.

## 6. Related Knowledge

- Secure Storage
- Session Persistence

## 7. Source

- Harvest 1787116045805 Ce1291de.json
