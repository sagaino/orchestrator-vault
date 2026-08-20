---
title: Project Skeleton Template & Boilerplate Guide
type: pattern
tags: [synthesis, boilerplate, skeleton, project-setup]
created: 2026-08-12
updated: 2026-08-15
sources: ["[[03-Sources/documentation/AGENTS.md]]", "[[03-Sources/documentation/rules-react.md]]", "[[03-Sources/documentation/rules-api.md]]"]
---

# Project Skeleton Template & Boilerplate Guide

Panduan cetak biru struktur folder dan berkas *starter* wajib saat menginisialisasi proyek Frontend baru.

## Automation Contract

Section ini adalah kontrak authoritative untuk Personal AI Orchestrator. Potongan kode di bawahnya adalah referensi bentuk implementasi, bukan source yang harus disalin mentah.

- Blueprint ID: `frontend-vite`.
- Blueprint policy version: `3`; deterministic template version: `2`.
- Project dibuat sebagai Vite + React + TypeScript melalui Shadcn CLI `4.18.0` dengan Base UI dan preset Nova.
- Orchestrator wajib menjalankan `npx -y shadcn@4.18.0 add --all -y`; instalasi sebagian komponen tidak memenuhi blueprint. Versi CLI dikunci agar hasil onboarding dapat direproduksi.
- Setelah Shadcn selesai, orchestrator menerapkan deterministic template dari `templates/frontend-vite/` untuk struktur, dependency, konfigurasi, contoh modul Login, dan script verification. Normal path tidak memanggil coding agent dan menggunakan `0` token AI.
- Coding agent hanya menjadi fallback bila deterministic template gagal verification. Agent menerima error tail tanpa bulk-loading isi blueprint, hanya boleh mengubah file template yang diaudit, dan tidak boleh membaca atau menulis ulang `src/components/ui/`.
- Toolchain yang dikelola orchestrator memakai known-good TypeScript `~5.9.3`. Policy ini menormalkan output Shadcn dan setiap fallback sebelum instalasi agar konflik peer TypeScript 5 seperti kasus `react-i18next@15` sebelumnya tidak muncul kembali.
- Sebelum instalasi penuh, orchestrator menjalankan dependency-resolution preflight. Konflik resolusi mendapat satu deterministic retry setelah policy diterapkan ulang; `--force` dan `--legacy-peer-deps` tidak digunakan.
- Script minimum yang wajib tersedia dan lulus adalah `typecheck`, `lint`, dan `build`; `test` ikut dijalankan bila tersedia.
- Generated Shadcn UI diperlakukan sebagai vendored output: ESLint mengabaikan `src/components/ui/**` dan `src/hooks/use-mobile.ts`. TypeScript tetap menjalankan strict typecheck untuk seluruh `src`, tetapi unused-local checks diserahkan ke ESLint agar source Shadcn yang valid tidak memblokir baseline.
- Implementasi baru wajib type-safe tanpa `any`, tidak boleh membuat dummy/fallback authentication, tidak boleh mengarang kontrak API, dan memakai i18n untuk user-facing text.
- Nilai endpoint dan secret berasal dari environment/configuration yang eksplisit. Baseline hanya boleh membuat `.env.example` dengan placeholder aman; `.env` dan variannya diabaikan dan ditolak oleh initial-commit security gate.
- Repository baru wajib menjadi Git repository dan memiliki project-local `graphify-out/graph.json` sebelum diregistrasikan ke Wiki.
- Pembuatan dilakukan di staging. Registry, project page, index, dan Wiki log baru diperbarui setelah scaffold, verification, Git, dan Graphify berhasil; kegagalan harus di-rollback.

Path minimum yang divalidasi otomatis:

```text
src/components/ui/
src/hooks/useLocalStorage.ts
src/lib/constant/endpoints.ts
src/lib/axios.ts
src/lib/signature.ts
src/lib/error-utils.ts
src/pages/Login/
src/routes/
src/services/auth.ts
```

---

## 📁 Pohon Direktori Standar Wajib (`src/`)

