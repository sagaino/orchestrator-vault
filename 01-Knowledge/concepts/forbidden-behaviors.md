---
title: Forbidden Behaviors
type: concept
tags: [concept, rules, forbidden, constraints]
created: 2026-08-12
updated: 2026-08-12
sources: ["[[03-Sources/documentation/AGENTS.md]]"]
---

# Forbidden Behaviors (Hal yang Dilarang Keras)

Daftar aturan dan larangan eksplisit bagi pengembangan proyek:

1. **Dilarang Menulis Ulang Berkas Secara Total**: Gunakan localized edit yang terfokus saat hanya sedikit perubahan yang dibutuhkan.
2. **Dilarang Refactoring Kode yang Tidak Relevan**: Jangan mengubah komponen atau modul lain yang tidak sedang dikerjakan.
3. **Dilarang Menambah Dependency NPM Baru**: Tanpa persetujuan dari pengguna.
4. **Dilarang Membuat-Buat Kontrak API**: Jangan mengartifikasikan request payload, model respon, atau aturan bisnis tanpa kepastian spec.
5. **Dilarang Melewati Service Layer**: UI tidak boleh memanggil Axios atau fetch API langsung.
6. **Dilarang Akses `localStorage` Langsung untuk Auth**: Gunakan `useLocalStorage` terpusat.
7. **Dilarang Hardcode String UI**: Selalu gunakan `i18next` melalui `useTranslation()`.
8. **Dilarang Menyisakan Kode Debug**: Hapus `console.log`, `TODO`, atau `FIXME` sebelum menyelesaikan tugas.
9. **Dilarang Mengubah Nama Export Tanpa Sebab**: Jangan mengubah nama API, komponen, atau berkas tanpa alasan mendesak.

## Halaman Terkait
- [[01-Knowledge/concepts/architecture/frontend-architecture|Frontend Architecture Hub]]
- [[01-Knowledge/concepts/definition-of-done|Definition of Done (DoD)]]
