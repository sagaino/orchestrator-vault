---
title: "two-tier-typed-error-normalization-toast-diagnostic-pipeline"
type: pattern
tags: [pattern, frontend, error-handling, http-client, toast, validation, axios]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787132260416-42b894f9
sources: ["Harvest 1787132260416 42b894f9.json"]
---

# two-tier-typed-error-normalization-toast-diagnostic-pipeline

Two-Tier Typed Error Normalization & Multi-Line Toast Diagnostic Pipeline

## 1. Overview & Architecture

Pola normalisasi error API berjenjang (field-specific validation map vs high-level message) yang memetakan error HTTP/AxiosError secara deterministik ke dalam visual notifikasi toast multi-baris yang rapi dan terstruktur.

## 2. Implementation & Code Structure

src/config/error-utils.ts (formatErrorMessage, showErrorToast) -> TanStack Query useMutation onError handlers across feature hooks

## 3. Key Implementation Points

- Standardized ApiError interface mirroring backend validation error schemas
- Hierarchical error fallback resolution (field errors -> message string -> network error -> generic fallback)
- Automated bullet formatting (• item) for multi-field validation error displays
- Direct integration with mutation hooks (useMutation onError: showErrorToast)

## 4. Code Examples

### Two-tier Error Normalization & Structured Multi-Bullet Toast Notification Pipeline

```typescript
export interface ApiError {
  code: number;
  message: string;
  errors?: Record<string, string>;
  result: null;
}

export const formatErrorMessage = (error: unknown) => {
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

  // Fallback error message
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

- Typed AxiosError unwrapping: Safely casts error to ApiError without runtime exceptions when response payload is malformed or absent.
- Field validation vs general error precedence: Prioritizes field-specific validation errors over high-level error strings to provide granular feedback.
- Multi-line toast styling: Explicitly applies style: { whiteSpace: 'pre-line' } to Sonner toast notifications for bulleted lists.

## 6. Related Knowledge

- Error Handling
- Toast Notifications

## 7. Source

- Harvest 1787132260416 42b894f9.json
