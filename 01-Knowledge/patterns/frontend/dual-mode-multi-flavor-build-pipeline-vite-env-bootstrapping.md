---
title: "dual-mode-multi-flavor-build-pipeline-vite-env-bootstrapping"
type: pattern
tags: [pattern, frontend, devops, vite, environment, build-pipeline, typescript]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787132260416-42b894f9
sources: ["Harvest 1787132260416 42b894f9.json"]
---

# dual-mode-multi-flavor-build-pipeline-vite-env-bootstrapping

Pola konfigurasi bundler dan bootstrapping lingkungan multi-flavor Vite untuk pemisahan build target dan resolusi endpoint runtime yang aman.

## 1. Overview & Architecture

Pola arsitektur konfigurasi build multi-flavor dan dynamic environment bootstrapping menggunakan Vite. Pola ini memisahkan konfigurasi target deployment (development vs production) melalui skrip build mode-aware, mengamankan resolusi base URL / broker URL secara runtime, serta mengaktifkan conditional debug diagnostics yang aman di level frontend.

## 2. Implementation & Code Structure

package.json (Build script flavor declarations)
├── vite.config.ts (Path aliasing & bundler plugin pipeline)
├── src/config/axios.ts (Base URL environment resolution)
└── src/pages/Alert/hooks/useStompNotifications.ts (import.meta.env.DEV debug guards)

## 3. Key Implementation Points

- Multi-flavor NPM build script matrix differentiating development and production bundle modes.
- Modern bundler pipeline integrating `@vitejs/plugin-react` with `@tailwindcss/vite` and alias resolution `@/*`.
- Defensive environment variable resolution using `import.meta.env` with fallback default values.
- Tree-shakeable development debug logging guarded by `import.meta.env.DEV`.

## 4. Code Examples

### Vite Multi-Flavor build scripts, path resolution configuration, and runtime environment variable consumption with fallbacks.

```typescript
// package.json
"scripts": {
  "dev": "vite",
  "build": "vite build",
  "build:dev": "vite build --mode development",
  "build:prod": "vite build --mode production",
  "lint": "eslint .",
  "preview": "vite preview"
}

// vite.config.ts
import path from "path"
import tailwindcss from "@tailwindcss/vite"
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
})

// src/config/axios.ts
const axiosInstance = axios.create({
  baseURL: import.meta.env.VITE_API_URL || "http://localhost:3000",
  timeout: 10000,
});
```

### Environment-gated debug logging and dynamic protocol endpoint resolution.

```typescript
// src/pages/Alert/hooks/useStompNotifications.ts
const WEBSOCKET_URL = import.meta.env.VITE_WEBSOCKET_URL;
...
if (import.meta.env.DEV) {
  console.log("WebSocket connected successfully to:", WEBSOCKET_URL);
}
```

## 5. Considerations & Best Practices

- Environment variables intended for client-side injection must be prefixed with `VITE_` to prevent leaking private host secrets.
- Sensitive keys (e.g. VITE_SECRET_KEY) injected into the bundle remain visible to reverse-engineering; avoid storing unencrypted critical backend secrets.
- Build scripts must explicitly set `--mode <env>` so Vite reads corresponding `.env.<mode>` files deterministically.

## 6. Related Knowledge

- `Vite Environment Variables and Modes`
- `Vite Tailwind v4 Bundler Plugin`
- `Client-side Secret Isolation Patterns`

## 7. Source

- Harvest 1787132260416 42b894f9.json
