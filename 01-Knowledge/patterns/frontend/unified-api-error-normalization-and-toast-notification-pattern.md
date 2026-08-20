---
title: "Unified API Error Normalization and Toast Notification Pattern"
type: pattern
tags: [pattern, frontend, error-handling, axios, sonner, toast, ux]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787107893306-92ee562e
sources: ["Harvest 1787107893306 92ee562e.json"]
---

# Unified API Error Normalization and Toast Notification Pattern

Normalisasi hierarkis respons error Axios / backend ke dalam format list toast yang ramah pengguna.

## 1. Overview & Architecture

Pattern penanganan error terpusat yang menormalisasi AxiosError dari API backend menjadi pesan terstruktur dan menampilkannya sebagai toast notifikasi multi-line yang ramah pengguna.

## 2. Implementation & Code Structure

src/config/
└── error-utils.ts (formatErrorMessage, showErrorToast & ApiError interface)

## 3. Key Implementation Points

- Interface ApiError yang terdefinisi jelas mencakup status code, pesan global, dan map validation error.
- Fungsi ekstraksi bertingkat: field-level error -> error message umum -> fallback generic error.
- Penyajian toast notifikasi multi-line menggunakan Sonner dengan styling whiteSpace: pre-line.

## 4. Code Examples

### Error Normalization & Multi-Line Toast Notification

```typescript
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

  // Jika terdapat error validasi field-specific
  if (apiError.errors && Object.keys(apiError.errors).length > 0) {
    return Object.values(apiError.errors);
  }

  // Jika terdapat pesan umum
  if (apiError.message) {
    const formattedMessage = apiError.message.charAt(0).toUpperCase() + apiError.message.slice(1);
    return [formattedMessage];
  }

  return ["An error occurred"];
};

export const showErrorToast = (error: AxiosError<ApiError>) => {
  const errors = formatErrorMessage(error);
  const errorList = errors.map(err => `• ${err}`).join('\n');

  toast.error("Error", {
    description: errorList,
    style: { whiteSpace: 'pre-line' }
  });
};
```

## 5. Considerations & Best Practices

- Format respons backend harus mematuhi kontrak ApiError agar parsing error fields berjalan akurat.
- Perlu fallback aman untuk skenario non-Axios error (Network error, CORS, 502 Bad Gateway HTML).

## 6. Related Knowledge

- `Axios Error Handling`
- `Sonner Toast Notification`
- `Zod Validation Schema`

## 7. Source

- Harvest 1787107893306 92ee562e.json
