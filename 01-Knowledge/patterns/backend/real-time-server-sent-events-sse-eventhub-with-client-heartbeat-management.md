---
title: "Real-time Server-Sent Events (SSE) EventHub with Client Heartbeat Management"
type: pattern
tags: [pattern, backend, sse, realtime, event-hub, streaming]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787130041232-c9802673
sources: ["Harvest 1787130041232 C9802673.json"]
---

# Real-time Server-Sent Events (SSE) EventHub with Client Heartbeat Management

Implementasi Server-Sent Events (SSE) EventHub zero-dependency dengan keepalive heartbeat otomatis, dynamic subscriber management, dan memory leak prevention.

## 1. Overview & Architecture

Pola komunikasi asinkron real-time Server-Sent Events (SSE) yang ringan dan zero-dependency, dilengkapi pemeliharaan koneksi via periodic heartbeat dan pengelolaan siklus hidup subscriber yang efisien.

## 2. Implementation & Code Structure

src/
├── api/
│   ├── events.mjs       # SSE event hub, client tracking, and heartbeat timer
│   └── router.mjs       # SSE endpoint registration and event broadcasting
└── orchestrator.mjs     # Lifecycle event emission triggering SSE broadcasts

## 3. Key Implementation Points

- Bangun streaming SSE menggunakan Node.js native `http` response tanpa library eksternal.
- Kirim heartbeat berkala (komentar SSE `: heartbeat`) untuk menjaga koneksi TCP tetap hidup melewati NAT/proxy.
- Kelola lifecycle subscriber secara otomatis saat socket terputus (`req.on("close")`).
- Sediakan fungsi broadcast terpadu untuk mendistribusikan event lifecycle orchestrator ke semua client terhubung.

## 4. Code Examples

### Resilient SSE event hub with automatic unref'd heartbeat and client disconnection handling

```javascript
export function createEventHub({ heartbeatIntervalMs = 15000 } = {}) {
  const clients = new Set();
  let heartbeatTimer = null;

  const startHeartbeat = () => {
    if (heartbeatTimer) return;
    heartbeatTimer = setInterval(() => {
      for (const client of clients) {
        try {
          client.res.write(`: heartbeat ${new Date().toISOString()}\n\n`);
        } catch {
          clients.delete(client);
        }
      }
    }, heartbeatIntervalMs);
    if (heartbeatTimer.unref) heartbeatTimer.unref();
  };

  const addClient = (req, res) => {
    res.writeHead(200, {
      "Content-Type": "text/event-stream",
      "Cache-Control": "no-cache, no-transform",
      "Connection": "keep-alive",
      "Access-Control-Allow-Origin": "*",
      "X-Accel-Buffering": "no",
    });

    const client = { req, res, connectedAt: new Date().toISOString() };
    clients.add(client);
    res.write(`event: connected\ndata: ${JSON.stringify({ connectedAt: client.connectedAt })}\n\n`);

    if (clients.size === 1) startHeartbeat();

    req.on("close", () => {
      clients.delete(client);
      if (clients.size === 0 && heartbeatTimer) {
        clearInterval(heartbeatTimer);
        heartbeatTimer = null;
      }
    });

    return client;
  };

  const broadcast = (eventType, payload = {}) => {
    const data = typeof payload === "string" ? payload : JSON.stringify(payload);
    const message = `event: ${eventType}\ndata: ${data}\n\n`;
    for (const client of clients) {
      try {
        client.res.write(message);
      } catch {
        clients.delete(client);
      }
    }
  };

  return { addClient, broadcast, getClientCount: () => clients.size };
}
```

## 5. Considerations & Best Practices

- Gunakan `heartbeatTimer.unref()` agar timer SSE tidak menahan event loop Node.js saat proses hendak shutdown.
- Header `X-Accel-Buffering: no` dan `Cache-Control: no-transform` esensial untuk mencegah proxy/reverse-proxy menahan stream buffer.
- Dengarkan event `req.on("close")` untuk segera menghapus client mati dan membersihkan resource memory.

## 6. Related Knowledge

- Realtime Sse
- Event Streaming

## 7. Source

- Harvest 1787130041232 C9802673.json
