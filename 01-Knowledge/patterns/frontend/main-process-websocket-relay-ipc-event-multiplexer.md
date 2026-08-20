---
title: "Main-Process WebSocket Relay & IPC Event Multiplexer"
type: pattern
tags: [pattern, frontend, electron, websocket, realtime, ipc]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787112360086-3c0f5be1
sources: ["Harvest 1787112360086 3c0f5be1.json"]
---

# Main-Process WebSocket Relay & IPC Event Multiplexer

Main-process hosted WebSocket relay pattern bridging socket network events to renderer views through typed IPC channels with subscription cleanup.

## 1. Overview & Architecture

A persistent WebSocket connection maintained in Electron's Node.js main process that relays real-time messages and device events to renderer windows via IPC.

## 2. Implementation & Code Structure

src/
├── shared/ws/channels.ts      # WebSocket IPC channel contracts
├── main/
│   ├── services/ws.service.ts # Persistent Node.js WebSocket client lifecycle
│   └── handlers/ws.handler.ts # Relays socket frames to renderer via webContents.send
└── preload/modules/ws.ts      # Exposes subscription callback with unlistener closure

## 3. Key Implementation Points

- Node.js WebSocket client managed entirely in Electron main process.
- Dispatches incoming frames to renderer windows via `webContents.send(WS_IPC_CHANNELS.MESSAGE, data)`.
- Preload module provides a clean subscription function returning a cleanup closure.

## 4. Code Examples

### Preload IPC listener and sender bridge for main-process WebSocket events with auto-cleanup subscription

```typescript
// src/preload/modules/ws.ts
import { ipcRenderer } from 'electron'
import { WS_IPC_CHANNELS } from 'src/shared/ws/channels'

export const wsApi = {
  onWsMessage: (callback: (data: unknown) => void) => {
    const subscription = (_event: Electron.IpcRendererEvent, value: unknown) => callback(value)
    ipcRenderer.on(WS_IPC_CHANNELS.MESSAGE, subscription)
    return () => {
      ipcRenderer.removeListener(WS_IPC_CHANNELS.MESSAGE, subscription)
    }
  },
  sendWsMessage: (data: unknown) => {
    ipcRenderer.send(WS_IPC_CHANNELS.SEND, data)
  }
}
```

## 5. Considerations & Best Practices

- Always return an unsubscription cleanup function from listener attachments to avoid dangling listeners on React component unmount.
- Hosting WebSocket connections in the main process keeps the connection alive across page navigations and window reloads.

## 6. Related Knowledge

- Main Process Websocket Relay
- Ipc Event Broadcasting

## 7. Source

- Harvest 1787112360086 3c0f5be1.json
