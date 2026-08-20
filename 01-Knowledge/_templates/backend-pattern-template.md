---
title: Backend Pattern Template
type: pattern
tags: [template, backend, architecture, api]
created: 2026-08-19
updated: 2026-08-19
sources: []
---

# {{Title}}

Deskripsi singkat tentang pola backend ini, masalah yang dipecahkan, dan kapan harus digunakan.

## 1. Overview & Architecture

Jelaskan konsep arsitektur, diagram alur, dan pemisahan lapisan tanggung jawab (misal: Handler -> Service -> Repository).

```text
[Client / API Consumer]
         │
         ▼
[Controller / HTTP Handler]  <-- Validasi Input & Response Formatting
         │
         ▼
[Service / Use Case Layer]   <-- Logika Bisnis & Transaksi
         │
         ▼
[Repository / Data Access]   <-- Query Database / ORM / Cache
         │
         ▼
    [Database]
```

## 2. Implementation & Code Structure

Berikan struktur direktori standar dan contoh implementasi kode inti.

## 3. Database & Data Access

- **Query / ORM**: Contoh schema atau query (Prisma, Gorm, Drizzle, SQL).
- **Transactions**: Cara menangani atomic transactions.

## 4. Error Handling & Validation

- Standar format error response.
- Validasi input (Zod, Joi, Validator, dsb.).

## 5. Security & Best Practices

- Auth / Autorisasi (JWT, OAuth2, RBAC, API Keys).
- Rate limiting, CORS, input sanitization.

## 6. Verification & Testing

- Cara menjalankan unit test & integration test untuk pola ini.

## 7. Related Knowledge

- `01-Knowledge/concepts/backend/...`
- `01-Knowledge/snippets/backend/...`
