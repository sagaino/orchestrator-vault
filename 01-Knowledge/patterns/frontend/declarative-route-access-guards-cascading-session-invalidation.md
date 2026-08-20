---
title: "Declarative Route Access Guards & Cascading Session Invalidation"
type: pattern
tags: [pattern, frontend, routing, guards, react-router, auth-flow, security]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127443075-f8fd0716
sources: ["Harvest 1787127443075 F8fd0716.json"]
---

# Declarative Route Access Guards & Cascading Session Invalidation

Declarative route access guards coupled with IPC interceptor-driven session invalidation cascade.

## 1. Overview & Architecture

Sistem navigasi deklaratif berbasis Hash Router yang terintegrasi erat dengan Guard Components dan interceptor IPC otomatis saat sesi kadaluarsa.

## 2. Implementation & Code Structure

src/renderer/src/
├── routes/
│   └── index.tsx                    # createHashRouter with nested guarded layouts
├── components/common/Auth/
│   ├── PrivateRoute.tsx             # Auth guard (redirects unauthenticated to login)
│   └── PublicRoute.tsx              # Guest guard (redirects authenticated to home)
└── lib/
    ├── invoke-ipc-with-auth-handling.ts  # 401 Interceptor and Session Invalidation
    └── clear-auth-session.ts             # Session wipe & storage event broadcast

## 3. Key Implementation Points

- Declarative route guards memisahkan public dan private route hierarchy.
- Global IPC Interceptor mendeteksi 401 status code dan mengeksekusi `clearAuthSession`.
- Event `storage` yang dipicu oleh `clearAuthSession` secara otomatis membuat `PrivateRoute` me-render `Navigate to="/login"`.

## 4. Code Examples

### Route Guard and IPC Interceptor coordinating session invalidation and navigation redirect

```typescript
// src/renderer/src/components/common/Auth/PrivateRoute.tsx
export const PrivateRoute: React.FC<IPrivateRoute> = ({ children }: IPrivateRoute) => {
  const [token] = useLocalStorage<string | null>(LOCALSTORAGE_KEY.TOKEN, null, { encrypted: true });
  const location = useLocation();

  if (!token) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return <>{children}</>;
};

// src/renderer/src/lib/invoke-ipc-with-auth-handling.ts
export const invokeIpcWithAuthHandling = async <T>(
  action: () => Promise<T>,
  options: InvokeIpcWithAuthHandlingOptions
): Promise<T> => {
  try {
    return await action();
  } catch (error) {
    const statusCode = extractIpcErrorStatusCode(error, { methodName: options.methodName });
    if (statusCode === 401 && (options.treatUnauthorizedAsSessionExpired ?? true)) {
      clearAuthSession();
      toast.error('Sesi login telah berakhir. Silakan login kembali.');
      throw new IpcServiceError('Sesi login telah berakhir. Silakan login kembali.', 401);
    }
    throw error;
  }
};
```

## 5. Considerations & Best Practices

- Gunakan `createHashRouter` untuk kompatibilitas native protokol file di Electron (`file://`).
- Penggunaan cooldown toast pada session expiration mencegah toast spam saat banyak request gagal serentak karena 401.

## 6. Related Knowledge

- `React Router Declarative Guards`
- `Token Lifecycle & Session Invalidation Patterns`

## 7. Source

- Harvest 1787127443075 F8fd0716.json
