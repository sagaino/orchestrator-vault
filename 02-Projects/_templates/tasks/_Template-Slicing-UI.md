---
title: "Task [NOMOR]: Slicing UI - [NAMA KOMPONEN/HALAMAN]"
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

# Task [NOMOR]: Slicing UI - [NAMA KOMPONEN/HALAMAN]

## 🎯 Tujuan Utama
Melakukan *slicing UI* (menerjemahkan desain visual menjadi kode fungsional) untuk komponen/halaman **[Nama Komponen]** di proyek **[nama-proyek-anda]**. Kode harus dibuat modern, responsif, dan rapi menggunakan **React (TypeScript), Tailwind CSS, dan Shadcn UI**.

## 🖼️ Referensi Visual (Lampiran Desain)
*(Untuk Pengguna: Tempel/Copy-Paste gambar desain Anda ke folder Obsidian, lalu ubah path file di bawah ini mengarah ke file gambar fisik Anda).*

Optional reference: `03-Sources/assets/<mockup-file>`
*(Path absolut gambar untuk AI: `/Users/sagaino/Documents/Obsidian Vault/03-Sources/assets/mockup-anda.png`)*

**Instruksi Khusus untuk Coding Agent:**
1. Wajib **membaca dan menganalisis** file gambar pada path absolut di atas menggunakan alat pembaca file (`view_file`) sebelum menulis kode.
2. Perhatikan dengan detail warna, tata letak, margin/padding (spasi), tipografi, dan hierarki visual dari gambar tersebut.

## 📋 Spesifikasi & Langkah Pengerjaan
1. **Buat File**: Buat file komponen baru di direktori `src/components/ui/custom/` atau `src/pages/[NamaFitur]/components/`.
2. **Instalasi Shadcn (Jika Perlu)**: Jika desain membutuhkan komponen yang belum terinstal di proyek (seperti *accordion*, *dialog*, atau *carousel*), jalankan perintah `npx -y shadcn@latest add [nama-komponen] -y` terlebih dahulu.
3. **Penulisan Kode**: Terjemahkan gambar tersebut ke dalam susunan kelas utilitas Tailwind CSS. Usahakan *pixel-perfect* semirip mungkin dengan referensi gambar.

## Acceptance Criteria
*(Tuliskan viewport, state, interaksi, dan detail visual konkret yang harus cocok dengan referensi, serta verification command yang harus lulus).*

## ❌ Error Log (Self-Healing)
*(Kosongkan bagian ini. Akan diisi otomatis oleh orchestrator jika execution atau verification gagal).*

---
## 📝 Log Perubahan (Jurnal Eksekusi AI)
*(Akan diisi otomatis oleh orchestrator setelah tugas selesai dieksekusi)*
