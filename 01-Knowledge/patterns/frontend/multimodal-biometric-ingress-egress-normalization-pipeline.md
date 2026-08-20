---
title: "multimodal-biometric-ingress-egress-normalization-pipeline"
type: pattern
tags: [pattern, frontend, biometrics, data-pipeline, url-normalization, data-transformation]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127443075-f8fd0716
sources: ["Harvest 1787127443075 F8fd0716.json"]
---

# multimodal-biometric-ingress-egress-normalization-pipeline

Multimodal biometric data ingress/egress normalization pipeline with idempotent URL mapping and optional payload transformation.

## 1. Overview & Architecture

Aplikasi verifikasi identitas memproses data biometrik multimodal (foto wajah Base64, template sidik jari WSQ/ISO, dan asset URL). Pada layer gateway service di Main process, data hasil fetch dari server mengalami normalisasi URL absolut (normalizeFaceUrl), sementara data mutasi (upload/update) ditransformasikan secara aman menjadi struktur payload seragam dengan dynamic optional fields.

## 2. Implementation & Code Structure

src/main/
├── utils/
│   └── normalize-face-url.ts       # Base URL prepender & path cleaner
└── services/
    ├── operator.service.ts         # Response mapper untuk list & detail
    └── device.service.ts           # Transformation biometrik perangkat

## 3. Key Implementation Points

- Idempotent prefixing: mendeteksi http:// atau https:// sebelum menambahkan base URL.
- Deep mapping array data biometrik (face_list, fingerprint_list) pada service egress level.
- Dynamic optional payload merging untuk atribut biometrik pada ingress level (...(payload.face ? { face: payload.face } : {})).

## 4. Code Examples

### Biometric URL normalizer and structural response mapping pipeline

```typescript
// src/main/utils/normalize-face-url.ts
const IMAGE_BASE_URL = process.env.IMAGE_BASE_URL || 'http://localhost:8080/storage'

export const normalizeFaceUrl = (faceUrl: string): string => {
  if (!faceUrl) return ''
  if (faceUrl.startsWith('http://') || faceUrl.startsWith('https://')) return faceUrl

  const baseUrl = IMAGE_BASE_URL.replace(/\/+$/, '')
  const normalizedPath = faceUrl.startsWith('/') ? faceUrl : `/${faceUrl}`

  return `${baseUrl}${normalizedPath}`
}

// Ingress mapping in detail query (src/main/services/operator.service.ts)
return {
  ...response.data,
  result: {
    ...response.data.result,
    face_list: response.data.result.face_list.map((face) => ({
      ...face,
      face: normalizeFaceUrl(face.face),
    })),
    fingerprint_list: response.data.result.fingerprint_list.map((fingerprint) => ({
      ...fingerprint,
      fingerprint: normalizeFaceUrl(fingerprint.fingerprint),
    })),
  },
}
```

## 5. Considerations & Best Practices

- Renderer menerima URL gambar yang 100% siap konsumsi tanpa dependensi terhadap env IMAGE_BASE_URL di renderer bundle.
- Idempotent normalization mencegah URL ganda jika backend sewaktu-waktu mengembalikan full URI.

## 6. Related Knowledge

- `patterns/data-transformation/biometric-normalization-pipeline.md`
- `concepts/electron-asset-resolution.md`

## 7. Source

- Harvest 1787127443075 F8fd0716.json
