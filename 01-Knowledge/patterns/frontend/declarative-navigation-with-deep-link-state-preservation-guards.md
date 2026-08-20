---
title: "Declarative Navigation with Deep-Link State Preservation Guards"
type: pattern
tags: [pattern, frontend, routing, react-router, guards, deep-linking]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127791371-cc78a949
sources: ["Harvest 1787127791371 Cc78a949.json"]
---

# Declarative Navigation with Deep-Link State Preservation Guards

Declarative Navigation with Deep-Link State Preservation Guards.

## 1. Overview & Architecture

Declarative React Router v6 architecture with encapsulated authentication guards, guest route redirection, and deep-link target restoration.

## 2. Implementation & Code Structure

- src/routes/index.tsx: Declarative route definition with createBrowserRouter
- src/routes/PrivateRoute.tsx: Authentication guard with deep-link state preservation
- src/routes/PublicRoute.tsx: Guest-only route guard
- src/config/constant/routes.ts: Centralized route constants

## 3. Key Implementation Points

- Encapsulated layout nesting via Outlet inside PrivateRoute
- Deep-link preservation using state: { from: location } in Navigate
- PublicRoute redirection preventing logged-in users from accessing login page

## 4. Code Examples

### Declarative Route tree with PrivateRoute and PublicRoute guards

```typescript
// src/routes/index.tsx
export const router = createBrowserRouter([
  {
    path: "/",
    element: (
      <PrivateRoute>
        <Outlet />
      </PrivateRoute>
    ),
    children: [
      {
        index: true,
        element: <Navigate to={ROUTES.PROJECT} replace />
      },
      {
        path: ROUTES.EDITOR,
        element: <EditorPage />
      },
      {
        path: ROUTES.PROJECT,
        element: <ProjectPage />
      }
    ]
  },
  {
    path: ROUTES.LOGIN,
    element: (
      <PublicRoute>
        <LoginPage />
      </PublicRoute>
    ),
  },
  {
    path: "*",
    element: <NotFound />,
  },
]);

// src/routes/PrivateRoute.tsx
const PrivateRoute: React.FC<IPrivateRoute> = ({ children }: IPrivateRoute) => {
  const [token] = useLocalStorage<string | null>(LOCALSTORAGE_KEY.TOKEN, null, { encrypted: true });
  const location = useLocation();

  if (!token) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return <>{children}</>;
};
```

## 5. Considerations & Best Practices

- Post-login redirects must check location.state?.from before falling back to default route

## 6. Related Knowledge

- `react-router/protected-routes`
- `auth/deep-link-redirection`

## 7. Source

- Harvest 1787127791371 Cc78a949.json
