---
title: "bidirectional-native-websocket-bridge-with-ipc-multiplexing"
type: pattern
tags: [pattern, frontend, websocket, device-bridge, ipc, realtime, telemetry, electron]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127443075-f8fd0716
sources: ["Harvest 1787127443075 F8fd0716.json"]
---

# bidirectional-native-websocket-bridge-with-ipc-multiplexing

Main-process native WebSocket lifecycle manager with event-driven IPC channel multiplexing for real-time device telemetry.

## 1. Overview & Architecture

A robust bi-directional communication bridge pattern that runs WebSocket socket management inside Electron's Node.js main process and multiplexes status and data streams to the renderer process through clean, subscribable IPC channels.

## 2. Implementation & Code Structure

- src/shared/ws/channels.ts (WS_IPC_CHANNELS constant mappings)
- src/main/services/ws.service.ts (Node ws socket singleton, lifecycle, event forwarding)
- src/main/handlers/ws.handler.ts (IPC invocations unpacker & error guard)
- src/main/ipc/ws.ipc.ts (register/unregister IPC handlers)
- src/preload/modules/ws.ts (Subscribable listener closures returning unbind functions)

## 3. Key Implementation Points

- Main-process WebSocket management utilizing native 'ws' package with direct access to local network hardware and telemetry feeds.
- Event forwarding via WebContents.send to stream real-time sensor and device events without polluting renderer security scope.
- Teardown guarantees via app.on('before-quit') to unregister handlers and sever open connections cleanly.

## 4. Code Examples

### Singleton Node.js WebSocket service in main process managing socket lifecycle and WebContents event emission

```typescript
// src/main/services/ws.service.ts
import type { WebContents } from 'electron'
import { WebSocket, type RawData } from 'ws'
import { WS_IPC_CHANNELS } from 'src/shared/ws/channels'

let ws: WebSocket | null = null

export const WsService = {
  connect: (sender: WebContents, url: string): void => {
    if (ws) {
      ws.terminate()
      ws = null
    }

    ws = new WebSocket(url)

    ws.on('open', () => sender.send(WS_IPC_CHANNELS.STATUS, 'connected'))
    ws.on('close', () => sender.send(WS_IPC_CHANNELS.STATUS, 'disconnected'))
    ws.on('error', () => sender.send(WS_IPC_CHANNELS.STATUS, 'error'))
    ws.on('message', (data: RawData) => {
      sender.send(WS_IPC_CHANNELS.MESSAGE, data.toString())
    })
  },

  send: (message: string): void => {
    ws?.send(message)
  },

  disconnect: (): void => {
    ws?.terminate()
    ws = null
  }
}
```

### Preload module exposing typed socket commands and subscription listeners with automatic cleanup closures

```typescript
// src/preload/modules/ws.ts
import { ipcRenderer } from 'electron'
import { WS_IPC_CHANNELS } from 'src/shared/ws/channels'

export const wsApi = {
  wsConnect: (url: string): Promise<void> => ipcRenderer.invoke(WS_IPC_CHANNELS.CONNECT, url),
  wsSend: (message: string): Promise<void> => ipcRenderer.invoke(WS_IPC_CHANNELS.SEND, message),
  wsDisconnect: (): Promise<void> => ipcRenderer.invoke(WS_IPC_CHANNELS.DISCONNECT),
  onWsStatus: (cb: (status: string) => void): (() => void) => {
    const listener = (_event: unknown, status: string) => cb(status)
    ipcRenderer.on(WS_IPC_CHANNELS.STATUS, listener)
    return () => ipcRenderer.removeListener(WS_IPC_CHANNELS.STATUS, listener)
  },
  onWsMessage: (cb: (message: string) => void): (() => void) => {
    const listener = (_event: unknown, message: string) => cb(message)
    ipcRenderer.on(WS_IPC_CHANNELS.MESSAGE, listener)
    return () => ipcRenderer.removeListener(WS_IPC_CHANNELS.MESSAGE, listener)
  }
}
```

### IPC handler registration with lifecycle teardown on application shutdown (app.on('before-quit'))

```typescript
// src/main/ipc/ws.ipc.ts
import { ipcMain } from 'electron'
import { WS_IPC_CHANNELS } from 'src/shared/ws/channels'
import { WsHandler } from '../handlers/ws.handler'
import { WsService } from '../services/ws.service'

export function registerWsIpcHandlers(): void {
  ipcMain.handle(WS_IPC_CHANNELS.CONNECT, WsHandler.connect)
  ipcMain.handle(WS_IPC_CHANNELS.SEND, WsHandler.send)
  ipcMain.handle(WS_IPC_CHANNELS.DISCONNECT, WsHandler.disconnect)
}

export function unregisterWsIpcHandlers(): void {
  ipcMain.removeHandler(WS_IPC_CHANNELS.CONNECT)
  ipcMain.removeHandler(WS_IPC_CHANNELS.SEND)
  ipcMain.removeHandler(WS_IPC_CHANNELS.DISCONNECT)
  WsService.disconnect()
}
```

## 5. Considerations & Best Practices

- WebSockets in the Main process bypass browser CORS, mixed-content, and renderer thread UI blocking.
- Calling ws.terminate() instead of ws.close() guarantees immediate socket teardown when reconnecting or closing.
- Listener subscription functions in Preload return cleanup callbacks to prevent memory leaks in React useEffect.

## 6. Related Knowledge

- `Electron IPC Multiplexing (ipcMain.handle / webContents.send / ipcRenderer.on)`
- `Node.js WebSocket ('ws') Architecture`
- `Observer Teardown Pattern in React / ContextBridge`

## 7. Source

- Harvest 1787127443075 F8fd0716.json
