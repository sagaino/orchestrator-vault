---
title: API Service Data Flow
type: concept
tags: [concept, api, service-layer, axios]
created: 2026-08-12
updated: 2026-08-12
sources: ["[[03-Sources/documentation/AGENTS.md]]", "[[03-Sources/documentation/rules-api.md]]"]
---

# API Service Data Flow

Standar alur data komunikasi antara antarmuka (UI) dengan backend API.

## Alur Wajib (Mandatory Flow)

```
UI Component → Custom Hook → Service Layer → Axios Client → Backend API
```

## Aturan Komponen Alur
1. **UI Component**: Dilarang memanggil API/Axios secara langsung.
2. **Service Layer (`src/services/`)**:
   - Berisi fungsi eksekusi API yang bertipe data jelas (*strongly typed*).
   - Bebas dari dependensi framework/UI (tidak boleh mengandung JSX, toast, navigasi router, atau state UI).
3. **Central Endpoints (`src/lib/constant/endpoints.ts`)**:
   - Seluruh URL dan path API di-sentralisasi di sini. Dilarang menulis hardcoded URL string.
4. **Axios Client (`src/lib/axios.ts`)**:
   - Gerbang utama HTTP request, interceptor header, token auth, dan signature.
5. **Error Normalization (`src/lib/error-utils.ts`)**:
   - Semua error API diproses melalui helper ini untuk menampilkan pesan yang aman & ramah pengguna.

## Halaman Terkait
- [[01-Knowledge/concepts/architecture/frontend-architecture|Frontend Architecture Hub]]
- [[01-Knowledge/concepts/api/axios-client|Axios Client & Services]]
