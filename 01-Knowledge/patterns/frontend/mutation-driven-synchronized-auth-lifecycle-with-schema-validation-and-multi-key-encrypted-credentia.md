---
title: "Mutation-Driven Synchronized Auth Lifecycle with Schema Validation and Multi-Key Encrypted Credential Dispatch"
type: pattern
tags: [pattern, frontend, auth, react-query, mutation, validation, session-lifecycle]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127791371-cc78a949
sources: ["Harvest 1787127791371 Cc78a949.json"]
---

# Mutation-Driven Synchronized Auth Lifecycle with Schema Validation and Multi-Key Encrypted Credential Dispatch

Mutation-Driven Synchronized Auth Lifecycle with Schema Validation and Multi-Key Encrypted Credential Dispatch.

## 1. Overview & Architecture

An end-to-end type-safe authentication mutation pattern that links form handling (React Hook Form + Zod), backend service contracts, strict response verification, and atomic encrypted local storage persistence with decoupled toast feedback and async lifecycle tracking via TanStack Query.

## 2. Implementation & Code Structure

src/
├── pages/
│   └── Login/
│       ├── index.tsx              # Form UI bound to useLogin
│       ├── hooks/
│       │   └── useLogin.ts        # TanStack Query mutation & response validator
│       └── types/
│           ├── request.ts         # Zod schema and inferred TypeScript types
│           └── response.ts        # API response contracts
├── services/
│   └── auth.ts                    # AuthServices.login API client
└── hooks/
    └── useLocalStorage.ts         # setData encrypted dispatcher

## 3. Key Implementation Points

- Schema-driven type safety with Zod (`zodResolver`) binding seamless client-side validation into React Hook Form.
- Response assertion boundary (`validateLoginResponse`) ensuring only valid payloads with non-empty tokens proceed to storage.
- Atomic dual-key encrypted credential dispatch (`setData`) synchronizing token and user data simultaneously.
- Decoupled async mutation state (`isLoggingIn`, `isPending`) providing reactive UI loading feedback.

## 4. Code Examples

### Zod schema definition for type-safe credentials validation

```typescript
import { z } from "zod";

export const loginSchema = z.object({
  email: z.string().email("Email tidak valid").min(1, "Email wajib diisi"),
  password: z.string().min(6, "Password minimal 6 karakter"),
});

export type LoginFormData = z.infer<typeof loginSchema>;
```

### React Query mutation hook orchestrating validation, API execution, and encrypted storage sync

```typescript
import { useMutation } from "@tanstack/react-query";
import type { LoginFormData } from "../types/request";
import { AuthServices } from "@/services/auth";
import { setData } from "@/hooks/useLocalStorage";
import { LOCALSTORAGE_KEY } from "@/config/constant/localstorage";
import { toast } from "sonner";
import type { LoginData, LoginResponse } from "../types/response";

const validateLoginResponse = (response: LoginResponse): { token: string; user: LoginData } => {
  const token = response.result?.token;

  if (!token) {
    throw new Error(response.message || "Token tidak ditemukan pada response login");
  }

  if (response.code && response.code !== 200) {
    throw new Error(response.message || "Login gagal");
  }

  return {
    token,
    user: response.result?.user ?? {},
  };
};

export const useLogin = () => {
  const mutation = useMutation({
    mutationFn: async (payload: LoginFormData) => {
      const response = await AuthServices.login(payload);
      const { token, user } = validateLoginResponse(response);

      // Atomic encrypted persistence
      setData(LOCALSTORAGE_KEY.TOKEN, token);
      setData(LOCALSTORAGE_KEY.USER, user);

      return { response, user };
    },
    onSuccess: ({ user }) => {
      const userName = user?.name || user?.username || "Pengguna";
      toast.success(`Login berhasil. Selamat datang, ${userName}.`);
    },
    onError: (error: unknown) => {
      const message = error instanceof Error ? error.message : "Terjadi kesalahan saat login";
      toast.error(message);
    },
  });

  return {
    login: mutation.mutateAsync,
    isLoggingIn: mutation.isPending,
  };
};
```

### Login form integration with React Hook Form, Zod resolver, and mutation hook

```typescript
import React from 'react'
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Field, FieldError } from '@/components/ui/field'
import { useTranslation } from 'react-i18next'
import { loginSchema, type LoginFormData } from './types/request'
import { useLogin } from './hooks/useLogin'

const LoginPage: React.FC = () => {
  const { t } = useTranslation()
  const { login, isLoggingIn } = useLogin()
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
    defaultValues: { email: '', password: '' },
  })

  const onSubmit = async (data: LoginFormData) => {
    await login(data)
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Field>
        <Input placeholder={t('auth.email_placeholder')} {...register('email')} />
        {errors.email && <FieldError errors={[{ message: errors.email.message }]} />}
      </Field>
      <Field>
        <Input type="password" placeholder={t('auth.password_placeholder')} {...register('password')} />
        {errors.password && <FieldError errors={[{ message: errors.password.message }]} />}
      </Field>
      <Button type="submit" disabled={isSubmitting || isLoggingIn}>
        {t('auth.login')}
      </Button>
    </form>
  )
}

export default LoginPage
```

## 5. Considerations & Best Practices

- Client-side Zod validation provides immediate UI feedback but must be mirrored by strict server-side validation.
- Sensitive tokens are encrypted before persistence in localStorage; however, client-side encryption keys in frontend bundles are susceptible to extraction in high-security requirements (consider HttpOnly cookies for enterprise apps).
- Ensure mutateAsync rejections are handled in the component if additional navigation transitions depend on completion.

## 6. Related Knowledge

- [[01-Knowledge/patterns/frontend/encrypted-cross-tab-reactive-storage-hook-with-event-dispatching.md]]
- Tanstack Query Mutations

## 7. Source

- Harvest 1787127791371 Cc78a949.json
