---
title: "Structured Inline Code Comments and Targeted Revision Formatting for Agent Review Cycles"
type: pattern
tags: [pattern, orchestrator-promotion]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: task-007-20260819T015030Z-3fb07f02
sources: ["Task 007 20260819T015030Z 3fb07f02.json"]
---

# Structured Inline Code Comments and Targeted Revision Formatting for Agent Review Cycles

## Overview

Pola penataan dan pemformatan inline code comments terstruktur (file, line, comment) ke dalam prompt revisi coding agent pada siklus review / request-changes untuk memandu perbaikan kode secara presisi.

## Purpose

Pola ini reusable untuk berbagai sistem orchestrator atau tool review coding agent yang membutuhkan konversi komentar baris interaktif manusia menjadi prompt revisi agen yang terstruktur dan terfokus.

## Considerations

- Validasi array inlineComments secara robust agar mengabaikan entri tidak valid (tanpa file/line/pesan)
- Pemisahan bersih antara feedback umum (macro review) dan catatan baris spesifik (micro review)
- Penambahan instruksi penegasan prioritas perbaikan pada prompt agar agen tidak mengabaikan lokasi baris yang ditunjuk

## Related Knowledge

- [[AGENTS]]
- [[01-Knowledge/concepts/ai-engineering-guide]]
- [[01-Knowledge/patterns/performance/agent-context-token-optimization-and-error-sanitization]]

## Source

- Task 007 20260819T015030Z 3fb07f02.json
