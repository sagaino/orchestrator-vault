---
title: "Interactive Inline Diff Annotations and Review Revision Modal Pattern"
type: pattern
tags: [pattern, orchestrator-promotion]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: task-017-20260819T015532Z-6078715f
sources: ["Task 017 20260819T015532Z 6078715f.json"]
---

# Interactive Inline Diff Annotations and Review Revision Modal Pattern

## Overview

Pola frontend interaktif untuk Diff Viewer dan modal Request Changes: parsing hunk header patch git diff untuk kalkulasi line numbers, gutter hover button untuk inline notes per baris kode, visualisasi list catatan tersemat pada baris diff, rekapitulasi multi-file annotations pada modal revisi, dan pengiriman payload terstruktur inlineComments ke API agent revision.

## Purpose

Pola antarmuka code review interaktif dengan kalkulasi nomor baris git patch, inline annotations per baris, dan rekapitulasi multi-file pada modal revisi bersifat reusable untuk berbagai developer tools, dashboard CI/CD, maupun interface review coding agent berbasis React & TypeScript.

## Considerations

- Kalkulasi nomor baris Git patch memerlukan parsing hunk header @@ -old,len +new,len @@ yang akurat agar nomor baris sinkron dengan file target
- Gutter hover trigger harus dibatasi pada baris konten kode (menghindari baris header diff atau hunk boundary)
- Sinkronisasi state anotasi antara visual DiffViewer dan form rekapitulasi modal Request Changes dengan handler penambahan/penghapusan yang reaktif
- Integrasi payload inlineComments [{ file, line, comment }] bersama teks macro feedback menuju service API review

## Related Knowledge

- [[AGENTS]]
- [[01-Knowledge/patterns/architecture/structured-inline-code-comments-review-workflow]]
- [[01-Knowledge/concepts/architecture/state-logic-separation]]
- [[01-Knowledge/patterns/frontend/react-ui-rules]]

## Source

- Task 017 20260819T015532Z 6078715f.json
