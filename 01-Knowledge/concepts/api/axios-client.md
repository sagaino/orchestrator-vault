---
title: Axios Client & Services
type: concept
tags: [entity, axios, services, http]
created: 2026-08-12
updated: 2026-08-14
sources: ["[[03-Sources/documentation/rules-api.md]]"]
---

# Axios Client & Services

Instansi HTTP client tunggal dan lapisan servis aplikasi.

## Aturan Penggunaan
- **Single Gateway (`src/lib/axios.ts`)**: Seluruh request HTTP wajib melewati instansi Axios ini (mengelola interceptor, token, dan signature).
- **No Direct `localStorage`**: Dilarang mengambil token dari `localStorage` secara manual dalam service/komponen. Gunakan helper terpusat `useLocalStorage`.
- **No Manual Bearer Header**: Header otentikasi ditangani secara otomatis oleh interceptor Axios.
- **Service Isolation (`src/services/`)**: Fungsi service hanya melakukan panggilan API bertipe data statis tanpa kode JSX atau efek samping UI.

## Halaman Terkait
- [[01-Knowledge/concepts/architecture/frontend-architecture|Frontend Architecture Hub]]
- [[01-Knowledge/concepts/architecture/api-service-data-flow|API Service Data Flow]]
