---
title: "Request Signing Interceptor & Automated Session Invalidation"
type: pattern
tags: [pattern, frontend, security, axios, interceptor, request-signing, authentication]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787112018130-d8cc3a8b
sources: ["Harvest 1787112018130 D8cc3a8b.json"]
---

# Request Signing Interceptor & Automated Session Invalidation

Automated request signing interceptor using SHA-1 signature generation and centralized 401 session expiration handling.

## 1. Overview & Architecture

Pola keamanan HTTP client terpusat di mana setiap outgoing request disuntikkan signature kriptografi (SHA-1 dari flattened sorted payload) dan token Bearer, serta penanganan respon 401 Unauthorized secara global.

## 2. Implementation & Code Structure

src/lib/
├── signature.ts (Payload normalization, flattening, and SHA-1 hashing)
└── axios.ts (Axios client instance, Request Signing Interceptor, Response 401 Interceptor)

## 3. Key Implementation Points

- Lexicographical key sorting and recursive object flattening for deterministic hashing.
- Support for both JSON payloads and multipart FormData payloads in signature creation.
- Automated injection of Bearer token, signature hash, and timestamp headers on all outgoing requests.
- Centralized 401 session expiry interceptor with storage cleanup, SweetAlert2 prompt, and safe redirect.

## 4. Code Examples

### Payload flattening and SHA-1 deterministic cryptographic signature generation for API requests.

```typescript
// Flatten and sort payload parameters for deterministic hashing
function flattenObject(obj: any, prefix: string = ""): any {
  let flattened: any = {};
  for (const key in obj) {
    if (Object.prototype.hasOwnProperty.call(obj, key)) {
      const value = obj[key];
      const prefixedKey = prefix === "" ? key : `${prefix}.${key}`;
      if (typeof value === "object" && value !== null && !Array.isArray(value)) {
        const nested = flattenObject(value, prefixedKey);
        flattened = { ...flattened, ...nested };
      } else if (Array.isArray(value)) {
        flattened[prefixedKey] = "";
      } else {
        flattened[prefixedKey] = value;
      }
    }
  }
  return flattened;
}

export const makeSignature = (
  email: number,
  timestamp: number,
  data: any,
  isFormData?: boolean
) => {
  const formDataObject: any = { email, timestamp };
  if (isFormData) {
    for (const pair of data.entries()) {
      formDataObject[pair[0]] = pair[1];
    }
  }
  const newPayload = isFormData ? formDataObject : { email, ...data, timestamp };
  const finalPayload = formatObject(newPayload);
  return CryptoJS.SHA1(finalPayload);
};
```

### Axios interceptor injecting cryptographic signature headers and handling 401 unauthorized session expiry globally.

```typescript
// Request & Response Interceptors in src/lib/axios.ts
axios.interceptors.request.use(async (config) => {
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

  config.headers["sig"] = signature;
  config.headers["email"] = "0";
  config.headers["timestamp"] = timestamp.toString();

  return config;
});

axios.interceptors.response.use(
  (response) => response,
  async (error) => {
    const status = error.response?.status;
    const data = error.response?.data;

    if (status === 401) {
      const isLoginPage = window.location.pathname.includes("/login") || window.location.hash.includes("/login");
      if (!isLoginPage) {
        localStorage.removeItem(LOCALSTORAGE_KEY.TOKEN);
        localStorage.removeItem(LOCALSTORAGE_KEY.REFRESH_TOKEN);
        localStorage.removeItem(LOCALSTORAGE_KEY.USER);

        await Swal.fire({
          title: "Sesi telah kedaluwarsa.",
          icon: "warning",
          confirmButtonText: "Go to Login",
          allowOutsideClick: false,
          allowEscapeKey: false,
        });

        window.location.href = "/login";
      }
    }
    return Promise.reject(error);
  }
);
```

## 5. Considerations & Best Practices

- Timestamp synchronization between client and backend is critical to prevent signature replay expiration.
- FormData handling requires manual iterator extraction (`data.entries()`) to compute payload signature correctly.
- Login page checking (`isLoginPage`) prevents recursive modal loops during failed login attempts.

## 6. Related Knowledge

- `Axios Interceptors`
- `HMAC/Request Signing`
- `SweetAlert2 Modal Invalidation`
- `FormData API`

## 7. Source

- Harvest 1787112018130 D8cc3a8b.json
