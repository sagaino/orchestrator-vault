---
title: "Hardware-Backed Secure Keychain Persistence Architecture"
type: pattern
tags: [pattern, mobile, flutter, security, keychain, keystore, cache]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787125853633-86cbed7a
sources: ["Harvest 1787125853633 86cbed7a.json"]
---

# Hardware-Backed Secure Keychain Persistence Architecture

Pola penyimpanan lokal aman menggunakan hardware keychain untuk kredensial, token sesi, dan serialisasi data biner biometrik.

## 1. Overview & Architecture

Wrapper penyimpanan lokal aman berbasis hardware security module (KeyStore di Android dan Keychain di iOS) untuk menyimpan token otentikasi, flag sesi, dan template biometrik.

## 2. Implementation & Code Structure

lib/
└── common/
    └── util/cache_util.dart

## 3. Key Implementation Points

- Abstraksi penyimpanan hardware-backed iOS Keychain Services dan Android KeyStore via flutter_keychain
- Serialisasi dan deserialisasi array biner Uint8List untuk face biometric embeddings
- Metode atomic clear() untuk menjamin seluruh data sesi terhapus bersih saat user logout

## 4. Code Examples

### Hardware-backed secure cache utility using FlutterKeychain with Uint8List template serialization

```dart
class CacheUtil {
  CacheUtil._();
  static Future<void> putString(String key, String value) async =>
      await FlutterKeychain.put(key: key, value: value);

  static Future<void> putBoolean(String key, bool value) async =>
      await FlutterKeychain.put(key: key, value: value.toString());

  static Future<String> getString(String key) async =>
      (await FlutterKeychain.get(key: key)) ?? '';

  static Future<bool> getBoolean(String key) async =>
      (await FlutterKeychain.get(key: key)) == 'true';

  static Future<Uint8List?> getUint8List(String key) async {
    final value = await FlutterKeychain.get(key: key);
    if (value == null) return null;
    List<String> stringList = value.replaceAll('[', '').replaceAll(']', '').split(', ');
    List<int> intList = stringList.map(int.parse).toList();
    return Uint8List.fromList(intList);
  }

  static Future<void> clear() async => FlutterKeychain.clear();
}
```

## 5. Considerations & Best Practices

- Gunakan untuk data sensitif, token, dan biometrics template; hindari menyimpan data biner berukuran besar (gambar penuh) di Keychain

## 6. Related Knowledge

- [[01-Knowledge/patterns/mobile/keychain-backed-secure-session-persistence.md]]

## 7. Source

- Harvest 1787125853633 86cbed7a.json
