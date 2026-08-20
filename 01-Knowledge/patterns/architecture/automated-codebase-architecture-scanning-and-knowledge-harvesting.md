---
title: "Automated Codebase Architecture Scanning and Knowledge Harvesting Engine"
type: pattern
tags: [pattern, orchestrator-promotion]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: task-008-20260819T020208Z-3871c5f4
sources: ["Task 008 20260819T020208Z 3871c5f4.json"]
---

# Automated Codebase Architecture Scanning and Knowledge Harvesting Engine

## Overview

Pola arsitektur Codebase Knowledge Harvester yang melakukan pemindaian struktur repositori dan AST (auth, error handling, DB transactions, folder layout) secara terotomasi, mensintesis best practice ke format markdown terstruktur melalui LLM mode plan berdasar template domain, serta menyimpan immutable source artifact, routing WIKI/CANDIDATE berdasar confidence threshold, dan memperbarui index.md/wiki-log.md secara transaksional.

## Purpose

Pola arsitektur scanning repositori lokal dan harvesting knowledge ini sangat reusable lintas project untuk mengekstraksi best practice kode ke dalam LLM Wiki. Belum ada dokumen existing di 01-Knowledge yang mencakup mekanisme codebase architecture scanning & AST extraction.

## Considerations

- Validasi path repositori dan whitelist domain (frontend, backend, mobile, devops, architecture, general) sebelum pemindaian guna mencegah unhandled exceptions.
- Batasi kedalaman traversal rekursif (depth <= 4) dan filter direktori non-arsitektur (node_modules, dist, .git, graphify-out) untuk efisiensi I/O.
- Gunakan JSON Schema terikat saat invoking LLM agent (mode plan) untuk menjamin struktur data ekstraksi yang deterministik dan valid.
- Terapkan confidence threshold (>= 0.9 untuk promosi langsung ke 01-Knowledge, < 0.9 diarahkan ke 05-Knowledge-Candidates) dengan template domain yang sesuai.
- Simpan immutable source run artifact di 03-Sources/other/orchestrator-runs/ dan lakukan pembaruan atomic pada index.md serta wiki-log.md.

## Related Knowledge

- [[01-Knowledge/patterns/architecture/automated-raw-knowledge-ingestion-and-wiki-synthesis]]
- [[01-Knowledge/concepts/ai-engineering-guide]]
- [[AGENTS]]

## Source

- Task 008 20260819T020208Z 3871c5f4.json