```text
src/
├── components/
│   └── ui/                       # Shared presentation components (shadcn/ui)
├── hooks/                        # Shared non-business hooks (useLocalStorage.ts)
├── lib/                          # Central utilities & configurations
│   ├── constant/
│   │   └── endpoints.ts          # Centralized API endpoints
│   ├── i18n/
│   │   └── locales/              # Translation files (id.json, en.json)
│   ├── axios.ts                  # Centralized Axios instance with auth, sig & 401 interceptors
│   ├── signature.ts              # HMAC/SHA1 Request Signature Generator (crypto-js)
│   ├── error-utils.ts            # Error normalization & handling
│   └── types/                    # Shared TypeScript interfaces & types
├── services/                     # Global API service layer
├── pages/                        # Feature modules
│   ├── Login/                    # Starter auth UI with an explicit adapter boundary
│   ├── ProjectReady/             # Default authenticated landing page
│   └── [FeatureName]/            # Contoh: Gallery, PhotoTagging
│       ├── components/           # Feature-specific UI components
│       ├── hooks/                # Feature-specific custom hook (use[Feature].ts)
│       ├── types/                # Feature-specific TypeScript models/schemas
│       └── index.tsx             # Main Feature entry point
└── routes/                       # Route management
    ├── PrivateRoute.tsx          # Route guard untuk akses terproteksi
    ├── PublicRoute.tsx           # Route guard untuk akses publik
    └── index.tsx                 # Routing configuration (React Router)
```

---

## 📄 Berkas Boilerplate Dasar Wajib

1. **`src/lib/constant/endpoints.ts`**: Menyimpan semua daftar URL API.
2. **`src/lib/signature.ts`**: Helper penjelas & pembuat signature keamanan HTTP request berbasis `crypto-js` SHA1.
3. **`src/hooks/useLocalStorage.ts`**: Custom hook & helper helper `setData`, `getData`, `removeData` dengan enkripsi AES (`crypto-js`) dan event sync.
4. **`src/lib/axios.ts`**: Client HTTP tunggal dengan interceptor otomatis untuk Authorization Header, Signature (`sig`), Timestamp (`timestamp`), dan penanganan 401 Session Expiry.
5. **`src/lib/error-utils.ts`**: Helper penanganan pesan error API.
6. **`src/routes/PrivateRoute.tsx` & `PublicRoute.tsx`**: Pengaman rute halaman terproteksi.

### 📝 Template `src/hooks/useLocalStorage.ts`
```typescript
import { useEffect, useState } from "react";
import CryptoJS from "crypto-js";

type SetValue<T> = T | ((val: T) => T);

interface UseLocalStorageOptions {
  encrypted?: boolean;
}

const SECRET_KEY = `${import.meta.env.VITE_SECRET_KEY}`;

// Save encrypted data
export const setData = (storageKey: string, value: unknown) => {
  try {
    const jsonValue = JSON.stringify(value);
    const encrypted = CryptoJS.AES.encrypt(jsonValue, SECRET_KEY).toString();
    localStorage.setItem(storageKey, encrypted);
    window.dispatchEvent(new Event("storage"));
  } catch (e) {
    console.error("Gagal menyimpan di localStorage", e);
    return null;
  }
};

// Retrieve and decrypt data
export const getData = <T = unknown>(storageKey: string): T | null => {
  try {
    const encrypted = localStorage.getItem(storageKey);
    if (!encrypted) {
      return null;
    }
    const bytes = CryptoJS.AES.decrypt(encrypted, SECRET_KEY);
    const decrypted = bytes.toString(CryptoJS.enc.Utf8);
    if (!decrypted) {
      console.error(`Failed to decrypt data for key: ${storageKey} - removing corrupted data`);
      localStorage.removeItem(storageKey);
      return null;
    }
    return JSON.parse(decrypted) as T;
  } catch (e) {
    console.error(`Error retrieving data for key: ${storageKey}`, e);
    localStorage.removeItem(storageKey);
    return null;
  }
};

export const removeData = (storageKey: string) => {
  try {
    localStorage.removeItem(storageKey);
    window.dispatchEvent(new Event("storage"));
  } catch (e) {
    console.log("Gagal menghapus data di storage");
    return null;
  }
};

function useLocalStorage<T>(
  key: string,
  initialValue: T,
  options: UseLocalStorageOptions = {}
): [T, (value: SetValue<T>) => void] {
  const { encrypted = false } = options;

  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window !== "undefined") {
      try {
        if (encrypted) {
          // Use encrypted getData
          const item = getData<T>(key);
          if (item !== null) {
            return item;
          } else {
            if (initialValue) setData(key, initialValue);
            return initialValue;
          }
        } else {
          // Use regular localStorage
          const item = localStorage.getItem(key);
          if (item !== null) {
            return JSON.parse(item);
          } else {
            if (initialValue) localStorage.setItem(key, JSON.stringify(initialValue));
            return initialValue;
          }
        }
      } catch (error) {
        console.error(`Error reading from localStorage (key: ${key}):`, error);
        return initialValue;
      }
    }
    return initialValue;
  });

  // Sync state with localStorage changes
  useEffect(() => {
    const handleStorageChange = () => {
      if (typeof window !== "undefined") {
        try {
          if (encrypted) {
            const newValue = getData(key);
            if (newValue !== null) {
              setStoredValue(newValue);
            }
          } else {
            const newValue = localStorage.getItem(key);
            if (newValue !== null) {
              setStoredValue(JSON.parse(newValue));
            }
          }
        } catch (error) {
          console.error(`Error syncing localStorage (key: ${key}):`, error);
        }
      }
    };

    window.addEventListener("storage", handleStorageChange);
    return () => window.removeEventListener("storage", handleStorageChange);
  }, [key, encrypted]);

  // Update localStorage and state
  const setValue = (value: SetValue<T>) => {
    try {
      const newValue = value instanceof Function ? value(storedValue) : value;
      setStoredValue(newValue);
      if (typeof window !== "undefined") {
        if (encrypted) {
          setData(key, newValue);
        } else {
          localStorage.setItem(key, JSON.stringify(newValue));
          window.dispatchEvent(new Event("storage"));
        }
      }
    } catch (error) {
      console.error(`Error saving to localStorage (key: ${key}):`, error);
    }
  };

  return [storedValue, setValue];
}

export default useLocalStorage;
```

