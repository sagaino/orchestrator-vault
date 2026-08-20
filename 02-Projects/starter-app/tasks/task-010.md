---
title: "Task 010: Buat routed global"
type: task
task_id: FE-010
project: starter-app
status: DONE
tags: [task, starter-app]
created: 2026-08-13
updated: 2026-08-14
assigned_to: Antigravity
dependencies: []
sources: []
---

# Task 010: Buat routed global 

## 🎯 Apa Yang Ingin Dikerjakan (Instruksi)
Buatkan file routes.ts di dalam folder constant yang berguna untuk reuse path routes, contoh seperti ini : 
export const ROUTES = {

LOGIN: "/login",

GALLERY: "/gallery",

}

## 🏁 Hasil Yang Diharapkan
Kode `src/lib/constant/routes.ts` berhasil di buat dan di gunakan ke tempat yang membutuhkan

---
## ❌ Error Log (Self-Healing)


---
## 📝 Log Perubahan (Jurnal Eksekusi AI)
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]
- Melakukan pemeriksaan pre-edit pada `02-Projects/starter-app/graph-state.md`.
- Membuat file `src/lib/constant/routes.ts` dengan mendefinisikan objek konstanta `ROUTES` (`LOGIN`, `DASHBOARD`, `GALLERY`).
- Mengganti seluruh string path route hardcoded dengan `ROUTES` pada `src/routes/index.tsx`, `src/routes/PrivateRoute.tsx`, `src/routes/PublicRoute.tsx`, `src/pages/Login/hooks/useLogin.ts`, `src/pages/Dashboard/hooks/useDashboard.ts`, dan `src/lib/axios.ts`.
- Memverifikasi kompilasi TypeScript dengan `rtk npx tsc --noEmit` (0 errors).
- Melakukan pengkinian topologi proyek dengan `graphify update .` dan memperbarui `02-Projects/starter-app/graph-state.md`.

