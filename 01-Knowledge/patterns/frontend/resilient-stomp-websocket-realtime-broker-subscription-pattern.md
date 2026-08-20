---
title: "Resilient STOMP/WebSocket Realtime Broker Subscription Pattern"
type: pattern
tags: [pattern, frontend, websocket, stomp, realtime, hooks, state-management]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787111639165-e1d30e37
sources: ["Harvest 1787111639165 E1d30e37.json"]
---

# Resilient STOMP/WebSocket Realtime Broker Subscription Pattern

Resilient STOMP over WebSocket subscriber hook providing fault-tolerant real-time event streaming and state management.

## 1. Overview & Architecture

A production-grade React Hook abstraction for STOMP/WebSocket pub-sub channels featuring lifecycle management, exponential or timed auto-reconnection, heartbeat monitoring, and reactive UI updates.

## 2. Implementation & Code Structure

src/
└── pages/
    ├── Alert/
    │   ├── hooks/
    │   │   └── useStompNotifications.ts # Realtime notification STOMP subscriber hook
    │   └── types/
    │       └── NotificationResponse.ts  # Notification payload typing
    └── PersonAccess/
        └── hooks/
            └── useCardWebSocket.ts      # Card tap WebSocket subscriber

## 3. Key Implementation Points

- Maintains explicit connection states ('connecting' | 'connected' | 'disconnected' | 'error').
- Employs resilient auto-reconnect fallback on abnormal socket closures or broker errors.
- Uses ref synchronization for event callbacks to avoid unnecessary effect re-executions.

## 4. Code Examples

### Custom React Hook encapsulating STOMP over WebSocket subscription, auto-reconnection, and message state management.

```typescript
import { Client } from "@stomp/stompjs";
import { useCallback, useEffect, useRef, useState } from "react";
import type { INotificationValue } from "../types/NotificationResponse";

interface UseStompNotificationsProps {
  onNewNotification?: () => void;
}

type ConnectionStatus = "connecting" | "connected" | "disconnected" | "error";

export const useStompNotifications = ({
  onNewNotification,
}: UseStompNotificationsProps = {}) => {
  const [notifications, setNotifications] = useState<INotificationValue[]>([]);
  const [connectionStatus, setConnectionStatus] = useState<ConnectionStatus>("disconnected");

  const clientRef = useRef<Client | null>(null);
  const reconnectTimeoutRef = useRef<ReturnType<typeof setTimeout> | undefined>(undefined);
  const isInitialLoadRef = useRef(true);
  const onNewNotificationRef = useRef(onNewNotification);

  useEffect(() => {
    onNewNotificationRef.current = onNewNotification;
  }, [onNewNotification]);

  useEffect(() => {
    isInitialLoadRef.current = false;
  }, []);

  const WEBSOCKET_URL = import.meta.env.VITE_WEBSOCKET_URL;

  useEffect(() => {
    const connectWebSocket = () => {
      if (clientRef.current && clientRef.current.active) return;
      if (!WEBSOCKET_URL) {
        setConnectionStatus("error");
        return;
      }

      if (reconnectTimeoutRef.current) clearTimeout(reconnectTimeoutRef.current);

      const wsUrl = `${WEBSOCKET_URL}/web-socket`;
      setConnectionStatus("connecting");

      try {
        const client = new Client({
          brokerURL: wsUrl,
          reconnectDelay: 5000,
          heartbeatIncoming: 4000,
          heartbeatOutgoing: 4000,
        });

        client.onConnect = (frame) => {
          setConnectionStatus("connected");
          client.subscribe("/queue/notification", (message) => {
            try {
              const response: INotificationValue = JSON.parse(message.body);
              setNotifications((prev) => [response, ...prev]);
              if (!isInitialLoadRef.current && onNewNotificationRef.current) {
                onNewNotificationRef.current();
              }
            } catch (parseError) {
              console.error("Error parsing notification:", parseError);
            }
          });
        };

        client.onStompError = () => {
          setConnectionStatus("error");
          reconnectTimeoutRef.current = setTimeout(connectWebSocket, 5000);
        };

        client.onWebSocketClose = (event) => {
          setConnectionStatus("disconnected");
          if (event.code !== 1000) {
            reconnectTimeoutRef.current = setTimeout(connectWebSocket, 5000);
          }
        };

        clientRef.current = client;
        client.activate();
      } catch (error) {
        setConnectionStatus("error");
        reconnectTimeoutRef.current = setTimeout(connectWebSocket, 5000);
      }
    };

    connectWebSocket();

    return () => {
      if (reconnectTimeoutRef.current) clearTimeout(reconnectTimeoutRef.current);
      if (clientRef.current) {
        clientRef.current.deactivate();
        clientRef.current = null;
      }
    };
  }, [WEBSOCKET_URL]);

  const handleMarkAllRead = useCallback(() => {
    setNotifications((prev) => prev.map((n) => ({ ...n, is_read: true })));
  }, []);

  const clearNotifications = useCallback(() => {
    setNotifications([]);
  }, []);

  return { notifications, connectionStatus, handleMarkAllRead, clearNotifications };
};
```

## 5. Considerations & Best Practices

- Always clean up STOMP clients and timeouts in the React effect cleanup phase to prevent memory leaks and duplicate sockets.
- Use mutable refs for incoming event callbacks to prevent re-subscribing on each render cycle.
- Set up bidirectional heartbeats (heartbeatIncoming & heartbeatOutgoing) to detect dead TCP connections behind reverse proxies.

## 6. Related Knowledge

- Stomp Websocket Protocol
- React Realtime Hook

## 7. Source

- Harvest 1787111639165 E1d30e37.json
