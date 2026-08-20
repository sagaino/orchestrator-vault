---
title: "Multi-Tier Modular Electron-React Architecture with Typed IPC Contracts"
type: pattern
tags: [pattern, frontend, architecture, electron, vite, react, ipc]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127443075-f8fd0716
sources: ["Harvest 1787127443075 F8fd0716.json"]
---

# Multi-Tier Modular Electron-React Architecture with Typed IPC Contracts

Multi-tier Electron-React architecture with strictly isolated processes and typed shared IPC contracts.

## 1. Overview & Architecture

Arsitektur desktop modular berbasis Electron Vite yang memisahkan Main Process, Preload Context Bridge, dan Renderer UI dengan Typed IPC Channel Contracts di shared layer.

## 2. Implementation & Code Structure

src/
├── main/       # Node.js backend logic, window lifecycle, OS integration
│   ├── handlers/
│   ├── services/
│   └── ipc/
├── preload/    # Secure Context Bridge exposing typed APIs
│   └── modules/
├── shared/     # Universal contracts, channels, schemas, and DTOs
└── renderer/   # Pure React/Vite web application
    └── src/

## 3. Key Implementation Points

- Pemisahan domain yang ketat antara Main, Preload, Renderer, dan Shared.
- Penggunaan channel enums/constants terpusat di `src/shared` untuk mencegah typo IPC string.
- Preload modular memecah API per domain sebelum digabung di `contextBridge.exposeInMainWorld`.

## 4. Code Examples

### Strict boundary IPC exposure through contextBridge using shared channel identifiers

```typescript
// src/shared/auth/channels.ts
export const AUTH_IPC_CHANNELS = {
  LOGIN: 'auth:login',
} as const;

// src/preload/index.ts
import { contextBridge } from 'electron';
import { authApi } from './modules/auth';

const api = {
  auth: authApi,
  // ... other domain modules
};

if (process.contextIsolated) {
  contextBridge.exposeInMainWorld('api', api);
}
```

## 5. Considerations & Best Practices

- Context isolation harus selalu aktif (`contextIsolation: true`) untuk mencegah eksekusi Node runtime langsung di renderer.
- Shared directory tidak boleh mengimpor package khusus Node atau browser agar kompatibel di semua target.

## 6. Related Knowledge

- [[01-Knowledge/patterns/frontend/project-skeleton-template.md]]
- `Electron Security Best Practices`

## 7. Source

- Harvest 1787127443075 F8fd0716.json
