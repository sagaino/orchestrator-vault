---
title: "Task Template: Inisialisasi Proyek FE Baru + Modul Contoh Login"
type: task
task_id: FE-INIT
project: starter-app
status: DONE
tags: [task, starter-app]
created: 2026-08-12
updated: 2026-08-14
completed_at: 2026-08-12
assigned_to: Antigravity
dependencies: []
sources: []
---

# Task Template: Inisialisasi Proyek FE Baru + Modul Contoh Login

## 🎯 Deskripsi Utama
Tolong buat dan inisialisasi proyek Frontend baru di dalam folder `ciniru` menggunakan variabel nama proyek yang didefinisikan pada properti **`project:`** di YAML frontmatter paling atas.

*(Catatan untuk AI Agent: Gunakan nilai `project:` sebagai nama proyek `<PROJECT_NAME>`).*

## 📋 Spesifikasi & Langkah Eksekusi

### 1. Inisialisasi Proyek & Install Komponen Shadcn UI
Jalankan perintah terminal dari folder `/Users/sagaino/ciniru` menggunakan variabel nama proyek:
- **Langkah 1: Inisialisasi Vite + Base UI + Preset Nova**:
  ```bash
  npx -y shadcn@latest init -t vite -b base -n <PROJECT_NAME> -p nova --no-monorepo -y
  ```
- **Langkah 2: Install Seluruh Komponen UI Shadcn**:
  Masuk ke folder `/Users/sagaino/ciniru/<PROJECT_NAME>` lalu jalankan:
  ```bash
  npx -y shadcn@latest add --all -y
  ```

### 2. Penyesuaian Struktur Direktori & Modul Contoh Login
Susun struktur folder `src/` pada proyek `<PROJECT_NAME>` mengikuti cetak biru `01-Knowledge/patterns/frontend/project-skeleton-template.md`:
- `src/lib/constant/endpoints.ts`: Menyimpan URL API auth `/api/v1/auth/login`.
- `src/lib/axios.ts` & `signature.ts`: Terpasang interceptor token, signature, & 401 redirect.
- `src/services/auth.ts`: Service login dengan dummy fallback response data jika API backend offline.
- `src/pages/Login/`: 
  - `types/index.ts` (Zod schema & TypeScript interface)
  - `hooks/useLogin.ts` (React Hook Form + TanStack Query `useMutation`)
  - `components/LoginForm.tsx` (UI Form Login menggunakan Shadcn Card, Input, Button, Toast)
  - `index.tsx` (Halaman utama Login)
- `src/routes/`: Terpasang pengaman rute `PublicRoute` & `PrivateRoute`.

### 3. Pemetaan Graphify Pertama
- Jalankan pemindaian `graphify` pada proyek `<PROJECT_NAME>`.
- Buat file peta topologi di `02-Projects/<PROJECT_NAME>/project.md` pada Obsidian Vault ini.

---
## 📝 Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]
- **2026-08-12**: Inisialisasi proyek FE `starter-app` di `/Users/sagaino/ciniru/starter-app` berhasil diselesaikan:
  1. Dibuat dengan Vite + Base UI + Preset Nova (`npx -y shadcn@latest init -t vite -b base -n starter-app -p nova --no-monorepo -y`).
  2. Dipasang seluruh 61 komponen Shadcn UI (`npx -y shadcn@latest add --all -y`).
  3. Dibuat modul skeleton lengkap: `endpoints.ts`, `localstorage.ts`, `useLocalStorage.ts`, `signature.ts` (CryptoJS SHA1), `axios.ts` (with request & 401 interceptors), `error-utils.ts`, `services/auth.ts`, `pages/Login` (Zod schema, `useLogin` TanStack Query mutation, `LoginForm` component), `pages/Dashboard`, dan `routes` (`PublicRoute`, `PrivateRoute`, React Router).
  4. Pengujian build `npm run build` sukses 100% tanpa error TypeScript/compilation.
  5. Pemindaian topologi `graphify` berhasil dijalankan (695 nodes, 1236 edges) dan hasilnya didokumentasikan di `02-Projects/starter-app/graph-state.md`.
