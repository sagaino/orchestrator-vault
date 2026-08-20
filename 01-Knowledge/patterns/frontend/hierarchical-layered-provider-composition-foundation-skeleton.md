---
title: "Hierarchical Layered Provider Composition & Foundation Skeleton"
type: pattern
tags: [pattern, frontend, architecture, foundation, react, providers]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127791371-cc78a949
sources: ["Harvest 1787127791371 Cc78a949.json"]
---

# Hierarchical Layered Provider Composition & Foundation Skeleton

Hierarchical Layered Provider Composition & Project Foundation Skeleton for React + Vite apps.

## 1. Overview & Architecture

Deterministic, hierarchical context provider tree composition separating global theme bootstrapping from feature providers and declarative routing.

## 2. Implementation & Code Structure

- src/main.tsx: Root DOM renderer & ThemeProvider wrapper
- src/App.tsx: Top-level Provider orchestration (I18n, QueryClient, Tooltip, Sonner, Router)
- src/config/constant/routes.ts: Centralized route constants contract

## 3. Key Implementation Points

- Separation of Global Theme Provider in main.tsx from application context tree in App.tsx
- Single global QueryClient instance outside React lifecycle
- Compound provider nesting ensuring global access to Sonner toast and Tooltip systems

## 4. Code Examples

### Top-level declarative compound provider tree hierarchy

```typescript
import { Sonner } from "@/components/ui/sonner";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { RouterProvider } from "react-router-dom";
import { TooltipProvider } from "./components/ui/tooltip";
import I18nProvider from "./lib/i18n/i18nProvider";
import { router } from "./routes";

const queryClient = new QueryClient();

export function App() {
  return (
    <I18nProvider>
      <QueryClientProvider client={queryClient}>
        <TooltipProvider>
          <Sonner position="top-right" richColors />
          <RouterProvider router={router} />
        </TooltipProvider>
      </QueryClientProvider>
    </I18nProvider>
  );
}

export default App;
```

## 5. Considerations & Best Practices

- Data providers (I18n, QueryClient) must wrap UI feature providers (Tooltip, Router)
- Single QueryClient instance declared outside render cycle to avoid recreation

## 6. Related Knowledge

- `frontend/project-skeleton-template`
- `tanstack-query/provider-lifecycle`
- `react-router/data-router-composition`

## 7. Source

- Harvest 1787127791371 Cc78a949.json
