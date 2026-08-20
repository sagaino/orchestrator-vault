---
title: "Task [006]: [Change type input in login]"
type: task
task_id: FE-006
project: starter-app
status: DONE
tags: [task, starter-app]
created: 2026-08-12
updated: 2026-08-14
assigned_to: Antigravity
dependencies: []
sources: []
---

# Task [006]: [Change type input in login]

## 🎯 Apa Yang Ingin Dikerjakan (Instruksi)
di `src/pages/Login/components/LoginForm.tsx` ada 2 input saya ingin mengganti input typenya. input BIB typenya text dan input phone number typenya text dengan inputmode numeric, pattern="[0-9+]"  dan onChange={(event) => {

const raw = event.target.value

const hasLeadingPlus = raw.startsWith('+')

const digitsOnly = raw.replace(/\D/g, '')

field.onChange(

hasLeadingPlus ? `+${digitsOnly}` : digitsOnly

)

}}

## 🏁 Hasil Yang Diharapkan (Expected Result)
user input bib dengan text dan input phone number hanya bisa angka saja

---

## ❌ Error Log (Self-Healing)
*(Kosongkan bagian ini. Akan diisi otomatis jika terjadi error saat eksekusi).*

---

## 📝 Log Perubahan (Jurnal Eksekusi AI)
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]
- Mengganti penanganan input phone number di `src/pages/Login/components/LoginForm.tsx` menggunakan `Controller` dari `react-hook-form`.
- Menambahkan atribut `inputMode="numeric"`, `pattern="[0-9+]"`, dan handler `onChange` yang memfilter input sehingga hanya mengizinkan tanda `+` di posisi paling awal dan angka.
- Memastikan input BIB Number tetap bertipe `text`.
- Menjalankan `npm run typecheck` untuk verifikasi TypeScript tanpa error.

