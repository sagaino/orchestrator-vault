---
title: "HMAC Payload Signature & Centralized Auth Interceptor Pattern"
type: pattern
tags: [pattern, frontend, security, api, axios, hmac, interceptor, auth]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787111639165-e1d30e37
sources: ["Harvest 1787111639165 E1d30e37.json"]
---

# HMAC Payload Signature & Centralized Auth Interceptor Pattern

Comprehensive pattern for securing outgoing REST API requests with deterministic HMAC SHA-1 signature calculation and centralizing authentication lifecycle (session timeout/token eviction).

## 1. Overview & Architecture

Centralized HTTP communication pattern utilizing Axios interceptors to automatically enforce anti-tampering request signing (HMAC/SHA-1) on every outgoing HTTP request and gracefully manage authentication lifecycle/session revocation globally.

## 2. Implementation & Code Structure

src/
├── config/
│   ├── axios.ts         # Axios instance with request and response interceptors
│   ├── signature.ts     # Canonical flattening, sorting, and SHA-1 hashing logic
│   └── constant/
│       └── localstorage.ts # LocalStorage key constants (TOKEN, USER)
└── hooks/
    └── useLocalStorage.ts # Safe storage reader/writer

## 3. Key Implementation Points

- Request Interceptors automatically compute canonical SHA-1 payload signatures alongside request timestamps.
- Recursive object flattening ensures nested structures are deterministically sorted prior to hashing.
- Response Interceptor isolates 401 Unauthorized responses to gracefully clear auth storage and trigger SweetAlert2 modal.

## 4. Code Examples

### Axios Interceptor for automatic HMAC signature injection and centralized 401 session expiration handling.

```typescript
import Axios from "axios";
import { getTimestamp, makeSignature } from "./signature";
import { LOCALSTORAGE_KEY } from "./constant/localstorage";
import { getData } from "@/hooks/useLocalStorage";
import Swal from "sweetalert2";

export const axios = Axios.create({
  baseURL: import.meta.env.VITE_API_URL || "http://localhost:3000",
});

// Request Interceptor: Inject JWT Token & Cryptographic HMAC Signature
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

    config.headers["sig"] = signature;
    config.headers["email"] = "0";
    config.headers["timestamp"] = timestamp.toString();

    return config;
  },
  (error) => Promise.reject(error)
);

// Response Interceptor: Centralized 401 Session Invalidation & Expiration Handling
axios.interceptors.response.use(
  (response) => response,
  async (error) => {
    const status = error.response?.status;
    const data = error.response?.data;

    if (status === 401) {
      const isLoginPage = window.location.pathname.includes("/login") || window.location.hash.includes("/login");

      if (!isLoginPage) {
        localStorage.removeItem(LOCALSTORAGE_KEY.TOKEN);
        localStorage.removeItem(LOCALSTORAGE_KEY.USER);

        let msg = "Sesi telah kedaluwarsa.";
        if (data?.message) {
          msg = typeof data.message === "string" ? data.message : (data.message.msg_ind || JSON.stringify(data.message));
        }

        await Swal.fire({
          title: msg.split(",")[0],
          text: "Sesi telah kedaluwarsa.",
          icon: "warning",
          confirmButtonText: "Go to Login",
          allowOutsideClick: false,
          allowEscapeKey: false,
          confirmButtonColor: "#FFD700",
        });

        window.location.href = "/login";
      }
    }
    return Promise.reject(error);
  }
);
```

### Deterministic object flattening, alphabetical key sorting, and SHA-1 signature generation algorithm.

```typescript
import CryptoJS from "crypto-js";

export const getTimestamp = () => new Date().getTime();

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

function formatObject(obj: any): string {
  const flattened = flattenObject(obj);
  let result = "";
  Object.keys(flattened)
    .sort()
    .forEach((key) => {
      const value = flattened[key];
      result += `${key}${value}`;
    });
  return result;
}

export const makeSignature = (email: number, timestamp: number, data: any, isFormData?: boolean) => {
  const formDataObject: any = { email, timestamp };
  if (isFormData) {
    for (const pair of data.entries()) {
      formDataObject[pair[0]] = pair[1];
    }
  }
  const newPayload = { email, ...data, timestamp };
  const newData = isFormData ? formDataObject : newPayload;
  const finalPayload = formatObject(newData);
  return CryptoJS.SHA1(finalPayload);
};
```

## 5. Considerations & Best Practices

- Ensure backend uses identical deterministic key sorting and prefix flattening algorithm to verify signatures.
- FormData payload signing only captures primitive key-value pairs; binary streams should have dedicated checksums.
- 401 redirect ignores active login pages to avoid infinite redirect loops.

## 6. Related Knowledge

- Hmac Request Signing
- Axios Interceptor Architecture

## 7. Source

- Harvest 1787111639165 E1d30e37.json
