---
title: "deterministic-canonical-payload-flattening-request-signing-pipeline"
type: pattern
tags: [pattern, frontend, security, cryptography, request-signing, http-client, tamper-proofing]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787132260416-42b894f9
sources: ["Harvest 1787132260416 42b894f9.json"]
---

# deterministic-canonical-payload-flattening-request-signing-pipeline

Deterministic Canonical Payload Flattening & SHA-1 Request Signing Pipeline

## 1. Overview & Architecture

Pola arsitektur kriptografi untuk integritas request dan mitigasi replay attack melalui standardisasi kanonikal payload (JSON & FormData), dot-notation flattening, sorting kunci alfabetis, dan hashing SHA-1 otomatis sebelum request dikirimkan ke server.

## 2. Implementation & Code Structure

src/config/signature.ts (flattenObject, formatObject, makeSignature) -> src/config/axios.ts (axios request interceptor)

## 3. Key Implementation Points

- Recursive dot-notation flattening for nested object hierarchies
- Alphabetical key ordering before serializing into canonical string
- Multi-part FormData entries iteration for binary/file upload payload signatures
- SHA-1 hash calculation via CryptoJS integrated into HTTP client pipeline

## 4. Code Examples

### Deterministic Canonical Request Signing & Payload Flattening with FormData Deconstruction

```typescript
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
  email: number,
  timestamp: number,
  data: any,
  isFormData?: boolean
) => {
  const formDataObject: any = {
    email,
    timestamp,
  };
  if (isFormData) {
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
  return CryptoJS.SHA1(finalPayload);
};
```

## 5. Considerations & Best Practices

- Nested array normalization: Array values are converted to empty strings in payload flattening to match backend signing conventions.
- FormData iterator compatibility: Multi-part form data payloads are destructured using FormData.entries() to guarantee consistent key-value extraction prior to hashing.
- Key sorting guarantee: Object.keys().sort() ensures strict canonical representation regardless of JSON key insertion order.

## 6. Related Knowledge

- Request Signing
- Http Interceptor Pipeline

## 7. Source

- Harvest 1787132260416 42b894f9.json
