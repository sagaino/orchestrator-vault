---
title: "Type-Safe 3-Tier Desktop IPC Architecture"
type: pattern
tags: [pattern, frontend, electron, ipc, typescript, architecture, context-bridge]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787111316881-96cf68d4
sources: ["Harvest 1787111316881 96cf68d4.json"]
---

# Type-Safe 3-Tier Desktop IPC Architecture

Type-safe modular IPC communication bridging Renderer, Preload, and Main processes with shared channel definitions and modular handler registries.

## 1. Overview & Architecture

A robust architectural pattern for Electron desktop applications that establishes type safety and security boundaries across Main, Preload, and Renderer processes using shared channel constants and domain-specific DTOs.

## 2. Implementation & Code Structure

src/
├── shared/           # Shared types, Zod schemas, and channel constants (e.g. auth/channels.ts)
├── main/
│   ├── ipc/          # Main process IPC channel listener registration / unregistration
│   ├── handlers/     # Request validation and forwarding to services
│   └── services/     # Pure backend business logic and HTTP calls
├── preload/
│   ├── modules/      # Modular IPC wrappers exposing typed APIs via contextBridge
│   └── index.ts      # Exposes window.api to renderer
└── renderer/src/
    └── services/     # Frontend client wrapper handling auth tokens & response mapping

## 3. Key Implementation Points

- Centralize IPC channel names as constant objects with 'as const' in the shared directory.
- Decouple IPC handler registration (main/ipc) from request execution logic (main/handlers and main/services).
- Group contextBridge APIs into feature-specific modules (authModule, deviceModule, operatorModule) mounted under window.api.
- Enforce strict TypeScript return types on all invoke calls in the preload layer.

## 4. Code Examples

### End-to-end typed contract linking Shared Channels, Main IPC handler, and Preload Context Bridge

```typescript
// src/shared/auth/channels.ts
export const AUTH_IPC_CHANNELS = {
  LOGIN: 'auth:login',
} as const;

// src/main/ipc/auth.ipc.ts
import { ipcMain } from 'electron'
import { AuthHandler } from '../handlers/auth.handler'
import { AUTH_IPC_CHANNELS } from 'src/shared/auth/channels'

export function registerAuthIpcHandlers(): void {
  ipcMain.handle(AUTH_IPC_CHANNELS.LOGIN, AuthHandler.login)
}

export function unregisterAuthIpcHandlers(): void {
  ipcMain.removeHandler(AUTH_IPC_CHANNELS.LOGIN)
}

// src/preload/modules/auth.ts
import { ipcRenderer } from 'electron'
import { AUTH_IPC_CHANNELS } from 'src/shared/auth/channels'
import type { LoginFormData } from 'src/shared/auth/schema'
import type { LoginResponse } from 'src/shared/auth/types'

export const authModule = {
  login: (data: LoginFormData): Promise<LoginResponse> =>
    ipcRenderer.invoke(AUTH_IPC_CHANNELS.LOGIN, data),
}
```

## 5. Considerations & Best Practices

- Avoid putting business logic or credential storage directly inside preload scripts; preload should strictly act as a secure boundary.
- Always provide unregister handlers for dynamic windows or hot-reloading scenarios to prevent memory leaks and duplicate invocation errors.
- Types must be shared across processes without importing Node.js or Electron dependencies into the renderer bundle.

## 6. Related Knowledge

- `Electron Context Isolation`
- `TypeScript as const Assertions`
- `Clean Architecture Layering`

## 7. Source

- Harvest 1787111316881 96cf68d4.json
