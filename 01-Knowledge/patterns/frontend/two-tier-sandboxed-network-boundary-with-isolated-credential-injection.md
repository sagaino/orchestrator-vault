---
title: "two-tier-sandboxed-network-boundary-with-isolated-credential-injection"
type: pattern
tags: [pattern, frontend, security, api-gateway, token-injection, electron-main, sandbox]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127443075-f8fd0716
sources: ["Harvest 1787127443075 F8fd0716.json"]
---

# two-tier-sandboxed-network-boundary-with-isolated-credential-injection

Two-tier sandboxed network boundary with main-process isolated HTTP gateway and per-request JIT credential injection.

## 1. Overview & Architecture

Untuk memenuhi standar keamanan tinggi, Renderer Process (Chromium DOM Sandbox) diisolasi sepenuhnya dari jaringan luar (tidak ada request Axios/Fetch langsung dari browser window ke API backend). Semua interaksi jaringan diarahkan melalui IPC ke Node.js Main Process yang berfungsi sebagai isolated API Gateway. Renderer Service bertugas membaca token terenkripsi secara just-in-time dari penyimpanan lokal, membongkar parameter, dan mendelegasikan eksekusi HTTP lengkap ke Main Process.

## 2. Implementation & Code Structure

src/
├── renderer/src/services/
│   └── operator.ts             # Mengambil token terenkripsi, memvalidasi keberadaan sesi, memanggil window.api
├── preload/modules/
│   └── operator.ts             # Typed IPC bridge (ipcRenderer.invoke)
├── main/handlers/
│   └── operator.handler.ts     # IPC entrypoint & payload forwarding
└── main/services/
    └── operator.service.ts     # Axios instance, Bearer token injection, request signing, response normalization

## 3. Key Implementation Points

- Renderer process sepenuhnya terisolasi dari Axios/Fetch langsung ke internet.
- Renderer service mengekstrak token dari encrypted storage just-in-time sebelum dispatch IPC.
- Penggunaan TypeScript Omit<T, 'token'> di level UI service agar komponen React terbebas dari manajemen token.
- Main process bertindak sebagai API Gateway tunggal yang menyuntikkan Bearer token, security headers, dan timeout enforcement.

## 4. Code Examples

### JIT token extraction in renderer service and isolated execution in main process gateway

```typescript
// 1. Renderer Layer: JIT Token Extraction & IPC Invocation (src/renderer/src/services/operator.ts)
type OperatorCreatePayload = Omit<OperatorCreateRequest, 'token'>

export const OperatorService = {
  create: async (payload: OperatorCreatePayload): Promise<OperatorCreateResponse> => {
    const token = getData(LOCALSTORAGE_KEY.TOKEN)
    if (!token || typeof token !== 'string') {
      throw new Error('Sesi login tidak ditemukan. Silakan login ulang.')
    }

    return invokeIpcWithAuthHandling(
      () => window.api.operator.create({ ...payload, token }),
      {
        methodName: 'operator:create',
        fallbackMessage: 'Gagal menambahkan operator',
      }
    )
  }
}

// 2. Main Process Layer: Gateway Execution & Security Injection (src/main/services/operator.service.ts)
export const OperatorService = {
  create: async (payload: OperatorCreateRequest): Promise<OperatorCreateResponse> => {
    try {
      const requestBody = {
        name: payload.name,
        email: payload.email,
        phone: payload.phone,
        nrp: payload.nrp,
        nik: payload.nik,
        is_active: payload.is_active,
        divisi_code: payload.divisi_code,
        ...(payload.device_code_list ? { device_code_list: payload.device_code_list } : {}),
        ...(payload.face ? { face: payload.face } : {}),
      }

      const response = await axios.post<OperatorCreateResponse>(
        `${API_BASE_URL}${ENDPOINTS.OPERATOR.CREATE}`,
        requestBody,
        {
          headers: {
            Authorization: `Bearer ${payload.token}`,
            ...buildRequestSecurityHeaders(requestBody),
          },
          timeout: 15000,
        },
      )
      return response.data
    } catch (error) {
      if (axios.isAxiosError(error)) {
        const statusCode = error.response?.status
        const apiMessage = (error.response?.data as { message?: string })?.message || error.message
        throw new Error(statusCode ? `HTTP_STATUS_${statusCode}::${apiMessage}` : apiMessage)
      }
      throw error
    }
  }
}
```

## 5. Considerations & Best Practices

- Keamanan maksimum dengan isolasi total network stack dari DOM sandbox.
- Mengeliminasi masalah CORS browser karena HTTP client berjalan di lingkungan Node.js murni.
- Membutuhkan serialisasi IPC untuk payload berukuran besar (misalnya string gambar base64).

## 6. Related Knowledge

- `patterns/electron/two-tier-api-gateway.md`
- `concepts/zero-trust-renderer-architecture.md`
- `patterns/frontend/typed-ipc-contracts.md`

## 7. Source

- Harvest 1787127443075 F8fd0716.json
