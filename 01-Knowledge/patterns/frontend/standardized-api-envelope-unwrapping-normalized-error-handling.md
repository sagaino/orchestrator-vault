---
title: "Standardized API Envelope Unwrapping & Normalized Error Handling"
type: pattern
tags: [pattern, frontend, error-handling, api-contracts, typescript-generics, toast]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787112018130-d8cc3a8b
sources: ["Harvest 1787112018130 D8cc3a8b.json"]
---

# Standardized API Envelope Unwrapping & Normalized Error Handling

Standardized API response envelope unwrapper and normalized multi-field error formatting utility.

## 1. Overview & Architecture

Pola pembongkaran API envelope terstandar (`unwrapResult<T>()`) dan normalisasi pesan error untuk memisahkan validasi field backend dan pesan error umum secara konsisten.

## 2. Implementation & Code Structure

src/lib/error-utils.ts
├── interface ApiError (Standard backend response envelope)
├── unwrapResult<T>() (Generic payload extractor and error thrower)
├── formatErrorMessage() (Normalized array of validation/general error strings)
└── showErrorToast() (Direct integration with UI toast system)

## 3. Key Implementation Points

- Generic unwrapper `unwrapResult<T>()` extracting pure data models and eliminating repeated status checks in service callers.
- Hierarchical error parsing covering validation maps, API error strings, and network fallback messages.
- Unified toast UI dispatch formatting multiple errors as a bulleted list.

## 4. Code Examples

### Generic API envelope unwrapping utility and formatted multi-field error extractor.

```typescript
import { toast } from "@/components/ui/toast";
import { AxiosError } from "axios";

export interface ApiError {
  code: number;
  message: string;
  errors?: Record<string, string>;
  result: null;
}

export const formatErrorMessage = (error: unknown): string[] => {
  const axiosError = error as AxiosError<ApiError>;
  const apiError = axiosError.response?.data;

  if (!apiError) {
    return [axiosError.message || "An error occurred"];
  }

  // If we have field-specific validation errors
  if (apiError.errors && Object.keys(apiError.errors).length > 0) {
    return Object.values(apiError.errors);
  }

  // If we have a general error message
  if (apiError.message) {
    const formattedMessage = apiError.message.charAt(0).toUpperCase() + apiError.message.slice(1);
    return [formattedMessage];
  }

  return ["An error occurred"];
};

export function unwrapResult<T>(response: {
  code?: number;
  message?: string;
  result?: T | null;
}): T {
  if (response.code && response.code !== 200) {
    throw new Error(response.message || 'Request gagal');
  }

  if (!response.result) {
    throw new Error(response.message || 'Data tidak ditemukan');
  }

  return response.result;
}

export const showErrorToast = (error: AxiosError<ApiError>) => {
  const errors = formatErrorMessage(error);
  const errorList = errors.map(err => `• ${err}`).join('\n');

  toast.add({
    title: "Error",
    type: "error",
    description: errorList,
  });
};
```

## 5. Considerations & Best Practices

- Assumes backend adheres to the `{ code, message, errors, result }` envelope convention.
- UnwrapResult throws JavaScript Error objects that can be caught cleanly by React Query mutations or try-catch blocks.

## 6. Related Knowledge

- `Result pattern`
- `Axios Error handling`
- `TypeScript Generics`
- `Toast notification integration`

## 7. Source

- Harvest 1787112018130 D8cc3a8b.json