### 📝 Template `src/lib/signature.ts`
```typescript
import CryptoJS from "crypto-js";

export const getTimestamp = () => new Date().getTime();

type SignaturePrimitive = string | number | boolean | null | undefined;
type SignatureValue = SignaturePrimitive | SignatureValue[] | SignatureRecord;
type SignatureRecord = { [key: string]: SignatureValue };

function flattenObject(obj: SignatureRecord, prefix: string = ""): Record<string, SignaturePrimitive> {
  let flattened: Record<string, SignaturePrimitive> = {};
  for (const key in obj) {
    if (Object.prototype.hasOwnProperty.call(obj, key)) {
      const value = obj[key];
      const prefixedKey = prefix === "" ? key : `${prefix}.${key}`;
      if (typeof value === "object" && value !== null && !Array.isArray(value)) {
        const nested = flattenObject(value as SignatureRecord, prefixedKey);
        flattened = { ...flattened, ...nested };
      } else if (Array.isArray(value)) {
        flattened[prefixedKey] = "";
      } else {
        flattened[prefixedKey] = value;
      }
    }
  }
  return flattened;
}

function formatObject(obj: SignatureRecord): string {
  const flattened = flattenObject(obj);
  let result = "";
  Object.keys(flattened).sort().forEach((key) => {
    result += `${key}${flattened[key]}`;
  });
  return result;
}

export const makeSignature = (
  email: number,
  timestamp: number,
  data: SignatureRecord | FormData,
  isFormData?: boolean,
) => {
  const formDataObject: SignatureRecord = { email, timestamp };
  if (isFormData) {
    if (!(data instanceof FormData)) throw new Error("Expected FormData payload");
    for (const pair of data.entries()) {
      formDataObject[pair[0]] = typeof pair[1] === "string" ? pair[1] : pair[1].name;
    }
  }
  const payload = data instanceof FormData ? {} : data;
  const newPayload: SignatureRecord = { email, ...payload, timestamp };
  const newData = isFormData ? formDataObject : newPayload;
  const finalPayload = formatObject(newData);
  return CryptoJS.SHA1(finalPayload);
};
```

