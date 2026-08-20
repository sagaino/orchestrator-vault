---
title: "Embedded Native Plugin Bridge & Liveness Verification"
type: concept
tags: [concept, orchestrator-promotion]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787116045805-ce1291de
sources: ["Harvest 1787116045805 Ce1291de.json"]
---

# Embedded Native Plugin Bridge & Liveness Verification

## Overview

Pola integrasi plugin lokal platform channel untuk integrasi native camera dan on-device face recognition SDK.

## Purpose

Memungkinkan aplikasi Flutter memanfaatkan library native performa tinggi (C++/Kotlin/Obj-C) untuk pengenalan wajah secara offline/on-device.

## Considerations

- Plugin lokal membutuhkan konfigurasi permission kamera yang ketat di AndroidManifest.xml dan Info.plist.
- Stream frame kamera berpotensi menimbulkan memory leak jika tidak di-dispose saat perpindahan route.

## Related Knowledge

- No related page identified yet.

## Source

- Harvest 1787116045805 Ce1291de.json
