---
title: "standardized dto error envelope normalizer & multi-target toast dispatcher"
type: pattern
tags: [pattern, frontend, error-handling, dto, sonner, axios]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127791371-cc78a949
sources: ["Harvest 1787127791371 Cc78a949.json"]
---

# standardized dto error envelope normalizer & multi-target toast dispatcher

Standardized DTO Error Envelope Normalizer & Multi-Target Toast Dispatcher provides cascading normalization of backend API response envelopes into structured bulleted toasts.

## 1. Overview & Architecture

A resilient error normalization and presentation pattern that transforms unpredictable network and API backend errors into strongly-typed DTO envelopes, cascading down to structured multi-line toast notifications.

## 2. Implementation & Code Structure

src/config/error-utils.ts -> ApiError DTO interface, formatErrorMessage() normalization pipeline, and showErrorToast() Sonner integration.

## 3. Key Implementation Points

- Strongly typed ApiError generic envelope matching backend response contract: { code, message, errors?, result }.
- Cascading priority normalization: Field-specific dictionary values -> Uppercased generic message -> Axios HTTP message -> Fallback string.
- Bulleted string synthesis for multi-field form validation failures.

## 4. Code Examples

### Standardized ApiError envelope DTO normalization and multi-line Sonner toast dispatcher

```typescript
import { AxiosError } from "axios";
import { toast } from "sonner";

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

  // If we have a general error message (like 404 not found)
  if (apiError.message) {
    const formattedMessage = apiError.message.charAt(0).toUpperCase() + apiError.message.slice(1);
    return [formattedMessage];
  }

  return ["An error occurred"];
};

export const showErrorToast = (error: AxiosError<ApiError>) => {
  const errors = formatErrorMessage(error);
  const errorList = errors.map(error => `• ${error}`).join('\n');

  toast.error("Error", {
    description: errorList,
    style: { whiteSpace: 'pre-line' }
  });
};
```

## 5. Considerations & Best Practices

- Always guarantees a non-empty array of human-readable error strings regardless of the failure shape (Axios network failure, generic error, structured validation map, or single error message).
- Sonner toast configuration uses whitespace pre-line styling to cleanly display multi-error bullet points.
- Auto-capitalizes server message strings to maintain professional UI presentation standards.

## 6. Related Knowledge

- Route Aware Session Eviction Interceptor
- Mutation Driven Synchronized Auth Lifecycle

## 7. Source

- Harvest 1787127791371 Cc78a949.json
