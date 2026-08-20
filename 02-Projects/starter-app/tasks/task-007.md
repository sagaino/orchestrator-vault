---
title: "Task [007]: [change auth service]"
type: task
task_id: FE-007
project: starter-app
status: DONE
tags: [task, starter-app]
created: 2026-08-12
updated: 2026-08-14
assigned_to: Antigravity
dependencies: []
sources: []
---

# Task [007]: [change auth service]

## 🎯 Apa Yang Ingin Dikerjakan (Instruksi)
di `src/services/auth.ts` ganti style codenya menjadi ini : 
import axios from "@/lib/axios"

import { ENDPOINTS } from "@/lib/constant/endpoint"

import { unwrapResult } from "@/lib/error-utils"

import type { LoginFormData } from "@/pages/Login/types/request"

import type { LoginResponse, LoginResult } from "@/pages/Login/types/response"

  

export const AuthServices = {

login: async (data: LoginFormData): Promise<LoginResult> => {

const response = await axios.post<LoginResponse>(ENDPOINTS.LOGIN, {

bib: data.bibNumber,

phone: data.phoneNumber,

})

return unwrapResult(response.data)

},

logout: async (refreshToken: string): Promise<void> => {

const response = await axios.post(ENDPOINTS.LOGOUT, {

refresh_token: refreshToken,

})

if (response.data?.code && response.data.code !== 200) {

throw new Error(response.data?.message || "Gagal logout")

}

},

}

## 🏁 Hasil Yang Diharapkan (Expected Result)
sesuai dengan code yang saya berikan

---

## ❌ Error Log (Self-Healing)
*(Kosongkan bagian ini. Akan diisi otomatis jika terjadi error saat eksekusi).*

---

## 📝 Log Perubahan (Jurnal Eksekusi AI)
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]
- Mengubah `src/services/auth.ts` pada proyek `starter-app` (`/Users/sagaino/ciniru/starter-app`) sesuai dengan style code yang ditentukan (`AuthServices` object dengan method `login` dan `logout`).
- Dibuat/diperbarui berkas pendukung pendukung tipe dan konstanta:
  - `src/lib/constant/endpoint.ts` (konstanta `ENDPOINTS.LOGIN` dan `ENDPOINTS.LOGOUT`).
  - `src/lib/error-utils.ts` (penambahan helper `unwrapResult`).
  - `src/pages/Login/types/request.ts` (interface `LoginFormData`).
  - `src/pages/Login/types/response.ts` (interface `LoginResponse` dan `LoginResult`).
- Memperbarui `src/pages/Login/hooks/useLogin.ts` agar terintegrasi mulus menggunakan `AuthServices.login`.
- Memverifikasi kompilasi dan build dengan `rtk npx tsc --noEmit` (0 type error) dan `rtk npm run build` (build sukses).

