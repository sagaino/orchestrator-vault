---
title: "IPC Error Normalization & 401 Session Interceptor"
type: pattern
tags: [pattern, frontend, error-handling, authentication, ipc, session-management, toast]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787109583565-5ff8f29f
sources: ["Harvest 1787109583565 5ff8f29f.json"]
---

# IPC Error Normalization & 401 Session Interceptor

Centralized IPC error normalization with automatic 401 session expiration interceptor and toast rate-limiting.

## 1. Overview & Architecture

Encapsulated error normalization and session expiration interceptor bridging the Electron IPC serialization boundary.

## 2. Implementation & Code Structure

src/renderer/src/lib/invoke-ipc-with-auth-handling.ts -> src/renderer/src/lib/normalize-ipc-error-message.ts -> src/renderer/src/lib/clear-auth-session.ts

## 3. Key Implementation Points

- Main process encodes HTTP statuses into error messages as HTTP_STATUS_${code}::${message}
- Renderer extracts HTTP status and strips internal Electron IPC method prefix strings
- Centralized session invalidation and debounced toast notifications

## 4. Code Examples

### IPC action wrapper intercepting 401 status and normalizing serialized errors

```typescript
export const invokeIpcWithAuthHandling = async <T>(
  action: () => Promise<T>,
  options: InvokeIpcWithAuthHandlingOptions
): Promise<T> => {
  try {
    return await action();
  } catch (error) {
    const statusCode = extractIpcErrorStatusCode(error, { methodName: options.methodName });
    const shouldHandle401AsSessionExpired = options.treatUnauthorizedAsSessionExpired ?? true;

    if (statusCode === 401 && shouldHandle401AsSessionExpired) {
      clearAuthSession();
      const now = Date.now();
      if (now - lastSessionExpiredToastAt > SESSION_EXPIRED_TOAST_COOLDOWN_MS) {
        lastSessionExpiredToastAt = now;
        toast.error(SESSION_EXPIRED_MESSAGE);
      }
      throw new IpcServiceError(SESSION_EXPIRED_MESSAGE, 401);
    }

    throw new IpcServiceError(
      normalizeIpcErrorMessage(error, {
        methodName: options.methodName,
        fallbackMessage: options.fallbackMessage,
      }),
      statusCode ?? undefined
    );
  }
};
```

## 5. Considerations & Best Practices

- Debounced toast triggers prevent notification spam on bulk 401 failures.
- Configurable treatUnauthorizedAsSessionExpired flag supports form login errors where 401 means invalid credentials rather than session expiry.

## 6. Related Knowledge

- Ipc Error Boundary Interceptor
- `Axios / IPC error bridging`

## 7. Source

- Harvest 1787109583565 5ff8f29f.json
