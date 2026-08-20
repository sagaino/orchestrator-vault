---
title: "Automated Request Canonicalization & Cryptographic Signature Pipeline"
type: pattern
tags: [pattern, frontend, security, cryptography, api-signing, electron-main]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787112360086-3c0f5be1
sources: ["Harvest 1787112360086 3c0f5be1.json"]
---

# Automated Request Canonicalization & Cryptographic Signature Pipeline

Outbound request signature pipeline that deterministically flattens and hashes payloads with timestamps to prevent API tampering and replay attacks.

## 1. Overview & Architecture

Deterministic payload canonicalization and cryptographic hash generation executed in the Electron main process prior to dispatching outbound HTTP requests.

## 2. Implementation & Code Structure

src/main/
├── utils/
│   └── request-signature.ts # Canonicalization & signature generator
└── services/
    └── *.service.ts        # Injects timestamp, email, and sig into Axios request headers

## 3. Key Implementation Points

- Recursive alphabetical key sorting (`sort(([a], [b]) => a.localeCompare(b))`).
- Deterministic string flattening handling nested objects, arrays, and primitive values.
- SHA-1 digest computation over concatenated keys with synchronous timestamp and identity metadata.
- Centralized injection into Axios headers via `buildRequestSecurityHeaders`.

## 4. Code Examples

### Deterministic request payload canonicalization and SHA-1 signature generation for API security

```typescript
// src/main/utils/request-signature.ts
import { createHash } from 'node:crypto'

const mapBodyToSignatureString = (body: unknown, parentKey = ''): string => {
  if (!body || typeof body !== 'object' || Array.isArray(body)) return ''

  const entries = Object.entries(body as Record<string, unknown>)
    .sort(([a], [b]) => a.localeCompare(b))

  let keys = ''

  for (const [key, value] of entries) {
    const finalKey = parentKey ? `${parentKey}.${key}` : key

    if (Array.isArray(value)) {
      keys += finalKey
      continue
    }

    if (value && typeof value === 'object') {
      keys += mapBodyToSignatureString(value, finalKey)
      continue
    }

    keys += `${finalKey}${String(value ?? '')}`
  }

  return keys
}

export const buildRequestSecurityHeaders = (
  payload: Record<string, unknown>,
  email = '0'
): Record<'timestamp' | 'email' | 'sig', string> => {
  const timestamp = Date.now()
  const signaturePayload = {
    ...payload,
    timestamp,
    email: payload.email ?? email,
  }
  const keys = mapBodyToSignatureString(signaturePayload)
  const sig = createHash('sha1').update(keys).digest('hex')

  return {
    timestamp: String(timestamp),
    email,
    sig,
  }
}
```

## 5. Considerations & Best Practices

- Keys must be strictly sorted alphabetically at all nesting levels to maintain deterministic hashing across client and backend.
- Array values append key names without indexing; nested objects are traversed recursively.
- Timestamp header prevents replay attacks within the backend validity window.

## 6. Related Knowledge

- Api Request Signing
- Hmac Canonicalization

## 7. Source

- Harvest 1787112360086 3c0f5be1.json
