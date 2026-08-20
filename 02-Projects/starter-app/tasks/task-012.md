---
title: "Task 012: wiring logout api"
type: task
task_id: FE-012
project: starter-app
status: DONE
tags: [task, starter-app]
created: 2026-08-13
updated: 2026-08-14
assigned_to: Antigravity
dependencies: []
sources: []
---

# Task 012: wiring logout api

## 🎯 Apa Yang Ingin Dikerjakan (Instruksi)
untuk service auth.logout sudah di buat. tolong di gunakan untuk wiring sistem logout, buatkan saja global hooks untuk logout seperti useLogout dan gunakan di dashboardnya

## 🏁 Hasil Yang Diharapkan
ketika logout akan hit api logout dulu, jika berhasil akan clear localstorage. ketika gagal hit api jangan auto logout tapi tampilkan toast gagal logout gitu

---
## ❌ Error Log (Self-Healing)


---
## 📝 Log Perubahan (Jurnal Eksekusi AI)
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]
- Membaca dan menganalisis graph-state & arsitektur proyek `starter-app`.
- Membuat global hook `useLogout` (`src/hooks/useLogout.ts`) yang memanggil `AuthServices.logout` menggunakan TanStack Query `useMutation`.
- Pada `onSuccess`: Menghapus data token dari `localStorage` (`LOCALSTORAGE_KEY.TOKEN` & `LOCALSTORAGE_KEY.REFRESH_TOKEN`) dan melakukan navigasi ke halaman login (`ROUTES.LOGIN`).
- Pada `onError`: Tidak melakukan auto logout / pembersihan `localStorage`, melainkan menampilkan toast error notification ("Gagal Logout").
- Memasang komponen `<Toaster />` pada `src/App.tsx` agar notifikasi toast dapat muncul secara global.
- Memperbarui `useDashboard` (`src/pages/Dashboard/hooks/useDashboard.ts`) dan `DashboardPage` (`src/pages/Dashboard/index.tsx`) untuk menggunakan `useLogout`.
- Memverifikasi build TypeScript/Vite secara sukses (`npm run build`).
- Memperbarui peta topologi proyek dengan `graphify update .` dan memperbarui `/Users/sagaino/Documents/Obsidian Vault/02-Projects/starter-app/graph-state.md`.

