---
title: "Task [NOMOR]: Refactor - [NAMA FUNGSI / FITUR]"
type: task-template
task_id: FE-XXX
project: <project-id>
status: BACKLOG
tags: [task-template]
created: 2026-08-14
updated: 2026-08-14
dependencies: []
verification: []
allowed_paths: []
requires_changes: true
sources: []
---

# Task [NOMOR]: Refactor - [NAMA FUNGSI / FITUR]

## 🎯 Tujuan Utama
Mengubah/memperbaiki implementasi fungsi **`[NamaFungsi]`** pada berkas `[path/ke/file.ts]` agar sesuai dengan spesifikasi, logika bisnis, dan standar yang diinginkan pengguna.

## 📌 Context & Kode Saat Ini (Current State)
- **File Target:** `src/path/to/target-file.ts`
- **Fungsi / Komponen Target:** `[namaFungsiAtauKomponen]`
- **Masalah pada Kode Sekarang:**
  *(Jelaskan kenapa kode saat ini belum sesuai. Contoh: Urutan parameter salah, belum menggunakan enkripsi, atau tidak memproses exception dengan benar).*

---

## 💡 Ekspektasi & Instruksi Yang Diinginkan (Desired Implementation)
*(Jelaskan secara detail bagaimana fungsi ini seharusnya bekerja, atau sertakan contoh pola/pseudocode).*

1. **Aturan Logika / Spesifikasi:**
   - **Aturan 1:** *(Contoh: Parameter fungsi wajib menggunakan objek `{ email, password }` bukan argumen terpisah).*
   - **Aturan 2:** *(Contoh: Wajib memanggil helper `setData` dari `useLocalStorage` terenkripsi).*
   - **Aturan 3:** *(Contoh: Harus mengembalikan response ter-normalize).*

2. **Contoh Kode / Pseudocode Acuan:**
   ```typescript
   // Tuliskan contoh struktur/pola kode yang Anda harapkan di sini:
   export const myDesiredFunction = async (payload: DesiredPayload) => {
     // Logika yang Anda inginkan...
   };
   ```

---

## 📋 Spesifikasi & Langkah Pengerjaan
- [ ] **1. Review Kode Eksisting:**
  - Buka file target dan lacak seluruh tempat di mana fungsi ini dipanggil (*call sites*).
- [ ] **2. Refactor Implementasi Fungsi:**
  - Ubah fungsi target sesuai dengan spesifikasi dan pseudocode yang diminta.
  - Perbarui tipe data TypeScript (interfaces/types) jika ada perubahan parameter atau return value.
- [ ] **3. Update Pemanggilan (Call Sites):**
  - Perbarui seluruh file/komponen yang memanggil fungsi ini agar cocok dengan signature baru.
- [ ] **4. Verifikasi & Definition of Done:**
  - [ ] Jalankan `npx tsc --noEmit` untuk memastikan 0 error tipe data.
  - [ ] Uji fungsionalitas fungsi yang baru di-refactor.

## Acceptance Criteria
*(Tuliskan kondisi konkret untuk memastikan behavior tetap benar, call site sudah diperbarui, dan verification lulus).*

---

## ❌ Error Log (Self-Healing)
*(Kosongkan bagian ini. Akan diisi otomatis jika terjadi error saat AI mengeksekusi perbaikan).*

---

## 📝 Log Perubahan (Jurnal Eksekusi AI)
*(Bagian di bawah ini jangan diisi manual. Akan diisi otomatis oleh orchestrator setelah tugas selesai dieksekusi, beserta watermark verifikasi).*
