---
title: "Strict End-to-End Type-Safe Multi-Process IPC Bridge"
type: pattern
tags: [pattern, frontend, electron, ipc, typescript, security, architecture]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787112360086-3c0f5be1
sources: ["Harvest 1787112360086 3c0f5be1.json"]
---

# Strict End-to-End Type-Safe Multi-Process IPC Bridge

End-to-end type-safe multi-process IPC bridging architecture ensuring context isolation and preventing contract drift between Electron main and renderer processes.

## 1. Overview & Architecture

A strict 4-tier IPC architecture separating channel contracts, Electron main handlers, preload context bridges, and UI service invocations with end-to-end TypeScript types.

## 2. Implementation & Code Structure

src/
├── shared/         # Single source of truth for channel constants, request/response DTOs
├── main/
│   ├── ipc/        # ipcMain.handle / removeHandler registrations
│   ├── handlers/   # Payload unwrapping and routing
│   └── services/   # Node-level Axios/HTTP business logic
├── preload/
│   ├── modules/    # Typed invoke wrappers using ipcRenderer.invoke
│   └── index.ts    # contextBridge.exposeInMainWorld('api', api)
└── renderer/
    └── src/services/ # UI service callers invoking window.api

## 3. Key Implementation Points

- Define shared channel constants using `as const` under `src/shared/<module>/channels.ts`.
- Implement modular IPC registration functions (`register*IpcHandlers` & `unregister*IpcHandlers`).
- Bridge methods safely via `contextBridge.exposeInMainWorld` in `src/preload/index.ts` with context isolation checks.
- Call `window.api.<module>.<method>` through typed renderer services.

## 4. Code Examples

### 4-tier contract-driven IPC registration and exposure mechanism in Electron

```typescript
// 1. src/shared/auth/channels.ts
export const AUTH_IPC_CHANNELS = {
  LOGIN: 'auth:login',
} as const

// 2. src/main/ipc/auth.ipc.ts
import { ipcMain } from 'electron'
import { AuthHandler } from '../handlers/auth.handler'
import { AUTH_IPC_CHANNELS } from 'src/shared/auth/channels'

export function registerAuthIpcHandlers(): void {
  ipcMain.handle(AUTH_IPC_CHANNELS.LOGIN, AuthHandler.login)
}

export function unregisterAuthIpcHandlers(): void {
  ipcMain.removeHandler(AUTH_IPC_CHANNELS.LOGIN)
}

// 3. src/preload/index.ts
import { contextBridge } from 'electron'
import { authApi } from './modules/auth'

const api = {
  auth: authApi,
  // other modules...
}

if (process.contextIsolated) {
  contextBridge.exposeInMainWorld('api', api)
}
```

## 5. Considerations & Best Practices

- Ensure all IPC channels are defined with `as const` to prevent runtime typos and preserve TypeScript type narrowing.
- Always clean up IPC handlers with `removeHandler` during window or app lifecycle teardowns to prevent memory leaks and duplicate handler registrations.
- Never expose raw `ipcRenderer` to renderer context; strictly expose explicit typed domain modules.

## 6. Related Knowledge

- Electron Ipc Architecture
- Context Isolation Best Practices

## 7. Source

- Harvest 1787112360086 3c0f5be1.json
