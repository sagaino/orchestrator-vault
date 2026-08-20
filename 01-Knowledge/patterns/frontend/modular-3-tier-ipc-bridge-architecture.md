---
title: "Modular 3-Tier IPC Bridge Architecture"
type: pattern
tags: [pattern, frontend, electron, ipc, architecture, modularity, typescript]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787109583565-5ff8f29f
sources: ["Harvest 1787109583565 5ff8f29f.json"]
---

# Modular 3-Tier IPC Bridge Architecture

Modular 3-Tier IPC Bridge Architecture enforcing type safety and boundary isolation in Electron applications.

## 1. Overview & Architecture

A 3-tier architecture ensuring strict Electron security boundaries, type safety, and modularity between Main, Preload, and Renderer processes.

## 2. Implementation & Code Structure

src/shared/<module>/{channels,types,schema}.ts -> src/preload/modules/<module>.ts -> src/main/ipc/<module>.ipc.ts -> src/main/handlers/<module>.handler.ts

## 3. Key Implementation Points

- Single source of truth for IPC channel strings and payload types under src/shared/
- Modular preload modules exposed via contextBridge.exposeInMainWorld
- Decoupled handler and service logic in the Main process

## 4. Code Examples

### Modular separation of IPC definitions across Shared, Preload, and Main layers

```typescript
// 1. Channel Constants (src/shared/auth/channels.ts)
export const AUTH_IPC_CHANNELS = {
  LOGIN: 'auth:login',
} as const;

// 2. Preload Exposure (src/preload/modules/auth.ts)
export const authApi = {
  login: (payload: LoginFormData): Promise<LoginResponse> =>
    ipcRenderer.invoke(AUTH_IPC_CHANNELS.LOGIN, payload),
};

// 3. Main IPC Registry (src/main/ipc/auth.ipc.ts)
export function registerAuthIpcHandlers(): void {
  ipcMain.handle(AUTH_IPC_CHANNELS.LOGIN, AuthHandler.login);
}
```

## 5. Considerations & Best Practices

- Maintains strict context isolation by preventing direct Node.js API access from the Renderer.
- Eliminates magic strings through shared channel constant definitions.
- Pairs handler registration with unregistration to avoid IPC listener memory leaks.

## 6. Related Knowledge

- Electron Ipc Security Bridge
- `Electron Context Isolation Best Practices`

## 7. Source

- Harvest 1787109583565 5ff8f29f.json
