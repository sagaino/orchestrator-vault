---
title: "Encrypted Compile-Time Secure DotEnv Configuration"
type: pattern
tags: [pattern, mobile, flutter, security, dotenv, environment-flavors]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787125853633-86cbed7a
sources: ["Harvest 1787125853633 86cbed7a.json"]
---

# Encrypted Compile-Time Secure DotEnv Configuration

Pola konfigurasi environment terenkripsi berbasis compile-time code generator untuk keamanan endpoint dan kredensial mobile.

## 1. Overview & Architecture

Pola enkripsi environment variables saat compile-time menggunakan secure_dotenv generator untuk mengamankan URL endpoint dan secret credentials dari reverse engineering.

## 2. Implementation & Code Structure

lib/
├── env/
│   ├── env_config.dart
│   ├── env_type.dart
│   ├── env_values.dart
│   └── env_values.g.dart
├── .env.development
├── .env.uat
├── .env.production
└── Makefile

## 3. Key Implementation Points

- Pengacakan dan enkripsi key/value environment saat compile-time via secure_dotenv generator
- Konfigurasi singleton EnvConfig untuk startup lifecycle dan flavor identification (Dev/UAT/Prod)
- Makefile automation untuk sinkronisasi .env flavor ke .env.target sebelum build runner dieksekusi

## 4. Code Examples

### Compile-time encrypted environment values generated from .env.target

```dart
@DotEnvGen(
  filename: ".env.target",
)
abstract class EnvValue {
  const EnvValue._();

  static EnvValue read() {
    env ??= const _$EnvValue();
    return env!;
  }

  @FieldKey(name: 'TIBqWL92V0', defaultValue: '')
  String get baseUrl;

  @FieldKey(name: 'TiB208Ah4j', defaultValue: '')
  String get baseUrlExternal;

  @FieldKey(name: 'rI3R6dbE2Y', defaultValue: '')
  String get messageCollection;
}
```

## 5. Considerations & Best Practices

- Selalu gitignore file .env.target dan plaintext .env.* yang berisi kredensial rahasia
- Eksekusi Makefile generate-build atau build-*-app untuk menyalin env flavor sebelum build runner

## 6. Related Knowledge

- Secure Hardware Backed Persistence With Keychain

## 7. Source

- Harvest 1787125853633 86cbed7a.json
