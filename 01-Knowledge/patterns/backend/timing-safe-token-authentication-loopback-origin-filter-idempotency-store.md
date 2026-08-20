---
title: "Timing-Safe Token Authentication, Loopback Origin Filter & Idempotency Store"
type: pattern
tags: [pattern, backend, security, authentication, idempotency, timing-safe]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787130041232-c9802673
sources: ["Harvest 1787130041232 C9802673.json"]
---

# Timing-Safe Token Authentication, Loopback Origin Filter & Idempotency Store

Arsitektur keamanan API lokal zero-dependency menggunakan constant-time crypto token check, loopback origin filter, dan sliding-window TTL idempotency store.

## 1. Overview & Architecture

Pola keamanan API lokal zero-dependency yang memadukan verifikasi token waktu-konstan (constant-time verification), pembatasan origin loopback lokal, dan store idempotensi berbasis TTL.

## 2. Implementation & Code Structure

src/
└── api/
    ├── auth.mjs         # Authentication, origin check, and idempotency store
    ├── router.mjs       # Middleware integration and route dispatching
    └── server.mjs       # HTTP server instantiation with security layer

## 3. Key Implementation Points

- Gunakan `crypto.timingSafeEqual` untuk mencegah serangan timing side-channel saat memverifikasi API token.
- Batasi akses origin hanya ke host loopback lokal (`127.0.0.1`, `localhost`, `[::1]`).
- Dukung idempotency store dengan TTL dan persistensi disk untuk mencegah eksekusi ganda pada operasi berulang.
- Simpan token lokal dengan permission ketat `0o600`.

## 4. Code Examples

### Constant-time token validation preventing timing side-channel attacks

```javascript
import { timingSafeEqual, randomBytes } from "node:crypto";

export function authenticateRequest(req, expectedToken) {
  if (!expectedToken || typeof expectedToken !== "string") {
    return { authenticated: false, reason: "Server API token not configured" };
  }

  const extracted = extractTokenFromRequest(req);
  if (!extracted) {
    return { authenticated: false, reason: "Missing Authorization header or token" };
  }

  try {
    const expectedBuffer = Buffer.from(expectedToken, "utf8");
    const actualBuffer = Buffer.from(extracted, "utf8");
    if (expectedBuffer.length !== actualBuffer.length) {
      return { authenticated: false, reason: "Invalid API token" };
    }
    const matches = timingSafeEqual(expectedBuffer, actualBuffer);
    return matches
      ? { authenticated: true, reason: null }
      : { authenticated: false, reason: "Invalid API token" };
  } catch {
    return { authenticated: false, reason: "Token validation error" };
  }
}
```

### Loopback origin guard preventing unauthorized cross-origin requests

```javascript
export function validateOrigin(req) {
  const origin = req.headers["origin"] || req.headers["referer"];
  if (!origin || typeof origin !== "string") {
    return { valid: true, reason: null };
  }

  try {
    const parsed = new URL(origin);
    const hostname = parsed.hostname.toLowerCase();
    if (hostname === "127.0.0.1" || hostname === "localhost" || hostname === "[::1]") {
      return { valid: true, reason: null };
    }
    return { valid: false, reason: `Origin ${parsed.origin} is not allowed. Only localhost/127.0.0.1 is permitted.` };
  } catch {
    return { valid: false, reason: "Malformed Origin or Referer header" };
  }
}
```

## 5. Considerations & Best Practices

- Length check must precede `timingSafeEqual` to avoid buffer length mismatch errors in Node.js crypto.
- Origin and Referer headers must be strictly restricted to loopback addresses (`127.0.0.1`, `localhost`, `[::1]`) for local API security.
- Idempotency cache should have automated TTL eviction to avoid memory leaks.

## 6. Related Knowledge

- Api Security
- Idempotent Api Design

## 7. Source

- Harvest 1787130041232 C9802673.json
