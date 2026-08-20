---
title: "Bi-Directional Route Protection with Preserved Navigation State"
type: pattern
tags: [pattern, frontend, routing, react-router, route-guards, auth-guards]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787112018130-d8cc3a8b
sources: ["Harvest 1787112018130 D8cc3a8b.json"]
---

# Bi-Directional Route Protection with Preserved Navigation State

Bi-directional route protection architecture with preserved navigation state for React Router.

## 1. Overview & Architecture

Pola pelindung rute dua arah menggunakan React Router v6 untuk mengontrol hak akses halaman berdasarkan token autentikasi.

## 2. Implementation & Code Structure

src/routes/
├── index.tsx (Main router definition with nested route trees)
├── PrivateRoute.tsx (Guard for authenticated private pages)
└── PublicRoute.tsx (Guard for guest-only public pages)

## 3. Key Implementation Points

- Bi-directional route guarding separating public/guest-only flows and private authenticated areas.
- Preserving intended navigation history via `useLocation()` and `<Navigate state={{ from: location }} />`.
- Declarative composition inside `createBrowserRouter` without polluting presentation components.

## 4. Code Examples

### Guarded route component redirecting unauthenticated users to login while saving current location.

```typescript
// PrivateRoute: Guarantees authenticated access with preserved navigation state
const PrivateRoute: React.FC<IPrivateRoute> = ({ children }: IPrivateRoute) => {
  const [token] = useLocalStorage<string | null>(LOCALSTORAGE_KEY.TOKEN, null, { encrypted: true });
  const location = useLocation();

  if (!token) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return <>{children}</>;
};
```

### Guarded route component redirecting authenticated users away from login/public pages.

```typescript
// PublicRoute: Prevents logged-in users from visiting auth pages
const PublicRoute: React.FC<IPublicRoute> = ({ children }: IPublicRoute) => {
  const [token] = useLocalStorage<string | null>(LOCALSTORAGE_KEY.TOKEN, null, { encrypted: true });
  const location = useLocation();

  if (token) {
    return <Navigate to="/" replace state={{ from: location }} />;
  }

  return <>{children}</>;
};
```

### React Router v6 browser router definition composed with declarative guard wrappers.

```typescript
// Router configuration in src/routes/index.tsx
export const router = createBrowserRouter([
  {
    path: "/",
    element: (
      <PrivateRoute>
        <Outlet />
      </PrivateRoute>
    ),
    children: [
      { index: true, element: <Navigate to={ROUTES.GALLERY} replace /> },
      { path: ROUTES.GALLERY, element: <GalleryPage /> }
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
```

## 5. Considerations & Best Practices

- Location state preservation (`state: { from: location }`) enables seamless post-login redirection.
- Encrypted token read ensures consistent authentication state across routes.

## 6. Related Knowledge

- `React Router v6 createBrowserRouter`
- `React Router Navigate`
- `Authentication Route Guards`

## 7. Source

- Harvest 1787112018130 D8cc3a8b.json
