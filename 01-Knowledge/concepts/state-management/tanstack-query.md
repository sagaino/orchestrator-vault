---
supersession: "ACTIVE"
review_by: "2027-02-18"
owner: "knowledge-curator"
confidence: 0.5
orchestrator_run: "legacy-migration:p8-0-provenance-20260822T103839Z-efa8f1a9"
provenance_schema: 1
title: TanStack Query
type: concept
tags: [entity, tanstack-query, server-state, cache]
created: 2026-08-12
updated: 2026-08-14
sources: ["[[03-Sources/documentation/rules-react.md]]"]
---

# TanStack Query

Pustaka utama pengelolaan server state dan caching data API.

## Aturan Penggunaan
- **Single Source of Truth**: TanStack Query digunakan sebagai satu-satunya rujukan untuk data server.
- **Cache Management**: Setiap mutasi yang berhasil wajib melakukan pembaruan atau pembatalan cache (`invalidateQueries`) pada query key terkait.
- **Penggunaan Hook**: Dipanggil di dalam Custom Hook (`use[Feature].ts`), bukan langsung di dalam file tampilan komponen `.tsx`.

## Halaman Terkait
- [[01-Knowledge/concepts/architecture/frontend-architecture|Frontend Architecture Hub]]
- [[01-Knowledge/concepts/architecture/state-logic-separation|State & Logic Separation]]
