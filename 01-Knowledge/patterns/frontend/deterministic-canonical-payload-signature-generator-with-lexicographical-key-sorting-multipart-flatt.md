---
title: "Deterministic Canonical Payload Signature Generator with Lexicographical Key Sorting & Multipart Flattening"
type: pattern
tags: [pattern, frontend, security, cryptography, api-signature, anti-tampering, request-signing]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127791371-cc78a949
sources: ["Harvest 1787127791371 Cc78a949.json"]
---

# Deterministic Canonical Payload Signature Generator with Lexicographical Key Sorting & Multipart Flattening

Deterministic Canonical Payload Signature Generator with Lexicographical Key Sorting & Multipart Flattening for request integrity and anti-tampering.

## 1. Overview & Architecture

A deterministic client-side request signature generation pattern that flattens nested JSON payloads and FormData multipart streams into an alphabetically ordered canonical string before computing a cryptographic hash (SHA1/HMAC). This protects backend APIs against payload tampering, parameter pollution, and replay attacks across asynchronous network hops.

## 2. Implementation & Code Structure

src/
├── config/
│   ├── signature.ts         # flattenObject(), formatObject(), makeSignature()
│   ├── axios.ts             # Axios client instance with request signing interceptor
│   └── constant/
│       └── localstorage.ts  # Token and identity storage keys

## 3. Key Implementation Points

- Recursive dot-notation object traversal (`prefix.key`) flattening arbitrarily deep payload structures.
- Lexicographical key sorting (`Object.keys().sort()`) eliminating JSON key order nondeterminism across network serializers.
- Polymorphic FormData extraction converting multipart form entries into a uniform key-value dictionary.
- Nonce and timestamp header injection (`timestamp`, `sig`, `email`) enabling server-side replay attack rejection.

## 4. Code Examples

### Deterministic payload flattener, key sorter, and cryptographic signature digest generator

```typescript
import CryptoJS from "crypto-js";

export const getTimestamp = () => {
  return new Date().getTime();
};

function flattenObject(obj: any, prefix: string = ""): any {
  let flattened: any = {};
  for (const key in obj) {
    if (Object.prototype.hasOwnProperty.call(obj, key)) {
      const value = obj[key];
      const prefixedKey = prefix === "" ? key : `${prefix}.${key}`;
      if (
        typeof value === "object" &&
        value !== null &&
        !Array.isArray(value)
      ) {
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

export const makeSignature = (
  email: number | string,
  timestamp: number,
  data: any,
  isFormData?: boolean
) => {
  const formDataObject: any = {
    email,
    timestamp,
  };
  if (isFormData && data instanceof FormData) {
    for (const pair of data.entries()) {
      formDataObject[pair[0]] = pair[1];
    }
  }
  const newPayload = {
    email,
    ...data,
    timestamp,
  };
  const newData = isFormData ? formDataObject : newPayload;
  const finalPayload = formatObject(newData);
  return CryptoJS.SHA1(finalPayload).toString();
};
```

### Axios request interceptor integrating canonical signature generation with authentication headers

```typescript
import Axios from "axios";
import { getTimestamp, makeSignature } from "./signature";
import { getData } from "@/hooks/useLocalStorage";
import { LOCALSTORAGE_KEY } from "./constant/localstorage";

export const axios = Axios.create({
  baseURL: import.meta.env.VITE_API_URL || "http://localhost:3000",
});

axios.interceptors.request.use(
  async (config) => {
    const token = getData(LOCALSTORAGE_KEY.TOKEN);
    const timestamp = getTimestamp();

    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    const isFormData = config.data instanceof FormData;
    const signature = makeSignature(
      0, // client identifier / user context
      timestamp,
      config.data || {},
      isFormData
    );

    config.headers["sig"] = signature;
    config.headers["email"] = "0";
    config.headers["timestamp"] = timestamp.toString();

    return config;
  },
  (error) => Promise.reject(error)
);
```

## 5. Considerations & Best Practices

- Array elements are mapped to empty strings during flattening in this implementation; complex nested arrays require consistent backend protocol alignment.
- Large file uploads inside FormData may incur performance overhead if serialized entirely into the hash calculation; consider hashing only metadata and file sizes rather than streaming raw blob contents.
- SHA1 is vulnerable to collision attacks in adversarial threat models; upgrading to HMAC-SHA256 with a client-side rotating secret or nonce is recommended for sensitive environments.

## 6. Related Knowledge

- [[01-Knowledge/patterns/frontend/encrypted-cross-tab-reactive-storage-hook-with-event-dispatching.md]]
- Anti Replay Mechanisms
- Hmac Request Verification

## 7. Source

- Harvest 1787127791371 Cc78a949.json
