---
title: "Agent Context Token Optimization and Error Tail Sanitization"
type: pattern
tags: [pattern, orchestrator-promotion]
created: 2026-08-17
updated: 2026-08-17
orchestrator_run: task-002-20260817T023357Z-ebc176c5
sources: ["Task 002 20260817T023357Z Ebc176c5.json"]
---

# Agent Context Token Optimization and Error Tail Sanitization

## Overview

Pola deterministik untuk mengoptimalkan efisiensi token context LLM pada arsitektur AI agent melalui 3 teknik: (1) Top-K bounded knowledge retrieval dengan ekstraksi ringkas section esensial, (2) Sanitasi error tail dan output terminal (membersihkan ANSI codes, memfilter stack trace node_modules/internal runtime, dan menduplikasi baris berulang), serta (3) Kompresi prompt terstruktur pada tahap evaluasi/retrospective.

## Purpose

Pola optimasi token context agent melalui pembatasan retrieval knowledge, sanitasi stack trace/terminal error, dan kompresi prompt retrospective adalah pola reusable (durable) yang dapat diterapkan pada berbagai sistem AI orchestrator & coding agent untuk memangkas konsumsi token hingga lebih dari 30% tanpa mengorbankan kualitas eksekusi atau sinyal diagnostik.

## Considerations

- Pastikan filter error tail hanya memangkas node_modules dan internal runtime, bukan stack trace dari file source code internal project.
- Ekstraksi ringkas knowledge section harus tetap menyertakan heading, purpose, dan batasan kontraktual penting tanpa boilerplate atau raw code yang berlebihan.
- Kompresi prompt retrospective harus tetap mematuhi schema JSON yang ketat agar parsing hasil evaluasi tetap deterministik.

## Related Knowledge

- [[01-Knowledge/concepts/ai-engineering-guide]]
- [[01-Knowledge/patterns/performance/non-blocking-async-execution-with-sse-and-adaptive-polling]]

## Source

- Task 002 20260817T023357Z Ebc176c5.json
