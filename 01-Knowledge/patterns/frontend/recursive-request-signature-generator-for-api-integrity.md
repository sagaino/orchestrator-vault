---
title: "Recursive Request Signature Generator for API Integrity"
type: pattern
tags: [pattern, frontend, security, cryptography, api-signing, axios, node-crypto]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787111316881-96cf68d4
sources: ["Harvest 1787111316881 96cf68d4.json"]
---

# Recursive Request Signature Generator for API Integrity

Deterministic recursive payload sorting and hashing utility for generating tamper-proof API security headers.

## 1. Overview & Architecture

A deterministic payload canonicalization and hashing mechanism executed in the Node.js main process to sign HTTP requests with timestamp and integrity headers, preventing man-in-the-middle payload tampering.

## 2. Implementation & Code Structure

src/main/
├── utils/
│   └── request-signature.ts  # Canonical sorting, flattening, and SHA1 hashing utility
└── services/
    ├── auth.service.ts       # Integrates security headers into axios requests
    └── operator.service.ts   # Signs data and mutations before dispatching to API backend

## 3. Key Implementation Points

- Sort object keys lexicographically at every recursive depth.
- Recursively serialize nested JSON objects into dot-notated key-value strings.
- Append timestamp and identity (e.g. email or client identifier) to the signature payload.
- Digest the resulting canonical string with SHA1 (or HMAC) to generate the sig header.

## 4. Code Examples

### Recursive key sorting, string serialization, and SHA1 hash computation for request payloads

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

- Key sorting is strictly alphabetical (localeCompare); client and backend must use identical normalization rules.
- Array elements are indexed by key name only to prevent ordering anomalies.
- Timestamp drift between client machine and remote server must be accounted for in backend validation windows.

## 6. Related Knowledge

- `Cryptographic Hashes (SHA1/SHA256)`
- `Canonical Request Representation`
- `API Request Signing & HMAC`

## 7. Source

- Harvest 1787111316881 96cf68d4.json
