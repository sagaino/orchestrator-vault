---
title: "secure-dynamic-header-injection-request-interception-pipeline"
type: pattern
tags: [pattern, frontend, security, http-interceptor, jwt, headers, axios]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787132260416-42b894f9
sources: ["Harvest 1787132260416 42b894f9.json"]
---

# secure-dynamic-header-injection-request-interception-pipeline

Secure Dynamic Header Injection & Context-Aware Request Interception Pipeline

## 1. Overview & Architecture

Pola request interceptor pipeline yang menginjeksi kredensial otentikasi (Bearer token) dan metadata kriptografi (HMAC/SHA-1 signature, timestamp, identifier) secara transparan ke seluruh outbound HTTP request tanpa membebani domain service layer.

## 2. Implementation & Code Structure

src/config/axios.ts (request interceptor) -> src/hooks/useLocalStorage.ts (getData with AES decryption) -> src/config/signature.ts (makeSignature)

## 3. Key Implementation Points

- Transparent Bearer JWT injection from encrypted client-side storage
- Per-request dynamic timestamping for replay-attack defense window
- Unified header attachment pipeline (sig, email, timestamp, Authorization)
- Seamless support for JSON body and FormData binary payloads

## 4. Code Examples

### Request Interceptor injecting Bearer JWT, dynamic cryptographic signature, timestamp, and metadata headers

```typescript
// Add a request interceptor to add auth token and signature
axios.interceptors.request.use(
  async (config) => {
    const token = getData(LOCALSTORAGE_KEY.TOKEN);
    const timestamp = getTimestamp();

    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    const signature = makeSignature(
      0,
      timestamp,
      config.data || {},
      config.data instanceof FormData
    );

    // Add signature and timestamp to headers
    config.headers["sig"] = signature;
    config.headers["email"] = "0";
    config.headers["timestamp"] = timestamp.toString();

    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);
```

## 5. Considerations & Best Practices

- Decryption on the fly: Token retrieved using encrypted storage helper (getData) seamlessly decrypts AES cipher text before setting Authorization header.
- Zero pollution in service layer: Individual API services only define endpoint and payload; headers, token, and cryptographic signatures are injected transparently.
- Dynamic timestamp evaluation: Timestamp generated per-request rather than at client instantiation time, preventing replay window expiration.

## 6. Related Knowledge

- Jwt Authentication
- Http Pipeline

## 7. Source

- Harvest 1787132260416 42b894f9.json
