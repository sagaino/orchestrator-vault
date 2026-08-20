---
title: React useLocalStorage
type: snippet
tags: [snippet, react, local-storage, persistence, crypto-js, state-management]
created: 2026-08-14
updated: 2026-08-14
classification: NEW
sources: ["UseLocalStorage.ts"]
---

# React useLocalStorage

## Overview

`useLocalStorage` adalah custom React hook untuk menyimpan state secara persisten di browser `localStorage`. Hook ini mendukung dua mode penyimpanan: JSON biasa dan JSON yang dienkripsi menggunakan AES melalui `crypto-js`.

Source ini menggabungkan state React dengan helper imperatif `setData`, `getData`, dan `removeData`, serta memakai event `storage` untuk menyinkronkan perubahan antar pemakai hook dalam dokumen yang sama.

## Purpose

- Mempertahankan state setelah reload halaman.
- Menyediakan API setter yang mirip `useState`, termasuk functional updater.
- Memberi opsi penyimpanan terenkripsi untuk data yang tidak ingin terlihat langsung di `localStorage`.
- Menyatukan akses langsung dan akses berbasis hook melalui helper yang sama.

## API

### `useLocalStorage<T>(key, initialValue, options)`

Mengembalikan tuple `[storedValue, setValue]`.

- `key`: nama key di `localStorage`.
- `initialValue`: nilai fallback saat key belum ada atau tidak dapat dibaca.
- `options.encrypted`: jika `true`, gunakan AES; default `false`.
- `setValue`: menerima nilai langsung atau function `(currentValue) => nextValue`.

### `setData(storageKey, value)`

Mengubah value menjadi JSON, mengenkripsinya dengan AES, lalu menyimpannya ke `localStorage`. Helper ini selalu menggunakan mode terenkripsi.

### `getData(storageKey)`

Membaca value terenkripsi, mendekripsinya, lalu melakukan `JSON.parse`. Jika data kosong, rusak, gagal didekripsi, atau gagal diparse, key dihapus dan helper mengembalikan `null`.

### `removeData(storageKey)`

Menghapus key dari `localStorage` dan memancarkan event `storage` agar consumer dapat menyinkronkan state.

## Implementation

1. Initializer `useState` memeriksa apakah kode berjalan di browser.
2. Mode plain membaca dan menulis JSON langsung ke `localStorage`.
3. Mode encrypted mendelegasikan baca/tulis ke `getData` dan `setData`.
4. `useEffect` mendaftarkan listener `storage` untuk membaca ulang value ketika ada perubahan.
5. `setValue` memperbarui state React dan storage.
6. `SECRET_KEY` diambil dari `VITE_SECRET_KEY`, dengan fallback `starter-app-secret-key`.

## Usage

```tsx
const [filters, setFilters] = useLocalStorage<ProductFilters>(
  "product-filters",
  { category: "all" },
  { encrypted: true },
);

setFilters((current) => ({ ...current, category: "books" }));
```

Untuk akses langsung terhadap data yang memakai helper terenkripsi:

```ts
setData(LOCALSTORAGE_KEY.TOKEN, token);
const token = getData(LOCALSTORAGE_KEY.TOKEN);
removeData(LOCALSTORAGE_KEY.TOKEN);
```

## Behavior

- Jika key tersedia dan valid, nilai dari storage menjadi nilai awal hook.
- Jika key tidak tersedia, hook memakai `initialValue` dan mencoba menyimpannya.
- Mode plain menyimpan JSON yang dapat dibaca langsung.
- Mode encrypted menyimpan ciphertext AES.
- Data yang gagal didekripsi atau diparse dihapus sebagai mekanisme self-healing.
- Setter mendukung nilai langsung maupun functional updater.
- Custom event `storage` membuat perubahan dari setter/helper dapat memicu pembacaan ulang pada hook yang sedang terpasang.

## Considerations

- `setData` dan `getData` menggunakan `any`, sehingga type safety melemah. Untuk implementasi production, pertimbangkan generic atau wrapper bertipe `unknown` untuk setiap key.
- `VITE_SECRET_KEY` berada di bundle frontend. Siapa pun yang dapat membaca bundle berpotensi menemukan key tersebut; AES di sini adalah obfuscation/client-side confidentiality, bukan pengganti secret server atau cookie `HttpOnly` dan `Secure`.
- `localStorage` tetap sensitif terhadap XSS. Jangan menyimpan secret bernilai tinggi atau token berumur panjang tanpa threat model dan mitigasi yang jelas.
- `window.dispatchEvent(new Event("storage"))` adalah mekanisme custom untuk sinkronisasi same-tab. Event `storage` native browser biasanya dipicu pada dokumen atau tab lain, bukan dokumen yang melakukan perubahan.
- Listener tidak memfilter `storageKey`, sehingga setiap custom event dapat membuat semua instance hook membaca ulang key masing-masing.
- Initializer memakai `if (initialValue)`, sehingga nilai awal falsy seperti `false`, `0`, `""`, atau `null` tidak dipersist. Jika nilai falsy harus disimpan, gunakan pemeriksaan `initialValue !== undefined`.
- `getData` mengembalikan `any` atau `null` dan menghapus data saat terjadi error. Ini membantu pemulihan otomatis, tetapi dapat menyamarkan kebutuhan migrasi atau menyebabkan data lokal hilang.
- Saat key dihapus, listener hanya memperbarui state jika `newValue !== null`. Akibatnya state hook dapat tetap menyimpan nilai lama setelah `removeData`; implementasi yang diinginkan biasanya mengembalikannya ke `initialValue`.
- Guard SSR tersedia pada initializer, tetapi effect tetap mengakses `window`. Pada aplikasi SSR/hydration, guard juga perlu dipasang di effect bila environment mengharuskannya.
- Catch pada `removeData` memakai `console.log`, bukan mekanisme observability aplikasi. Source asli dipertahankan; catatan ini menjadi pertimbangan saat refactor.

## Classification

- Classification: `NEW`
- Scope: Reusable
- Promotion decision: `PROMOTED_TO_GLOBAL_WIKI`

## Related Knowledge

- [[01-Knowledge/concepts/react/react]]
- [[01-Knowledge/concepts/state-management/tanstack-query]]
- [[01-Knowledge/concepts/architecture/state-logic-separation]]
- [[01-Knowledge/patterns/frontend/project-skeleton-template]]
- [[01-Knowledge/concepts/forbidden-behaviors]]

## Source

- UseLocalStorage.ts
