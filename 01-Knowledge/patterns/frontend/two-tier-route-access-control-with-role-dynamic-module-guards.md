---
title: "Two-Tier Route Access Control with Role & Dynamic Module Guards"
type: pattern
tags: [pattern, frontend, routing, auth, rbac, react-router, security]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787132260416-42b894f9
sources: ["Harvest 1787132260416 42b894f9.json"]
---

# Two-Tier Route Access Control with Role & Dynamic Module Guards

Two-Tier Route Access Control with Role & Dynamic Module Guards

## 1. Overview & Architecture

Pola otentikasi dan otorisasi dua tingkat (Two-Tier Route Access Control) yang memisahkan verifikasi token otentikasi global (Level 1 - PrivateRoute) dengan hak akses per modul/fitur berbasis Role dan Module ACL (Level 2 - ModuleGuard) dalam router deklaratif React Router v6.

## 2. Implementation & Code Structure

src/
├── routes/
│   └── index.tsx                 # Central Router configuration (createBrowserRouter)
├── components/common/Auth/
│   ├── PrivateRoute.tsx          # Auth token guard with location preservation
│   ├── PublicRoute.tsx           # Inverse auth guard for guest-only pages
│   └── ModuleGuard.tsx           # RBAC / Feature module permission guard
└── layout/
    └── PrivateLayout.tsx         # Sidebar, Header, and Outlet wrapper

## 3. Key Implementation Points

- Nested route architecture in React Router v6 using Outlet and Layout wrappers.
- Encrypted session token verification before rendering child route elements.
- Fine-grained module level authorization with SUPERADMIN privilege overrides and fallback redirects.

## 4. Code Examples

### Level 1 Authentication Guard protecting root layout and redirecting unauthenticated users while preserving location state.

```typescript
// src/components/common/Auth/PrivateRoute.tsx
import { LOCALSTORAGE_KEY } from '@/config/constant/localstorage'
import useLocalStorage from '@/hooks/useLocalStorage'
import React from 'react'
import { Navigate, useLocation } from 'react-router-dom'

interface IPrivateRoute {
  children: React.ReactNode
}

const PrivateRoute: React.FC<IPrivateRoute> = ({ children }) => {
  const [token] = useLocalStorage<string | null>(LOCALSTORAGE_KEY.TOKEN, null, { encrypted: true })
  const location = useLocation()

  if (!token) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return <>{children}</>;
}

export default PrivateRoute;
```

### Level 2 Module and Role-based Access Control (RBAC/ACL) Route Guard.

```typescript
// src/components/common/Auth/ModuleGuard.tsx
import { ROUTES } from "@/config/constant/routes";
import { Navigate } from "react-router-dom";
import { getData } from "@/hooks/useLocalStorage";
import { LOCALSTORAGE_KEY } from "@/config/constant/localstorage";
import { defaultModules } from "@/config/constant/DefaultModules";

interface ModuleGuardProps {
  children: React.ReactNode;
  moduleKey: string;
}

const ModuleGuard = ({ children, moduleKey }: ModuleGuardProps) => {
  const userData = getData(LOCALSTORAGE_KEY.USER);
  const userRole = userData?.role_code;
  const modules = userData?.modules || [];

  // SUPERADMIN bypass with exceptions
  if (userRole === "SUPERADMIN") {
    if (moduleKey === "inventory") {
      return <Navigate to={ROUTES.DASHBOARD} replace />;
    }
    return <>{children}</>;
  }

  // Role ACL check against assigned and default modules
  const hasAccess = defaultModules.includes(moduleKey) || modules.includes(moduleKey);

  if (!hasAccess) {
    return <Navigate to={ROUTES.DASHBOARD} replace />;
  }

  return <>{children}</>;
};

export default ModuleGuard;
```

### Declarative React Router v6 configuration composing nested layout and guards.

```typescript
// src/routes/index.tsx
export const router = createBrowserRouter([
  {
    path: "/",
    element: (
      <PrivateRoute>
        <PrivateLayout>
          <Outlet />
        </PrivateLayout>
      </PrivateRoute>
    ),
    children: [
      {
        index: true,
        element: <Navigate to={ROUTES.DASHBOARD} replace />,
      },
      {
        path: ROUTES.CCTV_MONITORING,
        element: (
          <ModuleGuard moduleKey="cctv_monitoring">
            <CCTVMonitoringPage />
          </ModuleGuard>
        ),
      },
      {
        path: ROUTES.PERSON_ACCESS.ROOT,
        element: (
          <ModuleGuard moduleKey="person_access">
            <PersonAccessPage />
          </ModuleGuard>
        ),
      }
    ],
  },
]);
```

## 5. Considerations & Best Practices

- Decoupling authentication from authorization ensures separation of concerns.
- State location preservation (`state={{ from: location }}`) enables seamless post-login redirect.
- Route protection occurs at routing boundary before feature components mount, avoiding unauthorized network requests.

## 6. Related Knowledge

- `React Router v6 Protected Routes`
- `Role-Based Access Control (RBAC)`
- `Client-side Security Best Practices`

## 7. Source

- Harvest 1787132260416 42b894f9.json
