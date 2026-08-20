---
title: "Encrypted Event-Driven Reactive Storage Hook"
type: pattern
tags: [pattern, frontend, state-management, crypto, localstorage, react-hooks]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127443075-f8fd0716
sources: ["Harvest 1787127443075 F8fd0716.json"]
---

# Encrypted Event-Driven Reactive Storage Hook

Encrypted reactive client storage synchronization with AES encryption and custom event dispatching.

## 1. Overview & Architecture

Pola reactive state synchronization berbasis Web Storage yang diperkuat dengan enkripsi AES dan sinkronisasi lintas komponen via browser storage events.

## 2. Implementation & Code Structure

src/renderer/src/
├── hooks/
│   └── useLocalStorage.ts      # Custom hook with reactive listener and AES crypto
└── lib/
    └── clear-auth-session.ts   # Atomic session removal trigger

## 3. Key Implementation Points

- Enkripsi dan dekripsi otomatis menggunakan CryptoJS AES.
- Sinkronisasi reaktif real-time menggunakan dispatch custom `storage` event pada perubahan data.
- Fallback graceful dan pembersihan otomatis jika terjadi korupsi data payload yang tersimpan.

## 4. Code Examples

### AES Encrypted Storage Setter & Getter with Custom Event Dispatch for Cross-Component Sync

```typescript
// src/renderer/src/hooks/useLocalStorage.ts
export const setData = (storageKey: string, value: any) => {
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

export const getData = (storageKey: string) => {
  try {
    const encrypted = localStorage.getItem(storageKey);
    if (!encrypted) return null;
    const bytes = CryptoJS.AES.decrypt(encrypted, SECRET_KEY);
    const decrypted = bytes.toString(CryptoJS.enc.Utf8);
    if (!decrypted) {
      localStorage.removeItem(storageKey);
      return null;
    }
    return JSON.parse(decrypted);
  } catch (e) {
    localStorage.removeItem(storageKey);
    return null;
  }
};
```

## 5. Considerations & Best Practices

- Enkripsi client-side AES melindungi dari inspeksi kasual di LocalStorage namun secret key tetap berada di environment client.
- Event 'storage' asli browser hanya mentrigger tab lain; `window.dispatchEvent(new Event('storage'))` diperlukan agar komponen di tab yang sama ikut sinkron seketika.

## 6. Related Knowledge

- `React State Management Patterns`
- `Web Cryptography & Local Storage Security`

## 7. Source

- Harvest 1787127443075 F8fd0716.json
