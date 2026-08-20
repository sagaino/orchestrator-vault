---
title: "Encrypted Cross-Tab Reactive Storage Hook with Event Dispatching"
type: pattern
tags: [pattern, frontend, localstorage, encryption, cross-tab-sync, react-hook]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127791371-cc78a949
sources: ["Harvest 1787127791371 Cc78a949.json"]
---

# Encrypted Cross-Tab Reactive Storage Hook with Event Dispatching

Encrypted Cross-Tab Reactive Storage Hook with Event Dispatching.

## 1. Overview & Architecture

Custom reactive localStorage hook with AES encryption, corrupted payload self-healing, and unified same-tab/cross-tab state synchronization.

## 2. Implementation & Code Structure

- src/hooks/useLocalStorage.ts: AES encrypted storage hook with custom storage event dispatcher
- src/config/constant/localstorage.ts: Storage key constants

## 3. Key Implementation Points

- AES encryption/decryption on read and write with auto-purge on corruption
- Manual window.dispatchEvent(new Event('storage')) for same-tab reactive propagation
- Native storage event listener for automatic cross-tab synchronization

## 4. Code Examples

### Encrypted localStorage helper with storage event dispatching

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

- Client-side AES encryption is an obfuscation and integrity measure, not a replacement for backend security

## 6. Related Knowledge

- `security/client-side-storage-encryption`
- `react/storage-event-cross-tab-sync`

## 7. Source

- Harvest 1787127791371 Cc78a949.json
