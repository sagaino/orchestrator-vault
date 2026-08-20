---
title: "Client-Side Encrypted Reactive Local Storage"
type: pattern
tags: [pattern, frontend, react-hook, storage, encryption, crypto-js, state-management]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787109583565-5ff8f29f
sources: ["Harvest 1787109583565 5ff8f29f.json"]
---

# Client-Side Encrypted Reactive Local Storage

Reactive client-side local storage hook with transparent AES encryption and event-driven state synchronization.

## 1. Overview & Architecture

A reactive React hook wrapping localStorage with transparent AES encryption, corruption resilience, and intra-window state synchronization.

## 2. Implementation & Code Structure

src/renderer/src/hooks/useLocalStorage.ts

## 3. Key Implementation Points

- AES encryption/decryption using CryptoJS
- Custom window storage event dispatch for instant intra-window reactivity
- Automatic cleanup of corrupted or unparseable entries

## 4. Code Examples

### AES encrypted storage helper with event dispatching for cross-component sync

```typescript
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

- Symmetric encryption key is bundled client-side and protects against passive snooping, not determined local root attacks.
- Automatic corruption pruning prevents runtime parsing crashes.

## 6. Related Knowledge

- Encrypted Reactive Localstorage
- `Web Storage API and Cryptography`

## 7. Source

- Harvest 1787109583565 5ff8f29f.json
