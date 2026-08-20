---
title: "Centralized IPC Error Normalization & Auth Session Expiration Gate"
type: pattern
tags: [pattern, frontend, error-handling, auth, session-management, toast-cooldown]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787112360086-3c0f5be1
sources: ["Harvest 1787112360086 3c0f5be1.json"]
---

# Centralized IPC Error Normalization & Auth Session Expiration Gate

Centralized IPC error handler and auth session gate with status code extraction, rate-limited toast alerts, and automatic credential cleanup.

## 1. Overview & Architecture

A centralized IPC invocation wrapper in the renderer process that intercepts unauthorized status codes, clears authentication credentials, rate-limits error toasts, and normalizes error messages.

## 2. Implementation & Code Structure

src/renderer/src/lib/
├── clear-auth-session.ts          # Removes auth tokens and redirects
├── ipc-service-error.ts           # Custom error class with HTTP status code
├── normalize-ipc-error-message.ts # Parses IPC error serialization strings (e.g. HTTP_STATUS_401::...)
└── invoke-ipc-with-auth-handling.ts # Higher-order execution wrapper for all IPC calls

## 3. Key Implementation Points

- Extracts HTTP status codes across serialized Electron IPC error strings.
- Automatically invokes `clearAuthSession()` on 401 Unauthorized responses.
- Rate-limits expired session toast notifications via a timestamp cooldown guard.
- Normalizes raw errors into strongly-typed `IpcServiceError` instances.

## 4. Code Examples

### Centralized IPC error handler with 401 interceptor, toast cooldown, and session cleanup

```typescript
// src/renderer/src/lib/invoke-ipc-with-auth-handling.ts
import { clearAuthSession } from './clear-auth-session'
import { IpcServiceError } from './ipc-service-error'
import { extractIpcErrorStatusCode, normalizeIpcErrorMessage } from './normalize-ipc-error-message'
import { toast } from 'sonner'

interface InvokeIpcWithAuthHandlingOptions {
  methodName: string
  fallbackMessage: string
  treatUnauthorizedAsSessionExpired?: boolean
}

const SESSION_EXPIRED_MESSAGE = 'Sesi login telah berakhir. Silakan login kembali.'
const SESSION_EXPIRED_TOAST_COOLDOWN_MS = 3000
let lastSessionExpiredToastAt = 0

export const invokeIpcWithAuthHandling = async <T>(
  action: () => Promise<T>,
  options: InvokeIpcWithAuthHandlingOptions
): Promise<T> => {
  try {
    return await action()
  } catch (error) {
    const statusCode = extractIpcErrorStatusCode(error, { methodName: options.methodName })
    const shouldHandle401AsSessionExpired = options.treatUnauthorizedAsSessionExpired ?? true

    if (statusCode === 401 && shouldHandle401AsSessionExpired) {
      clearAuthSession()
      const now = Date.now()
      if (now - lastSessionExpiredToastAt > SESSION_EXPIRED_TOAST_COOLDOWN_MS) {
        lastSessionExpiredToastAt = now
        toast.error(SESSION_EXPIRED_MESSAGE)
      }
      throw new IpcServiceError(SESSION_EXPIRED_MESSAGE, 401)
    }

    throw new IpcServiceError(
      normalizeIpcErrorMessage(error, {
        methodName: options.methodName,
        fallbackMessage: options.fallbackMessage,
      }),
      statusCode ?? undefined
    )
  }
}
```

## 5. Considerations & Best Practices

- Requires main-process services to serialize status codes in a known prefix format (`HTTP_STATUS_${status}::${message}`).
- Toast cooldown window (3000ms) prevents UI jitter and toast storming when multiple parallel queries fail with 401 simultaneously.
- Throws structured `IpcServiceError` allowing downstream React Query hooks to selectively suppress alerts for already-handled status codes.

## 6. Related Knowledge

- Ipc Error Normalization
- Session Expiration Handling

## 7. Source

- Harvest 1787112360086 3c0f5be1.json
