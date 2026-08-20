---
title: "ipc-exception-protocol-and-debounced-session-invalidation-interceptor"
type: pattern
tags: [pattern, frontend, auth, error-handling, ipc-bridge, electron-renderer, session-management]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127443075-f8fd0716
sources: ["Harvest 1787127443075 F8fd0716.json"]
---

# ipc-exception-protocol-and-debounced-session-invalidation-interceptor

IPC exception protocol with status-code-aware error normalizer, debounced toast cooldown, and multi-window session invalidation.

## 1. Overview & Architecture

Ketika error HTTP terjadi di Main process Electron, error tersebut diserialisasi dan dilempar melintasi jembatan IPC sebagai error string. Pola ini menyediakan layer abstraction interceptor di Renderer yang mengurai kode status HTTP (misalnya HTTP_STATUS_401::...), membersihkan sesi autentikasi terenkripsi secara serentak via storage event, menerapkan cooldown debouncing pada toast notifikasi, dan membungkus error menjadi IpcServiceError bertipe kuat.

## 2. Implementation & Code Structure

src/renderer/src/
├── lib/
│   ├── invoke-ipc-with-auth-handling.ts  # Higher-order IPC call interceptor
│   ├── normalize-ipc-error-message.ts    # IPC method prefix stripper & status code extractor
│   ├── clear-auth-session.ts             # Synchronous session eviction & broadcast
│   └── ipc-service-error.ts              # Custom Error subclass with statusCode
└── services/
    ├── login.ts                          # Auth caller (treatUnauthorizedAsSessionExpired = false)
    └── operator.ts                       # Authenticated caller (treatUnauthorizedAsSessionExpired = true)

## 3. Key Implementation Points

- RegEx-based parsing untuk mengekstrak HTTP status code dari serialized IPC exception strings.
- Pembersihan otomatis prefix internal Electron (Error invoking remote method '...').
- Flag treatUnauthorizedAsSessionExpired untuk membedakan kesalahan login vs session expiration.
- Cooldown debouncing pada sonner toast untuk mencegah toast storm pada concurrent request failures.
- Sinkronisasi multi-window via window.dispatchEvent(new Event('storage')).

## 4. Code Examples

### Higher-order IPC wrapper with status code extraction, debounced toast handling, and multi-window session purge

```typescript
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

- Menyatukan error boundary IPC dan auth lifecycle di satu pintu gerbang tanpa mengotori view components.
- Mencegah toast storm saat concurrent queries (misalnya React Query) serentak gagal 401.
- Bergantung pada konvensi penamaan error HTTP_STATUS_xxx:: dari Main process service layer.

## 6. Related Knowledge

- `patterns/frontend/ipc-error-handling-pipeline.md`
- `decisions/auth-session-invalidation-flow.md`
- `debugging/electron-ipc-error-serialization.md`

## 7. Source

- Harvest 1787127443075 F8fd0716.json
