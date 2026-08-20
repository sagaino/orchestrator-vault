---
title: i18next Localization
type: concept
tags: [entity, i18n, localization, i18next]
created: 2026-08-12
updated: 2026-08-14
sources: ["[[03-Sources/documentation/AGENTS.md]]", "[[03-Sources/documentation/rules-react.md]]"]
---

# i18next Localization

Sistem lokalisasi bahasa untuk memastikan tidak ada teks antarmuka yang ditulis secara hardcoded.

## Aturan Penggunaan
- **Default Locale**: Bahasa Indonesia (`id`).
- **Penyimpanan Kunci**: Kunci terjemahan disimpan di berkas `src/lib/i18n/locales/`.
- **Penggunaan di UI**: Selalu gunakan hook `useTranslation()` untuk menampilkan teks pengguna.
- **Dilarang Hardcoded String**: Dilarang menulis string teks langsung pada elemen JSX (seperti `<button>Submit</button>`), gunakan `<button>{t('common.submit')}</button>`.

## Halaman Terkait
- [[01-Knowledge/concepts/architecture/frontend-architecture|Frontend Architecture Hub]]
- [[01-Knowledge/concepts/forbidden-behaviors|Forbidden Behaviors]]
