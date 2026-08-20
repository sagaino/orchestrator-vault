---
title: "Hierarchical RBAC & Dynamic Module Authorization Guard Pattern"
type: pattern
tags: [pattern, frontend, rbac, security, authorization, routing, react-router]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787111639165-e1d30e37
sources: ["Harvest 1787111639165 E1d30e37.json"]
---

# Hierarchical RBAC & Dynamic Module Authorization Guard Pattern

Declarative, multi-tiered Role-Based Access Control (RBAC) and module entitlement guarding pattern for React Router applications.

## 1. Overview & Architecture

Multi-layered route and module authorization pattern combining authentication route wrappers (PrivateRoute/PublicRoute) with granular module capability guards (ModuleGuard) based on user roles and assigned feature flags.

## 2. Implementation & Code Structure

src/
├── components/
│   └── common/
│       └── auth/
│           ├── ModuleGuard.tsx # Module entitlement guard
│           ├── PrivateRoute.tsx # Auth token presence guard
│           └── PublicRoute.tsx  # Guest-only route guard
├── config/
│   └── constant/
│       ├── DefaultModules.ts    # Whitelisted modules for all users
│       └── routes.ts            # Canonical route path mappings
└── routes/
    └── index.tsx                # Nested route tree applying guards

## 3. Key Implementation Points

- Layered authorization: PrivateRoute ensures token validity, while ModuleGuard evaluates fine-grained module entitlements.
- Special SUPERADMIN bypass with targeted module blacklisting.
- Fallback to whitelisted default modules for baseline operational access.

## 4. Code Examples

### ModuleGuard component providing declarative role and module-level permission enforcement for UI views.

```typescript
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

  // SUPERADMIN: Access to all modules except designated exclusions
  if (userRole === "SUPERADMIN") {
    if (moduleKey === "inventory") {
      return <Navigate to={ROUTES.DASHBOARD} replace />;
    }
    return <>{children}</>;
  }

  // Standard Role: Access restricted to default whitelist or explicit user modules
  const hasAccess = defaultModules.includes(moduleKey) || modules.includes(moduleKey);

  if (!hasAccess) {
    return <Navigate to={ROUTES.DASHBOARD} replace />;
  }

  return <>{children}</>;
};

export default ModuleGuard;
```

### PrivateRoute component safeguarding protected routing subtrees against unauthenticated access.

```typescript
import { Navigate, Outlet } from "react-router-dom";
import { getData } from "@/hooks/useLocalStorage";
import { LOCALSTORAGE_KEY } from "@/config/constant/localstorage";
import { ROUTES } from "@/config/constant/routes";

const PrivateRoute = () => {
  const token = getData(LOCALSTORAGE_KEY.TOKEN);

  if (!token) {
    return <Navigate to={ROUTES.LOGIN} replace />;
  }

  return <Outlet />;
};

export default PrivateRoute;
```

## 5. Considerations & Best Practices

- Avoid relying solely on client-side RBAC guards; backend endpoints must independently enforce role authorization.
- Keep route constants and module names synchronized via TypeScript string literals/enums to avoid permission drift.

## 6. Related Knowledge

- Rbac Authorization
- React Router Guarding

## 7. Source

- Harvest 1787111639165 E1d30e37.json
