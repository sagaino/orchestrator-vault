---
title: "Interactive Raw Knowledge Ingestion Studio Modal with Multi-Destination Routing and Domain Filtering"
type: pattern
tags: [pattern, orchestrator-promotion]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: task-016-20260819T013216Z-8dc91dd0
sources: ["Task 016 20260819T013216Z 8dc91dd0.json"]
---

# Interactive Raw Knowledge Ingestion Studio Modal with Multi-Destination Routing and Domain Filtering

## Overview

Pola antarmuka modal Knowledge Ingest Studio dan domain tag filtering pada frontend: form interaktif multimodal (inferred title, domain selector, tipe schema, destinasi routing CANDIDATE vs WIKI, raw textarea), feedback state AI synthesis, notifikasi toast/alert, dan TanStack Query cache invalidation.

## Purpose

TASK-016 memperkenalkan pola antarmuka Knowledge Ingest Studio dan Domain Tag Pill Filter yang reusable untuk aplikasi manajemen knowledge/LLM Wiki lainnya.

## Considerations

- Pastikan form reset dan status alert dibersihkan saat modal ditutup atau submit berhasil.
- Gunakan visual loading state / spinner saat backend AI memproses sintesis dokumen teks mentah.
- Invalidasikan TanStack Query cache (knowledgeCandidates, knowledgeHealth) pasca-mutasi sukses agar catalog data langsung tersinkronisasi.
- Dukung multi-destination routing switch (CANDIDATE vs WIKI) untuk mengakomodasi alur human-in-the-loop review vs direct promotion.

## Related Knowledge

- [[01-Knowledge/patterns/architecture/automated-raw-knowledge-ingestion-and-wiki-synthesis]]
- [[01-Knowledge/concepts/architecture/api-service-data-flow]]
- [[01-Knowledge/concepts/state-management/tanstack-query]]
- [[01-Knowledge/patterns/frontend/react-ui-rules]]

## Source

- Task 016 20260819T013216Z 8dc91dd0.json

## Update from TASK-018 — 2026-08-19

<!-- orchestrator-run:task-018-20260819T020614Z-ab205748 -->
Pola antarmuka modal Knowledge Ingest Studio multi-tab (Raw Text & Codebase Harvester) dan integrasi TanStack Query mutation: navigasi tab dinamis, form scanning path repositori lokal dengan selector domain, visual loading spinner selama analisis arsitektur repositori, feedback terstruktur daftar artikel knowledge yang dihasilkan, serta sinkronisasi/invalidasi cache katalog knowledge.

- Rationale: Task TASK-018 memperkaya modal Knowledge Ingest Studio (yang sebelumnya dibuat di TASK-016) dengan tab kedua untuk Codebase Architecture Harvesting (memanggil POST /api/knowledge/harvest via useHarvestKnowledge mutation hook). Pola penanganan form scanning repositori lokal, visual loading state selama analisis arsitektur, dan feedback breakdown artikel hasil harvest merupakan pembaruan langsung pada pola interactive-knowledge-ingest-modal-flow.md.
- Source: Task 018 20260819T020614Z Ab205748.json
