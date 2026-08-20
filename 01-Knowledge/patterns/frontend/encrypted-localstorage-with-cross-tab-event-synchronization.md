---
title: "Encrypted LocalStorage with Cross-Tab Event Synchronization"
type: pattern
tags: [pattern, frontend, storage, encryption, react-hooks, crypto-js, state-management]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787112018130-d8cc3a8b
sources: ["Harvest 1787112018130 D8cc3a8b.json"]
---

# Encrypted LocalStorage with Cross-Tab Event Synchronization

Encrypted LocalStorage persistence hook and helper functions with cross-tab reactive synchronization and self-healing data recovery.

## 1. Overview & Architecture

Penyimpanan state client-side terenkripsi menggunakan AES (CryptoJS) yang membungkus Web Storage API dengan dukungan reaktivitas React hook dan event synchronization antar-tab/window.

## 2. Implementation & Code Structure

src/hooks/useLocalStorage.ts
├── setData() (Standalone encrypted setter)
├── getData() (Standalone encrypted getter with self-healing)
├── removeData() (Standalone remover with event dispatch)
└── useLocalStorage<T>() (Reactive state hook with event listener)

## 3. Key Implementation Points

- AES-256 symmetric encryption/decryption using CryptoJS with environment-configured secret key.
- Self-healing error handling that purges corrupted local storage keys automatically.
- Reactive synchronization across components and browser tabs via Window 'storage' event dispatching.
- Dual-mode support for both encrypted and standard JSON persistence.

## 4. Code Examples

### Core encryption/decryption utilities with automatic corruption recovery and same-window event dispatch.

```typescript
import { useEffect, useState } from "react";
import CryptoJS from "crypto-js";

type SetValue<T> = T | ((val: T) => T);

interface UseLocalStorageOptions {
  encrypted?: boolean;
}

const SECRET_KEY = `${import.meta.env.VITE_SECRET_KEY}`;

// Save encrypted data
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

// Retrieve and decrypt data with self-healing corrupted data cleanup
export const getData = (storageKey: string) => {
  try {
    const encrypted = localStorage.getItem(storageKey);
    if (!encrypted) return null;
    const bytes = CryptoJS.AES.decrypt(encrypted, SECRET_KEY);
    const decrypted = bytes.toString(CryptoJS.enc.Utf8);
    if (!decrypted) {
      console.error(`Failed to decrypt data for key: ${storageKey} - removing corrupted data`);
      localStorage.removeItem(storageKey);
      return null;
    }
    return JSON.parse(decrypted);
  } catch (e) {
    console.error(`Error retrieving data for key: ${storageKey}`, e);
    localStorage.removeItem(storageKey);
    return null;
  }
};
```

### React hook integrating synchronized state with cross-tab window event listener and encrypted storage.

```typescript
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
          const item = getData(key);
          if (item !== null) return item;
          if (initialValue) setData(key, initialValue);
          return initialValue;
        } else {
          const item = localStorage.getItem(key);
          if (item !== null) return JSON.parse(item);
          if (initialValue) localStorage.setItem(key, JSON.stringify(initialValue));
          return initialValue;
        }
      } catch (error) {
        return initialValue;
      }
    }
    return initialValue;
  });

  useEffect(() => {
    const handleStorageChange = () => {
      if (typeof window !== "undefined") {
        try {
          if (encrypted) {
            const newValue = getData(key);
            if (newValue !== null) setStoredValue(newValue);
          } else {
            const newValue = localStorage.getItem(key);
            if (newValue !== null) setStoredValue(JSON.parse(newValue));
          }
        } catch (error) {
          console.error(`Error syncing localStorage:`, error);
        }
      }
    };

    window.addEventListener("storage", handleStorageChange);
    return () => window.removeEventListener("storage", handleStorageChange);
  }, [key, encrypted]);

  const setValue = (value: SetValue<T>) => {
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
  };

  return [storedValue, setValue];
}
```

## 5. Considerations & Best Practices

- Client-side encryption using a frontend environment key mitigates casual plaintext DevTools tampering/inspection, but is not equivalent to secure server-side hardware vaults.
- Dispatches 'storage' custom event so changes within the same tab update all active hooks immediately, alongside native multi-tab storage events.
- Auto-deletes invalid/tampered ciphertext on decryption failure to prevent permanent app lockup or crashes.

## 6. Related Knowledge

- `Web Storage API`
- `CryptoJS.AES`
- `React custom hooks`
- `Cross-tab communication patterns`

## 7. Source

- Harvest 1787112018130 D8cc3a8b.json
