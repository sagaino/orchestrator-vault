---
title: "Unified Schema Validation & Multi-format Error Normalization Pattern"
type: pattern
tags: [pattern, frontend, validation, zod, error-handling, toast, user-experience]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787111639165-e1d30e37
sources: ["Harvest 1787111639165 E1d30e37.json"]
---

# Unified Schema Validation & Multi-format Error Normalization Pattern

Unified validation pipeline combining client-side Zod schemas with centralized API error message normalization and toast presentation.

## 1. Overview & Architecture

A cohesive validation and error handling pipeline combining Zod runtime schema validation on the client with structured error normalization from backend Axios responses.

## 2. Implementation & Code Structure

src/
├── config/
│   └── error-utils.ts       # Unified error formatter and Sonner toast bridge
└── pages/
    └── PersonAccess/
        └── schema/
            └── index.ts     # Zod validation schemas for forms and API mutations

## 3. Key Implementation Points

- Dual error handling: extracts both key-value validation dictionaries and single error messages.
- Multi-line formatting formatted cleanly for Sonner toast components with pre-line whitespace.
- Zod schema inference establishes compile-time and runtime validation contract parity.

## 4. Code Examples

### API error parser extracting and normalizing field validation errors vs generic messages into formatted toast notifications.

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

  // Handle field-specific validation errors dictionary
  if (apiError.errors && Object.keys(apiError.errors).length > 0) {
    return Object.values(apiError.errors);
  }

  // Handle general API error message
  if (apiError.message) {
    const formattedMessage =
      apiError.message.charAt(0).toUpperCase() + apiError.message.slice(1);
    return [formattedMessage];
  }

  return ["An error occurred"];
};

export const showErrorToast = (error: AxiosError<ApiError>) => {
  const errors = formatErrorMessage(error);
  const errorList = errors.map((err) => `• ${err}`).join("\n");

  toast.error("Error", {
    description: errorList,
    style: { whiteSpace: "pre-line" },
  });
};
```

### Zod schema declaration exporting inferable TypeScript types for forms and API requests.

```typescript
import { z } from "zod";

export const addCategorySchema = z.object({
  category_name: z
    .string()
    .min(1, "Category name is required")
    .max(50, "Category name cannot exceed 50 characters"),
  category_description: z
    .string()
    .max(255, "Description cannot exceed 255 characters")
    .optional(),
});

export type AddCategoryRequest = z.infer<typeof addCategorySchema>;
```

## 5. Considerations & Best Practices

- Ensure backend field error keys align with Zod schema property names for smooth field-level error mapping.
- Always supply a defensive fallback string when parsing undefined error payloads.

## 6. Related Knowledge

- Zod Schema Validation
- Unified Error Handling

## 7. Source

- Harvest 1787111639165 E1d30e37.json
