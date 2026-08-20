---
title: "Non-Blocking Async Execution Endpoints with SSE and Adaptive Polling Backoff"
type: pattern
tags: [pattern, orchestrator-promotion]
created: 2026-08-17
updated: 2026-08-17
orchestrator_run: task-001-20260817T023005Z-5213f004
sources: ["Task 001 20260817T023005Z 5213f004.json"]
---

# Non-Blocking Async Execution Endpoints with SSE and Adaptive Polling Backoff

## Overview

Pola implementasi endpoint eksekusi agent yang merespons secara non-blocking dengan status HTTP 202 Accepted, menjalankan lifecycle agent di background worker pool, menyiarkan status via SSE, serta mengoptimalkan polling daemon menggunakan adaptive backoff (1s saat aktif, 5s saat idle) untuk meminimalkan beban I/O disk lokal.

## Purpose

Pola ini reusable untuk sistem orkestrasi task berbasis agent dan antrean background daemon guna menjaga latency API tetap instan (<50ms) dan mengurangi beban disk polling yang berlebihan.

## Considerations

- Pastikan error synchronous (seperti validasi missing reason atau run tidak ditemukan) diverifikasi sebelum HTTP 202 dikirimkan.
- Pastikan event stream SSE memiliki error event handler agar client menerima notifikasi jika eksekusi background gagal.
- Daemon job poller harus segera di-wake up saat ada event task ready yang masuk tanpa menunggu interval idle selesai.

## Related Knowledge

- [[01-Knowledge/concepts/architecture/api-service-data-flow]]

## Source

- Task 001 20260817T023005Z 5213f004.json
