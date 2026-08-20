---
title: "hwid-bound-multimodal-hardware-licensing-matrix-provisioning"
type: pattern
tags: [pattern, frontend, hardware-licensing, hwid, device-provisioning, biometrics, security]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127443075-f8fd0716
sources: ["Harvest 1787127443075 F8fd0716.json"]
---

# hwid-bound-multimodal-hardware-licensing-matrix-provisioning

HWID-bound multi-capability hardware licensing and biometric module matrix provisioning architecture.

## 1. Overview & Architecture

An enterprise hardware provisioning and biometric capability matrix pattern that maps distinct physical sensors and software capabilities to dedicated cryptographic license slots bound by unique hardware identifiers (HWID and Serial Numbers).

## 2. Implementation & Code Structure

- src/shared/device/types.ts (Hardware and capability license domain models)
- src/shared/license/types.ts (License allocation & validation models)
- src/main/services/device.service.ts (Provisioning & hardware binding operations)
- src/main/services/license.service.ts (Modality-based license filtering and allocation)

## 3. Key Implementation Points

- Granular multi-slot biometric and scanner capability provisioning per hardware terminal (FACE, FINGERPRINT, KTP, PASSPORT, APPLICATION).
- State machine tracking license availability ('is_used', 'is_active', 'device_license_is_active') to prevent double-binding.
- Cryptographic request signing wrapping all device hardware provisioning requests.

## 4. Code Examples

### Device hardware metadata schema binding hardware identity (HWID, serial_number) to capability-specific license slots

```typescript
// src/shared/device/types.ts
export interface DeviceRecord {
  code: string
  hwid: string
  serial_number: string
  type: string
  is_init: boolean
  is_active: boolean
  deleted: boolean
  status_txt: string
  division: DeviceDetailDivision | null
  license_list: DeviceLicenseRecord[]
}

export interface DeviceLicenseRecord {
  code: string
  keys: string
  name: string
  type: 'APPLICATION' | 'FACE' | 'FINGERPRINT' | string
  owner: string
  software: string
  description: string
  is_active: boolean
  is_used: boolean
  device_license_is_active: boolean | null
}

export interface DeviceCreateRequest {
  serial_number: string
  license_application: string
  license_face: string
  license_fingerprint: string
  license_ktp: string
  license_passport: string
  is_active: boolean
  division_code: string
  face?: string
  token: string
}
```

### Device provisioning service binding individual hardware modules to signed cryptographic payload dispatch

```typescript
// src/main/services/device.service.ts
export const DeviceService = {
  create: async (payload: DeviceCreateRequest): Promise<DeviceCreateResponse> => {
    try {
      const requestBody = {
        serial_number: payload.serial_number,
        license_application: payload.license_application,
        license_face: payload.license_face,
        license_fingerprint: payload.license_fingerprint,
        license_ktp: payload.license_ktp,
        license_passport: payload.license_passport,
        is_active: payload.is_active,
        division_code: payload.division_code,
        ...(payload.face ? { face: payload.face } : {}),
      }

      const response = await axios.post<DeviceCreateResponse>(
        `${API_BASE_URL}${ENDPOINTS.DEVICE.CREATE}`,
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
        const apiMessage =
          (error.response?.data as { message?: string } | undefined)?.message ||
          error.message ||
          'Gagal menambahkan perangkat'

        throw new Error(
          statusCode ? `HTTP_STATUS_${statusCode}::${apiMessage}` : apiMessage
        )
      }

      throw error
    }
  },
}
```

## 5. Considerations & Best Practices

- Hardware ID (HWID) and Serial Number form an immutable cryptographic tuple for field device verification.
- Granular license slots prevent unauthorized hardware module activation (e.g. fingerprint or passport OCR scanner) without valid capability keys.
- Signed security headers prevent tampering with license allocation payloads during provisioning.

## 6. Related Knowledge

- `Hardware-Locked Licensing Architecture (HWID / Dongle binding)`
- `Biometric Modality Capabilities Provisioning`
- `Cryptographic Request Signing (HMAC-SHA256 Payload Canonicalization)`

## 7. Source

- Harvest 1787127443075 F8fd0716.json
