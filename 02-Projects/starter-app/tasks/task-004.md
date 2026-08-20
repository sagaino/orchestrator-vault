---
title: "Task 004: Slicing UI - Halaman Login"
type: task
task_id: FE-004
project: starter-app
status: DONE
tags: [task, starter-app]
created: 2026-08-12
updated: 2026-08-14
completed_at: 2026-08-12T16:18:45+07:00
assigned_to: Antigravity
dependencies: []
sources: []
---

# Task 004: Slicing UI - Halaman Login

## 🎯 Tujuan Utama
Melakukan *slicing UI* (menerjemahkan desain visual menjadi kode fungsional) untuk **Halaman Login** di proyek **starter-app**. Kode harus dibuat modern, responsif, dan rapi menggunakan **React (TypeScript), Tailwind CSS, dan Shadcn UI**.

## 🖼️ Referensi Visual (Lampiran Desain & Aset)
Berikut adalah referensi visual UI yang harus di-*slicing*:
!Login Ui.png
*(Path absolut gambar untuk AI: `/Users/sagaino/Documents/Obsidian Vault/03-Sources/assets/login-ui.png`)*

Gunakan gambar berikut ini sebagai *background* (latar belakang) halaman penuh (*full screen*):
!Bg_img.webp
*(Path absolut gambar untuk AI: `/Users/sagaino/Documents/Obsidian Vault/03-Sources/assets/bg_img.webp`)*

Gunakan gambar berikut ini sebagai Logo di dalam Card Login:
!Langkah_rasa_img.png
*(Path absolut gambar untuk AI: `/Users/sagaino/Documents/Obsidian Vault/03-Sources/assets/langkah_rasa_img.png`)*

**Instruksi Khusus untuk AI Agen (Antigravity):**
1. Wajib **membaca dan menganalisis** file `login-ui.png` pada path absolut di atas menggunakan alat pembaca file (`view_file`) sebelum menulis kode.
2. Gunakan `bg_img.webp` sebagai gambar latar belakang halaman. Pastikan path gambar ini diintegrasikan dengan benar ke dalam kode React/Vite.
3. Perhatikan dengan detail warna, tata letak, margin/padding (spasi), tipografi, dan hierarki visual dari gambar `login-ui.png`.
4. **Spesifikasi Elemen UI (Figma Export)**: Jadikan kode CSS di bawah ini sebagai acuan wajib agar ukuran *pixel-perfect*:

   **Card Container:**
   ```css
   width: 586px;
   height: 628px;
   border-radius: 16px;
   background: #FBF8F0;
   box-shadow: 
     0px 0.54px 3.49px 0px #35405C0E,
     0px 1.5px 9.64px 0px #35405C14,
     0px 3.62px 23.21px 0px #35405C1B,
     0px 12px 77px 0px #35405C29;
   ```
   
   **Logo (`langkah_rasa_img.png`):**
   ```css
   width: 182px;
   height: 129px;
   top: 68px;
   left: 202px;
   ```

   **Input Field:**
   ```css
   width: 450px;
   height: 48px;
   border-radius: 58px;
   border-width: 1px;
   padding-top: 12px;
   padding-right: 20px;
   padding-bottom: 12px;
   padding-left: 20px;
   gap: 7px;
   background: #D1D1D6; /* var(--Grays-Gray-4) */
   font-family: Poppins;
   font-weight: 500;
   font-size: 16px;
   line-height: 27px;
   ```

   **Submit Button:**
   ```css
   width: 450px;
   height: 56px;
   border-radius: 50px;
   background: #FFD400; /* Gunakan variabel warna global 'primary' */
   color: #2A2318;
   padding-top: 16px;
   padding-right: 28px;
   padding-bottom: 16px;
   padding-left: 28px;
   gap: 10px;
   font-family: Poppins;
   font-weight: 700;
   font-size: 14px;
   line-height: 24px;
   text-align: center;
   ```

## 📋 Spesifikasi & Langkah Pengerjaan
1. **Buat File/Folder**: Perbarui komponen/halaman Login di direktori proyek `starter-app` (misalnya di `src/pages/Login/`).
2. **Integrasi Aset Image**: Salin/Copy file `bg_img.webp` dari folder Obsidian ke folder web publik Vite di dalam proyek `starter-app` (contoh: salin ke `public/` atau `src/assets/`) agar bisa di-load oleh tag `img` atau CSS *background*.
3. **Instalasi Shadcn (Jika Perlu)**: Jika desain membutuhkan komponen yang belum terinstal (seperti `Card`, `Input`, `Button`, dll), jalankan perintah `npx -y shadcn@latest add [nama-komponen] -y` terlebih dahulu.
4. **Konfigurasi Global & Penulisan Kode**: 
   - Konfigurasikan warna-warna utama di `tailwind.config.ts` agar dapat di-reuse ke depannya (contoh: `colors: { primary: { DEFAULT: '#FFD400' } }`).
   - Integrasikan *font* `Poppins` ke dalam sistem proyek.
   - Terjemahkan gambar referensi dan spesifikasi CSS di atas ke dalam susunan kelas utilitas Tailwind CSS yang *pixel-perfect*.
5. **Sinkronisasi Ingatan (Graph-State)**: Setelah tugas koding selesai, Anda (AI) **WAJIB** membaca dan memperbarui file `02-Projects/<PROJECT_NAME>/project.md` (buat filenya jika belum ada). Daftarkan komponen `Login` baru ini ke dalam dokumen tersebut agar arsitektur proyek tetap *up-to-date* di dalam *Primary Brain* Obsidian.

## ❌ Error Log (Self-Healing)
*(Tidak ada error saat eksekusi).*

---
## 📝 Log Perubahan (Jurnal Eksekusi AI)
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]
- **Visual Inspection**: Membaca dan menganalisis `login-ui.png`, `bg_img.webp`, dan `langkah_rasa_img.png` menggunakan tool `view_file`.
- **Copy Assets**: Menyalin file `bg_img.webp` dan `langkah_rasa_img.png` ke folder `src/assets/` dan `public/` di proyek `starter-app`.
- **Font & Theme Setup**: Menginstal `@fontsource/poppins`, memperbarui `src/index.css` dengan import font Poppins dan variabel warna utama (`--primary: #FFD400`, `--primary-foreground: #2A2318`). Membikin `tailwind.config.ts` dengan ekstensi tema Poppins, warna primary, dan custom box-shadow.
- **UI Slicing & Code Refactoring**:
  - `src/pages/Login/components/LoginForm.tsx`: Menyusun card login berukuran pixel-perfect `586px x 628px`, rounded 16px, background `#FBF8F0`, box shadow multi-layered, logo 182px x 129px, input rounded pill `450px x 48px`, serta button `SEARCH` berwarna `#FFD400` dengan font Poppins bold.
  - `src/pages/Login/index.tsx`: Mengintegrasikan `bg_img.webp` sebagai background full-screen responsif.
  - `src/pages/Login/types/index.ts` & `src/services/auth.ts`: Memperbarui schema validasi Zod dan service auth untuk field `bibNumber` & `phoneNumber`.
- **Verification**: Menjalankan `npm run typecheck` dan `npm run build` dengan hasil 100% sukses tanpa error.
