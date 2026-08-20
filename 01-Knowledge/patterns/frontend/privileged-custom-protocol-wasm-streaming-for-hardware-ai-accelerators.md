---
title: "privileged-custom-protocol-wasm-streaming-for-hardware-ai-accelerators"
type: pattern
tags: [pattern, frontend, electron, webassembly, biometrics, custom-protocol, innovatrics, camera]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127443075-f8fd0716
sources: ["Harvest 1787127443075 F8fd0716.json"]
---

# privileged-custom-protocol-wasm-streaming-for-hardware-ai-accelerators

Privileged custom protocol streaming WebAssembly AI models (sam.wasm) across Electron boundaries to power client-side biometric camera auto-capture.

## 1. Overview & Architecture

A secure native WASM resource streaming and custom web component integration pattern that allows packaged Electron applications to load and execute heavy AI/biometric models (Innovatrics SAM WebAssembly) under strict CSP and context isolation boundaries without file:// protocol security restrictions.

## 2. Implementation & Code Structure

- src/main/index.ts (protocol.registerSchemesAsPrivileged, registerAppResourceProtocol, net.fetch)
- src/preload/index.ts (getFaceCaptureWasmUrl, contextBridge exposure)
- src/renderer/index.html (Content Security Policy for wasm-unsafe-eval and app-resource:)
- src/renderer/src/pages/UserManagement/components/FaceCamera.tsx (Dynamic import of Innovatrics web component and sam.wasm binding)

## 3. Key Implementation Points

- Registering 'app-resource' scheme as privileged with standard, secure, supportFetchAPI, and corsEnabled before app.whenReady().
- Serving local filesystem resources via net.fetch with file:// URLs resolved from process.resourcesPath.
- Preload environment sniffing between development ('/sam.wasm') and packaged production ('app-resource://sam.wasm').
- Asynchronous dynamic import of @innovatrics/dot-face-auto-capture/main.js and web component hydration.

## 4. Code Examples

### Privileged scheme registration and streaming handler for native WebAssembly binary in Electron Main

```typescript
// src/main/index.ts
protocol.registerSchemesAsPrivileged([
  {
    scheme: 'app-resource',
    privileges: {
      standard: true,
      secure: true,
      supportFetchAPI: true,
      corsEnabled: true,
    },
  },
])

function registerAppResourceProtocol(): void {
  protocol.handle('app-resource', (request) => {
    const requestUrl = new URL(request.url)
    const pathname = requestUrl.pathname.replace(/^\/+/, '')
    const assetPath = pathname || requestUrl.hostname

    if (assetPath !== 'sam.wasm') {
      return new Response('Not found', { status: 404 })
    }

    const wasmFileUrl = pathToFileURL(join(process.resourcesPath, 'sam.wasm')).toString()
    return net.fetch(wasmFileUrl)
  })
}
```

### Dynamic protocol resolution in Preload script providing environment-aware WASM URL

```typescript
// src/preload/index.ts
const getFaceCaptureWasmUrl = (): string => {
  if (window.location.protocol !== 'file:') {
    return '/sam.wasm'
  }
  return 'app-resource://sam.wasm'
}

const extendedElectronApi: PreloadElectronAPI = {
  ...electronAPI,
  ...wsApi,
  getFaceCaptureWasmUrl
}
```

### Web component lifecycle bridge and WebAssembly initialization for biometric face auto-capture

```tsx
// src/renderer/src/pages/UserManagement/components/FaceCamera.tsx
import { useEffect, useRef } from 'react';

export type FaceCallback = (data: Blob | { imageData: string | Blob }) => void;

export interface FaceCameraProps {
  imageType: 'png' | 'jpg';
  cameraFacing: 'user' | 'environment';
  photoTakenCb: FaceCallback;
  samWasmUrl: string;
}

const samWasmUrl = window.electron.getFaceCaptureWasmUrl();

const FaceCamera: React.FC<FaceCameraProps> = (props: FaceCameraProps) => {
  const ref = useRef<HTMLFaceCaptureElement>(null);

  useEffect(() => {
    window.DOT_DEBUG_MODE = false;

    const loadComponent = async () => {
      await import('@innovatrics/dot-face-auto-capture/main.js');
      await customElements.whenDefined('x-dot-face-auto-capture');

      if (ref.current) {
        ref.current.cameraOptions = {
          imageType: props.imageType === 'jpg' ? 'jpeg' : 'png',
          cameraFacing: props.cameraFacing,
          photoTakenCb: props.photoTakenCb,
          samWasmUrl: props.samWasmUrl,
        };
      }
    };

    loadComponent();
  }, [props.cameraFacing, props.imageType, props.photoTakenCb, props.samWasmUrl]);

  return <x-dot-face-auto-capture ref={ref} id="x-dot-face-auto-capture" className="block h-full w-full" />;
};
```

## 5. Considerations & Best Practices

- WASM binary size (sam.wasm) is large; streaming via net.fetch from process.resourcesPath avoids buffering entire binary into memory.
- Content Security Policy in index.html must explicitly allow connect-src app-resource: and script-src 'wasm-unsafe-eval' for WASM compilation under strict sandbox.
- Custom elements in React require customElements.whenDefined and dynamic DOM property assignment (ref.current.cameraOptions).

## 6. Related Knowledge

- `Electron Custom Privileged Schemes (protocol.handle, protocol.registerSchemesAsPrivileged)`
- `WebAssembly Compilation & Streaming (WebAssembly.instantiateStreaming, CSP wasm-unsafe-eval)`
- `Innovatrics DOT (Digital Onboarding Toolkit) Face Auto Capture Web Component`

## 7. Source

- Harvest 1787127443075 F8fd0716.json