### 📝 Template `src/lib/axios.ts`
```typescript
import Axios from "axios";
import { getTimestamp, makeSignature } from "./signature";
import { LOCALSTORAGE_KEY } from "./constant/localstorage";
import { getData } from "@/hooks/useLocalStorage";

const API_URL = import.meta.env.VITE_API_URL;
if (!API_URL) throw new Error("Missing required environment value: VITE_API_URL");

export const axios = Axios.create({ baseURL: API_URL });

axios.interceptors.request.use(async (config) => {
  const token = getData<string>(LOCALSTORAGE_KEY.TOKEN);
  const timestamp = getTimestamp();

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  const signature = makeSignature(0, timestamp, config.data || {}, config.data instanceof FormData);

  config.headers["sig"] = signature;
  config.headers["email"] = "0";
  config.headers["timestamp"] = timestamp.toString();

  return config;
}, (error) => Promise.reject(error));

axios.interceptors.response.use(
  (response) => response,
  async (error) => {
    const status = error.response?.status;
    if (status === 401) {
      localStorage.removeItem(LOCALSTORAGE_KEY.TOKEN);
      localStorage.removeItem(LOCALSTORAGE_KEY.REFRESH_TOKEN);
      window.location.href = "/login";
    }
    return Promise.reject(error);
  }
);

export default axios;
```

## 🔑 Starter Sample Feature: Modul Contoh Login (`src/pages/Login`)

Saat menginisialisasi project FE baru, deterministic template version `2` menyediakan modul Login sebagai contoh integrasi React Hook Form, Zod, TanStack Query, Shadcn, encrypted local storage, toast, dan i18n. Coding agent tidak diperlukan pada normal path.

1. **Environment contract**: `VITE_API_URL`, `VITE_LOGIN_ENDPOINT`, dan `VITE_SECRET_KEY` hanya tersedia sebagai placeholder di `.env.example`. HTTP client dan endpoint helper gagal secara eksplisit ketika konfigurasi yang diperlukan belum diberikan; tidak ada fallback localhost atau endpoint produksi buatan.
2. **Auth adapter boundary**: `src/services/auth.ts` mengekspor `configureAuthLoginAdapter`. Baseline tidak mengirim network request dan tidak mengembalikan dummy user/token. Project harus memasang adapter yang mengembalikan response sesuai kontrak project sebelum Login digunakan.
3. **Type-safe form**: `src/pages/Login/types/request.ts` membuat schema Zod dari `TFunction`; semua validation message berasal dari locale. Login page memakai React Hook Form dan `@hookform/resolvers`.
4. **Mutation behavior**: `useLogin` menjalankan adapter melalui TanStack Query, memvalidasi keberadaan token, menyimpan token/user, dan menampilkan success/error toast yang sudah dilokalisasi.
5. **Routing**: `PublicRoute` dan `PrivateRoute` membaca token dari storage. `ProjectReadyPage` berada di file terpisah agar router tetap memenuhi Fast Refresh lint.
6. **Generated Shadcn compatibility**: strict TypeScript tetap aktif, sedangkan `noUnusedLocals` dan `noUnusedParameters` dinonaktifkan pada compiler karena `shadcn add --all` dapat menghasilkan import yang sah tetapi tidak terpakai. ESLint tetap memeriksa source milik project dan mengabaikan generated UI.

---

## 🛠️ Perintah Wajib Inisialisasi Shadcn UI (Vite + Base UI + Preset Nova)

Gunakan dua langkah perintah berikut untuk membuat proyek FE baru sekaligus memasang seluruh komponen UI Shadcn:

```bash
# 1. Inisialisasi Vite + Base UI + Preset Nova + Nama Proyek
npx -y shadcn@4.18.0 init -t vite -b base -n <nama-proyek> -p nova --no-monorepo -y

# 2. Install Seluruh Komponen UI Shadcn
npx -y shadcn@4.18.0 add --all -y
```

---

## 🚀 Cara Membuat Project melalui Orchestrator

User tidak perlu membuat tiket atau mengedit Wiki secara manual. Jalankan dari Personal AI Orchestrator:

```bash
npm run add-project -- new <nama-project> --path <absolute-target-path> --by user
```

Orchestrator membaca kontrak blueprint ini, menjalankan dua command Shadcn di atas, menerapkan deterministic template, memasang dependency, menjalankan verification, membuat baseline Git, membangun Graphify, lalu meregistrasikan project ke Wiki. Coding agent hanya dipanggil sebagai fallback terukur ketika deterministic verification gagal.
