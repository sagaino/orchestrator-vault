---
title: "Wiring API Login"
type: task
task_id: FE-AUTH-LOGIN
project: starter-app
status: DONE
tags: [task, starter-app]
created: 2026-08-13
updated: 2026-08-14
completed_at: 2026-08-13
assigned_to: Antigravity
dependencies: []
sources: ["[[03-Sources/documentation/api-docs.md]]"]
---

# Wiring API Login

**Deskripsi Tugas:**
Mengintegrasikan antarmuka (UI) halaman Login dengan endpoint backend `POST /api/v1/auth/login` berdasarkan dokumentasi API.

## Referensi API

- **Endpoint:** `POST /api/v1/auth/login`
- **Base URL:** `http://172.19.253.36:8001`
- **Request Body:**
  ```json
  {
    "bib": "string",
    "phone": "string"
  }
  ```
- **Response Sukses:** Menyimpan `access_token` dan `refresh_token` ke penyimpanan lokal yang aman (mis. localStorage / HttpOnly cookies).

## Checklist Pekerjaan (Implementation Plan)

- [x] **1. Setup Service API Auth:**
  - Buat fungsi/service `loginRequest(bib, phone)` di layer HTTP client.
  - Tambahkan konfigurasi tipe data `LoginRequest` dan `LoginResponse` (berisi access_token, refresh_token, dll).
- [x] **2. State Management:**
  - Buat action/mutasi untuk memanggil `loginRequest` dan mengelola state _loading_, _error_, dan _success_.
- [x] **3. Wiring ke UI (Login Component):**
  - Hubungkan form login (input `bib` dan `phone`) ke fungsi state login.
  - Tampilkan pesan validasi atau notifikasi error (contoh: kredensial salah) dari API.
- [x] **4. Manajemen Token & Navigasi:**
  - Saat `200 OK`, simpan token dari response.
  - Redirect pengguna ke halaman utama (Dashboard / Beranda).
- [x] **5. Testing:**
  - Lakukan *manual testing* untuk skenario sukses (login berhasil).
  - Lakukan *manual testing* untuk skenario gagal (input kosong, credential salah).

## ❌ Error Log (Self-Healing)
*(Kosong - tidak ada error selama eksekusi).*

---
## 📝 Log Perubahan (Jurnal Eksekusi AI)
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

- **`src/services/auth.ts`**: Menambahkan fungsi `loginRequest(bib, phone)` dan tipe data `LoginRequest`, `LoginResponse`, serta `LoginResponseData`. Menghapus dummy fallback response sesuai standar API wiring.
- **`src/pages/Login/hooks/useLogin.ts`**: Memperbarui custom hook `useLogin` untuk mengeksekusi `loginRequest` dengan payload `{ bib, phone }` bertipe `LoginRequest`.
- **`src/pages/Login/types/index.ts`**: Memperbarui schema Zod `loginSchema` menggunakan properti `bib` dan `phone`.
- **`src/pages/Login/components/LoginForm.tsx`**: Menghubungkan input formulir `bib` dan `phone` ke mutasi `login`, menyimpan `access_token` dan `refresh_token` ke local storage (`LOCALSTORAGE_KEY.TOKEN` dan `LOCALSTORAGE_KEY.REFRESH_TOKEN`), serta menampilkan notifikasi error API menggunakan `getErrorMessage()`.
- **`src/lib/axios.ts`**: Memperbarui `baseURL` default ke `http://172.19.253.36:8001` dan menangani error 401 pada endpoint login tanpa redirect loop.
- **`.env`**: Membuat file konfigurasi lingkungan dengan `VITE_API_URL=http://172.19.253.36:8001`.
- **Verifikasi Build**: Menjalankan `npm run build` dengan hasil 0 type errors & kompilasi sukses.

