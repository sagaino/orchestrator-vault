---
title: "AES-Encrypted Reactive Storage Hook with Multi-Window Sync"
type: pattern
tags: [pattern, frontend, react, hooks, localstorage, crypto-js, encryption, state-management]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787111316881-96cf68d4
sources: ["Harvest 1787111316881 96cf68d4.json"]
---

# AES-Encrypted Reactive Storage Hook with Multi-Window Sync

AES-encrypted React local storage hook with live window event synchronization and corrupted data self-healing.

## 1. Overview & Architecture

A custom React hook and helper utility that provides transparent AES encryption for localStorage entries along with cross-window/cross-component state synchronization and automated corrupted-data self-healing.

## 2. Implementation & Code Structure

src/renderer/src/
├── hooks/
│   └── useLocalStorage.ts      # Encrypted storage hook and standalone getData/setData helpers
├── config/constant/
│   └── localstorage.ts         # Central storage key constants (e.g. TOKEN, USER, PERMISSIONS)
└── services/                   # Reads token securely via getData(LOCALSTORAGE_KEY.TOKEN)

## 3. Key Implementation Points

- Transparent AES encryption/decryption using CryptoJS with a configurable secret key.
- Self-healing logic that catches decryption/parse exceptions and purges invalid keys automatically.
- Custom 'storage' DOM Event dispatching enabling instant cross-component and cross-window reactivity.
- Support for functional state updates (value instanceof Function ? value(storedValue) : value).

## 4. Code Examples

### AES-encrypted custom React hook with multi-window event synchronization and auto-healing on corruption

```typescript
// src/renderer/src/hooks/useLocalStorage.ts
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

function useLocalStorage<T>(
  key: string,
  initialValue: T,
  options: UseLocalStorageOptions = {}
): [T, (value: SetValue<T>) => void] {
  const { encrypted = false } = options;
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = encrypted ? getData(key) : localStorage.getItem(key);
      if (item !== null) return encrypted ? item : JSON.parse(item);
      return initialValue;
    } catch {
      return initialValue;
    }
  });

  useEffect(() => {
    const handleStorageChange = () => {
      try {
        const newValue = encrypted ? getData(key) : localStorage.getItem(key);
        setStoredValue(newValue !== null ? (encrypted ? newValue : JSON.parse(newValue)) : initialValue);
      } catch (error) {
        console.error("Error syncing storage:", error);
      }
    };
    window.addEventListener("storage", handleStorageChange);
    return () => window.removeEventListener("storage", handleStorageChange);
  }, [key, encrypted, initialValue]);

  const setValue = (value: SetValue<T>) => {
    const newValue = value instanceof Function ? value(storedValue) : value;
    setStoredValue(newValue);
    if (encrypted) {
      setData(key, newValue);
    } else {
      localStorage.setItem(key, JSON.stringify(newValue));
      window.dispatchEvent(new Event("storage"));
    }
  };

  return [storedValue, setValue];
}
```

## 5. Considerations & Best Practices

- Client-side symmetric keys protect against casual plain-text inspection on disk, but true hardware-level security in Electron should ideally leverage SafeStorage API for sensitive master keys.
- Dispatching custom 'storage' events is essential because native window.onstorage only fires in other windows/tabs, not the emitting window.
- Corrupted data (e.g. invalid JSON or decrypt failure) must be wiped to prevent infinite render crashes.

## 6. Related Knowledge

- `React Custom Hooks`
- `Web Storage API`
- `AES Symmetric Encryption (CryptoJS)`
- `DOM Custom Events`

## 7. Source

- Harvest 1787111316881 96cf68d4.json
