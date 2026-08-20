---
title: "Task Template: Inisialisasi Proyek FE Baru + Modul Contoh Login"
type: task-template
task_id: FE-XXX
project: <project-id>
status: BACKLOG
tags: [task-template]
created: 2026-08-14
updated: 2026-08-14
dependencies: []
verification: [typecheck, build]
allowed_paths: []
requires_changes: true
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
- Setelah verification, biarkan orchestrator menjalankan refresh Graphify pada repository `<PROJECT_NAME>`.
- Simpan hanya metadata repository dan pointer Graphify di `02-Projects/<PROJECT_NAME>/project.md`; jangan menyalin `graph.json` ke Obsidian.

---
## 📝 Log Perubahan
*(Akan diisi otomatis oleh orchestrator setelah tugas dieksekusi)*
