---
title: "Secure Encrypted Persistent State Store with Cross-Tab Event Synchronization"
type: pattern
tags: [pattern, frontend, state-management, localstorage, crypto, react-hook, security]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787132260416-42b894f9
sources: ["Harvest 1787132260416 42b894f9.json"]
---

# Secure Encrypted Persistent State Store with Cross-Tab Event Synchronization

Secure Encrypted Persistent State Store with Cross-Tab Event Synchronization

## 1. Overview & Architecture

Pola manajemen state terenkripsi pada browser storage yang mengombinasikan utilitas stand-alone (bisa diakses di luar lifecycle React seperti pada Axios interceptors) dan custom hook reaktif dengan sinkronisasi multi-tab/multi-window.

## 2. Implementation & Code Structure

src/
├── hooks/
│   └── useLocalStorage.ts        # Encrypted storage utilities (getData, setData) and custom React hook
├── config/constant/
│   └── localstorage.ts           # Type-safe storage key dictionary constants

## 3. Key Implementation Points

- AES Encryption/Decryption pipeline using crypto-js.
- Self-healing corrupted state handler with automatic invalidation and removal.
- Dual synchronization: same-window custom event dispatch and cross-window native storage listener.

## 4. Code Examples

### AES-encrypted LocalStorage getter/setter with corrupted state auto-purge and reactive event dispatching.

```typescript
// src/hooks/useLocalStorage.ts
import { useEffect, useState } from "react";
import CryptoJS from "crypto-js";

type SetValue<T> = T | ((val: T) => T);
interface UseLocalStorageOptions {
  encrypted?: boolean;
}

const SECRET_KEY = `${import.meta.env.VITE_SECRET_KEY}`;

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

### Reactive useLocalStorage hook synchronizing local React state with browser storage across tabs.

```typescript
// src/hooks/useLocalStorage.ts (Hook Implementation)
function useLocalStorage<T>(
  key: string,
  initialValue: T,
  options: UseLocalStorageOptions = {}
): [T, (value: SetValue<T>) => void] {
  const { encrypted = false } = options;

  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window !== "undefined") {
      try {
        const item = encrypted ? getData(key) : JSON.parse(localStorage.getItem(key) || "null");
        return item !== null ? item : initialValue;
      } catch {
        return initialValue;
      }
    }
    return initialValue;
  });

  useEffect(() => {
    const handleStorageChange = () => {
      const newValue = encrypted ? getData(key) : JSON.parse(localStorage.getItem(key) || "null");
      setStoredValue(newValue !== null ? newValue : initialValue);
    };

    window.addEventListener("storage", handleStorageChange);
    return () => window.removeEventListener("storage", handleStorageChange);
  }, [key, encrypted, initialValue]);

  const setValue = (value: SetValue<T>) => {
    const valueToStore = value instanceof Function ? value(storedValue) : value;
    setStoredValue(valueToStore);
    if (encrypted) {
      setData(key, valueToStore);
    } else {
      localStorage.setItem(key, JSON.stringify(valueToStore));
      window.dispatchEvent(new Event("storage"));
    }
  };

  return [storedValue, setValue];
}
```

## 5. Considerations & Best Practices

- Browser-side AES keys mitigate casual inspection but are not impervious to reverse engineering; sensitive tokens should also rely on secure server-side TTL and refresh mechanisms.
- Custom storage event dispatching (`window.dispatchEvent(new Event('storage'))`) is necessary because native storage events only fire across separate browser tabs/windows, not within the same document window.

## 6. Related Knowledge

- `Web Cryptography & AES-256`
- `Cross-tab React State Synchronization`
- `Browser Storage Resiliency Patterns`

## 7. Source

- Harvest 1787132260416 42b894f9.json
