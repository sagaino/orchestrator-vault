---
title: "Centralized IPC Invoker with Rate-Limited 401 Session Interceptor"
type: pattern
tags: [pattern, frontend, error-handling, ipc, auth, sonner, session-management]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787111316881-96cf68d4
sources: ["Harvest 1787111316881 96cf68d4.json"]
---

# Centralized IPC Invoker with Rate-Limited 401 Session Interceptor

Centralized IPC invocation wrapper with error status code extraction, session revocation, and notification throttling.

## 1. Overview & Architecture

A centralized higher-order IPC invocation wrapper in the renderer process that intercepts 401 Unauthorized responses, cleans up local auth sessions, and throttles toast notifications across concurrent requests.

## 2. Implementation & Code Structure

src/renderer/src/lib/
├── invoke-ipc-with-auth-handling.ts  # Core IPC invocation wrapper with 401 interceptor
├── normalize-ipc-error-message.ts    # Extracts status codes (e.g. HTTP_STATUS_401) and clean messages
├── ipc-service-error.ts              # Custom Error class retaining status code and origin
└── clear-auth-session.ts             # Storage reset & redirect trigger

## 3. Key Implementation Points

- Extract HTTP status codes from structured error prefixes (e.g. 'HTTP_STATUS_401::...').
- Automatically wipe credentials and tokens via clearAuthSession on 401 Unauthorized responses.
- Debounce/throttle session-expired toast notifications using timestamp comparison (SESSION_EXPIRED_TOAST_COOLDOWN_MS).
- Re-throw standard IpcServiceError instances containing clean human-readable messages and status codes.

## 4. Code Examples

### IPC invoker wrapper with status extraction, session clearing, and toast throttling

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

- Session expired toasts must be throttled with a cooldown timer to avoid multiple notifications during concurrent API queries.
- The wrapper preserves the typed response promise <T> while normalizing arbitrary IPC error shapes.

## 6. Related Knowledge

- `Axios / Fetch Interceptors`
- `Sonner Toast Notifications`
- `Error Handling in Async IPC`

## 7. Source

- Harvest 1787111316881 96cf68d4.json
