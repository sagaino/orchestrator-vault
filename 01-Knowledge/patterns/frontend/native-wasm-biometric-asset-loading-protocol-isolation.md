---
title: "Native WASM Biometric Asset Loading & Protocol Isolation"
type: pattern
tags: [pattern, frontend, electron, wasm, biometrics, protocol-handling]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787112360086-3c0f5be1
sources: ["Harvest 1787112360086 3c0f5be1.json"]
---

# Native WASM Biometric Asset Loading & Protocol Isolation

Protocol-aware WASM asset loader resolving binaries via custom app protocols in Electron packages and standard paths in web dev servers.

## 1. Overview & Architecture

Dual-mode protocol resolution mechanism for loading large WebAssembly biometric models seamlessly across both Vite development servers and packaged Electron applications.

## 2. Implementation & Code Structure

public/
└── sam.wasm                   # Biometric WASM binary engine
src/
├── preload/
│   └── index.ts              # Protocol-aware WASM URL resolver
└── renderer/
    └── src/lib/              # Face & biometric verification services consuming window.electron.getFaceCaptureWasmUrl()

## 3. Key Implementation Points

- Detects execution protocol (`window.location.protocol !== 'file:'`).
- Routes to custom Electron resource protocol scheme (`app-resource://`) in production packages.
- Exposes helper method securely via Preload Context Bridge without exposing file system paths.

## 4. Code Examples

### Protocol-aware WASM asset locator bridge for desktop and web runtime environments

```typescript
// src/preload/index.ts
const getFaceCaptureWasmUrl = (): string => {
  if (window.location.protocol !== 'file:') {
    return '/sam.wasm'
  }

  return 'app-resource://sam.wasm'
}

const extendedElectronApi = {
  ...electronAPI,
  ...wsApi,
  getFaceCaptureWasmUrl
}

if (process.contextIsolated) {
  contextBridge.exposeInMainWorld('electron', extendedElectronApi)
}
```

## 5. Considerations & Best Practices

- Packaged Electron applications serving content from `file://` must route heavy binary assets through registered custom protocols (`app-resource://`) to bypass CSP and origin restrictions.
- Development mode under Vite dev server continues to serve the WASM file directly from root (`/sam.wasm`).

## 6. Related Knowledge

- Custom Protocols Asset Loading
- Wasm Biometrics Integration

## 7. Source

- Harvest 1787112360086 3c0f5be1.json
