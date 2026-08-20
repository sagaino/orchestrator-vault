---
title: "Main-Process WebSocket Proxy for Native Desktop Event Streaming"
type: pattern
tags: [pattern, frontend, electron, websocket, networking, ipc, streaming]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787111316881-96cf68d4
sources: ["Harvest 1787111316881 96cf68d4.json"]
---

# Main-Process WebSocket Proxy for Native Desktop Event Streaming

Main-process WebSocket proxy managing socket connections and event forwarding to Electron WebContents.

## 1. Overview & Architecture

A desktop networking pattern that encapsulates WebSocket lifecycle management inside Electron's main process, routing streaming events and binary data to the UI renderer via typed IPC events.

## 2. Implementation & Code Structure

src/
├── shared/ws/
│   └── channels.ts       # Status, Message, Connect, and Disconnect IPC channel constants
├── main/
│   ├── services/
│   │   └── ws.service.ts # Pure Node.js ws instance management
│   └── handlers/
│       └── ws.handler.ts # Listens to renderer connect/disconnect/send commands
└── preload/modules/
    └── ws.ts             # Exposes onMessage, onStatusChange, connect, and send to renderer

## 3. Key Implementation Points

- Maintain single active WebSocket reference in main process service singleton.
- Forward WebSocket lifecycle events ('open', 'close', 'error', 'message') directly to sender WebContents.
- Provide explicit terminate() and clean teardown methods to handle network switches or user logouts.

## 4. Code Examples

### Main process WebSocket manager forwarding status and messages to Electron WebContents

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

## 5. Considerations & Best Practices

- Using Node.js 'ws' in the main process bypasses browser-level WebSocket restrictions (such as self-signed certs or local IP CORS).
- Ensure the previous WebSocket instance is explicitly terminated before initializing a new connection to prevent dangling connections.
- Large binary payloads can be compressed (e.g. pako) or forwarded as ArrayBuffers.

## 6. Related Knowledge

- `Node.js ws Library`
- `Electron WebContents IPC Eventing`
- `Binary Data Streaming & Pako Compression`

## 7. Source

- Harvest 1787111316881 96cf68d4.json
