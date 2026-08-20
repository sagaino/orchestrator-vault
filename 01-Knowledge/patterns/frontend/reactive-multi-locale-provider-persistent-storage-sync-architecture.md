---
title: "reactive-multi-locale-provider-persistent-storage-sync-architecture"
type: pattern
tags: [pattern, frontend, i18n, localization, react-context, localstorage, services]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787132260416-42b894f9
sources: ["Harvest 1787132260416 42b894f9.json"]
---

# reactive-multi-locale-provider-persistent-storage-sync-architecture

Arsitektur Reactive Multi-Locale Provider dengan sinkronisasi local storage persisten dan resolusi dynamic fallback.

## 1. Overview & Architecture

Pola arsitektur penyedia multi-bahasa reaktif (i18n) yang mengintegrasikan i18next dengan React Provider dan penyimpanan persistent state. Memungkinkan transisi bahasa secara real-time di seluruh komponen UI tanpa reload aplikasi.

## 2. Implementation & Code Structure

src/lib/i18n/i18n.ts (i18next configuration & resource dictionary registration)
├── src/lib/i18n/i18nProvider.tsx (React Provider syncing language state from LocalStorage)
├── src/lib/i18n/locales/*.json (Multi-locale translation dictionaries)
└── src/components/common/LanguagesDropdown.tsx (Language switcher UI component)

## 3. Key Implementation Points

- Inisialisasi `i18next` terpusat dengan dukungan fallback language (`fallbackLng: 'en'`).
- Sinkronisasi reaktif antara state penyimpanan lokal (`useLocalStorage`) dan `i18next.changeLanguage` di level Provider.
- Integrasi modular dengan kamus locale JSON statis (`en.json`, `id.json`).
- Komponen switch bahasa yang dapat digunakan ulang (`LanguagesDropdown`) dengan feedback instan.

## 4. Code Examples

### i18next instance configuration with multi-language catalogs and fallback resolution.

```typescript
// src/lib/i18n/i18n.ts
import i18next from "i18next";
import en from "./locales/en.json";
import id from "./locales/id.json";
import { initReactI18next } from "react-i18next";

i18next
  .use(initReactI18next)
  .init({
    resources: {
      en: { translation: en },
      id: { translation: id }
    },
    lng: "en",
    fallbackLng: "en",
    interpolation: {
      escapeValue: false,
    },
  });

export default i18next;
```

### Reactive I18n Context Provider synchronized with encrypted persistent local storage.

```typescript
// src/lib/i18n/i18nProvider.tsx
import React, { useEffect } from "react";
import { I18nextProvider } from "react-i18next";
import i18next from "./i18n";
import useLocalStorage from "../../hooks/useLocalStorage";

const I18nProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [language] = useLocalStorage("language", "en");

  useEffect(() => {
    i18next.changeLanguage(language);
  }, [language]);

  return <I18nextProvider i18n={i18next}>{children}</I18nextProvider>;
};

export default I18nProvider;
```

## 5. Considerations & Best Practices

- `escapeValue: false` aman digunakan karena React sudah menangani XSS protection secara native pada string rendering.
- Sinkronisasi dengan `useLocalStorage` memastikan bahasa yang dipilih pengguna tetap aktif antar sesi dan antar tab browser.
- Struktur kamus JSON modular memudahkan penambahan locale baru tanpa mengubah arsitektur inti.

## 6. Related Knowledge

- `i18next & react-i18next Architecture`
- `Localization & Internationalization (L10n/I18n)`
- `Persistent Locale Context Synchronization`

## 7. Source

- Harvest 1787132260416 42b894f9.json
