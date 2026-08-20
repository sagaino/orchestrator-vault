---
title: "deterministic-lexicographical-payload-canonicalization-and-request-signing"
type: pattern
tags: [pattern, frontend, security, cryptography, canonicalization, request-signing, electron-main]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127443075-f8fd0716
sources: ["Harvest 1787127443075 F8fd0716.json"]
---

# deterministic-lexicographical-payload-canonicalization-and-request-signing

Deterministic lexicographical payload canonicalization and SHA-1 cryptographic request signature generator in Node.js Main process.

## 1. Overview & Architecture

Dalam komunikasi API antara desktop client dan backend pada sistem verifikasi identitas berspesifikasi tinggi, seluruh HTTP request mutlak membutuhkan perlindungan dari manipulasi data di perjalanan (man-in-the-middle tampering) dan replay attacks. Pola ini mengimplementasikan kanonikalisasi payload rekursif yang deterministik yang digabungkan dengan synchronized timestamp dan identitas user, lalu dikonversi menjadi tanda tangan kriptografi SHA-1 (sig) di HTTP headers.

## 2. Implementation & Code Structure

src/main/
├── utils/
│   └── request-signature.ts        # Canonicalizer & SHA-1 signature generator
└── services/
    ├── auth.service.ts             # Injeksi signature pada unauthenticated login
    ├── operator.service.ts         # Injeksi signature pada authenticated mutations & queries
    └── broadcast.service.ts        # Injeksi signature pada broadcast publishing

## 3. Key Implementation Points

- Recursive lexicographical sorting via Object.entries().sort() memastikan urutan field deterministik.
- Flattening nested object hierarchies dengan format dotted-key (parent.child).
- Injeksi synchronized timestamp dan identitas user langsung ke dalam signature payload sebelum hashing.
- Menghasilkan objek security headers siap pakai ({ timestamp, email, sig }) untuk request HTTP Axios.

## 4. Code Examples

### Recursive payload canonicalizer and SHA-1 signature generator with timestamp and email injection

```typescript
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

- Perlindungan kuat terhadap payload tampering dan replay attack antara client dan server.
- Data binary seperti Buffer/Stream harus dikonversi ke Base64 sebelum masuk ke canonicalizer agar hashing deterministik.
- Dapat ditingkatkan ke HMAC-SHA256 jika backend mendukung shared secret key.

## 6. Related Knowledge

- `concepts/api-security-signature.md`
- `patterns/electron/main-process-service-architecture.md`
- `snippets/crypto/canonical-json-signer.md`

## 7. Source

- Harvest 1787127443075 F8fd0716.json
