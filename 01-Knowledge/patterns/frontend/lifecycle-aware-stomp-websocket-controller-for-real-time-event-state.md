---
title: "Lifecycle-Aware STOMP WebSocket Controller for Real-Time Event State"
type: pattern
tags: [pattern, frontend, websocket, stomp, real-time, state-management, iot]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787132260416-42b894f9
sources: ["Harvest 1787132260416 42b894f9.json"]
---

# Lifecycle-Aware STOMP WebSocket Controller for Real-Time Event State

Lifecycle-Aware STOMP WebSocket Controller for Real-Time Event State

## 1. Overview & Architecture

Pola Real-Time State Controller berbasis STOMP WebSocket yang dikapsulkan ke dalam custom hook reaktif dengan proteksi kebocoran koneksi, auto-reconnection, dan sinkronisasi event queue.

## 2. Implementation & Code Structure

src/pages/DoorAccess/
├── hooks/
│   └── useCardWebSocket.ts       # STOMP broker subscriber hook
└── types/
    └── api.ts                    # WebSocket message payload type definitions

## 3. Key Implementation Points

- STOMP protocol management with configurable reconnect delay and bidirectional heartbeat intervals.
- Reactive state exposure (`isConnected`, `isConnecting`, `cardData`, `error`).
- Declarative lifecycle connection management driven by UI state.

## 4. Code Examples

### Lifecycle-aware STOMP WebSocket subscriber hook with auto-reconnect and heartbeat.

```typescript
// src/pages/DoorAccess/hooks/useCardWebSocket.ts
import { useEffect, useRef, useState, useCallback } from 'react';
import { Client } from '@stomp/stompjs';
import type { WebSocketCardData } from '../types/api';

export interface UseCardWebSocketReturn {
  isConnected: boolean;
  isConnecting: boolean;
  cardData: WebSocketCardData | null;
  error: string | null;
  resetCardData: () => void;
  disconnect: () => void;
}

export function useCardWebSocket(isEnabled: boolean = false): UseCardWebSocketReturn {
  const [isConnected, setIsConnected] = useState(false);
  const [isConnecting, setIsConnecting] = useState(false);
  const [cardData, setCardData] = useState<WebSocketCardData | null>(null);
  const [error, setError] = useState<string | null>(null);
  const clientRef = useRef<Client | null>(null);

  const resetCardData = useCallback(() => {
    setCardData(null);
    setError(null);
  }, []);

  const disconnect = useCallback(() => {
    if (clientRef.current) {
      clientRef.current.deactivate();
      clientRef.current = null;
    }
    setIsConnected(false);
    setIsConnecting(false);
    setError(null);
  }, []);

  useEffect(() => {
    if (!isEnabled) {
      disconnect();
      return;
    }

    if (clientRef.current) return;

    setIsConnecting(true);
    setError(null);

    const client = new Client({
      brokerURL: `${import.meta.env.VITE_WEBSOCKET_URL}/web-socket`,
      reconnectDelay: 5000,
      heartbeatIncoming: 4000,
      heartbeatOutgoing: 4000,
      onConnect: (frame) => {
        setIsConnected(true);
        setIsConnecting(false);
        client.subscribe('/queue/access-card', (message) => {
          try {
            const data = JSON.parse(message.body) as WebSocketCardData;
            setCardData(data);
          } catch (parseError) {
            setError('Failed to parse card data');
          }
        });
      },
      onStompError: (frame) => {
        setError(frame.headers['message'] || 'WebSocket connection error');
        setIsConnected(false);
      },
      onWebSocketClose: () => {
        setIsConnected(false);
      }
    });

    client.activate();
    clientRef.current = client;

    return () => {
      disconnect();
    };
  }, [isEnabled, disconnect]);

  return { isConnected, isConnecting, cardData, error, resetCardData, disconnect };
}
```

## 5. Considerations & Best Practices

- Condition-driven activation (`isEnabled` flag) prevents idle WebSocket connection leaks when modals or pages are closed.
- `useRef` holds the STOMP `Client` instance across re-renders without triggering unnecessary subscription resets.
- Graceful deactivation in useEffect cleanup ensures socket resources are released during unmount.

## 6. Related Knowledge

- `STOMP Protocol over WebSockets`
- `Real-time Event Streaming in React`
- `Resource Lifecycle in Custom Hooks`

## 7. Source

- Harvest 1787132260416 42b894f9.json
