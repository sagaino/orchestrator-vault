---
title: "Task 011: change useStrorage code"
type: task
task_id: FE-011
project: starter-app
status: DONE
tags: [task, starter-app]
created: 2026-08-13
updated: 2026-08-14
assigned_to: Antigravity
dependencies: []
sources: []
---

# Task 011: change useStrorage code

## 🎯 Apa Yang Ingin Dikerjakan (Instruksi)
ganti code yang ada di useStorage dengan code yang ada di 01-Knowledge/patterns/frontend/project-skeleton-template, di sana ada aturan useStorage baru. dan implementasi ke existing code yang menggunakannya

## 🏁 Hasil Yang Diharapkan
useStorage sesuai dengan yang ada di project-skeleton-template dan bisa di gunakan di exisiting code

---
## ❌ Error Log (Self-Healing)


---
## 📝 Log Perubahan (Jurnal Eksekusi AI)
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]
- **[2026-08-13]** Mengubah implementasi `src/hooks/useLocalStorage.ts` di proyek `starter-app` sesuai dengan aturan template cetak biru di `01-Knowledge/patterns/frontend/project-skeleton-template.md` (menggunakan enkripsi AES via `crypto-js` dan sinkronisasi event `storage`).
- Meng-update `src/lib/axios.ts` untuk menggunakan helper `removeData` dari `@/hooks/useLocalStorage` saat interceptor mendeteksi HTTP status 401.
- Menambahkan `VITE_SECRET_KEY` pada file `.env` di `starter-app`.
- Menjalankan verifikasi tipe `npm run typecheck` dan build `npm run build` — seluruh tes berhasil tanpa error.
- Meng-update state topologi proyek dengan `graphify update .` dan me-refresh `02-Projects/starter-app/graph-state.md`.

