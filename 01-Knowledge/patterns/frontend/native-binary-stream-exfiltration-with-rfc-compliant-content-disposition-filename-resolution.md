---
title: "Native Binary Stream Exfiltration with RFC-Compliant Content-Disposition Filename Resolution"
type: pattern
tags: [pattern, frontend, binary-stream, file-export, content-disposition, electron-main, report-service]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127443075-f8fd0716
sources: ["Harvest 1787127443075 F8fd0716.json"]
---

# Native Binary Stream Exfiltration with RFC-Compliant Content-Disposition Filename Resolution

Native Binary Stream Exfiltration with RFC-Compliant Content-Disposition Filename Resolution and Local Disk Persistence.

## 1. Overview & Architecture

A native binary stream exfiltration and local persistence service pattern that fetches large binary report payloads in the Main process via ArrayBuffer, dynamically resolves server-specified filenames using RFC-5987/6266 Content-Disposition headers with MIME-type fallbacks, and writes directly to native OS directories.

## 2. Implementation & Code Structure

src/
├── shared/
│   └── report/types.ts              # Data contracts for binary export parameters and output path
└── main/
    ├── services/
    │   └── report.service.ts        # Binary stream requester, MIME/Header parser, and FS writer
    └── handlers/
        └── report.handler.ts        # IPC invocation delegator returning local file metadata

## 3. Key Implementation Points

- Binary streaming via Axios ArrayBuffer with expanded timeout for large data exports.
- Dual-format RFC 5987 / RFC 6266 Content-Disposition header regex parsing with MIME-type fallback.
- Atomic persistence to native OS user directories (app.getPath('downloads')) using Node.js fs/promises.

## 4. Code Examples

### Report export request parameters and native filesystem result schema.

```typescript
// src/shared/report/types.ts
export interface ReportExportRequest {
  from: string
  to: string
  period: string
  token: string
}

export interface ReportExportResult {
  filename: string
  savedPath: string
}
```

### Binary arraybuffer streaming, RFC 5987/6266 header parsing, and direct OS filesystem export pipeline.

```typescript
// src/main/services/report.service.ts
const getFilenameFromHeaders = (
  contentDisposition: string | undefined,
  contentType: string,
  payload: ReportExportRequest,
): string => {
  const fallbackBaseName = `operator-reports-${payload.from}-to-${payload.to}`

  if (contentDisposition) {
    // RFC 5987 UTF-8 encoded filename resolution
    const utf8FilenameMatch = contentDisposition.match(/filename\*=UTF-8''([^;]+)/i)
    if (utf8FilenameMatch?.[1]) {
      return decodeURIComponent(utf8FilenameMatch[1])
    }

    // Standard RFC 6266 filename resolution
    const filenameMatch = contentDisposition.match(/filename="?([^";]+)"?/i)
    if (filenameMatch?.[1]) {
      return filenameMatch[1]
    }
  }

  if (contentType.includes('csv')) return `${fallbackBaseName}.csv`
  if (contentType.includes('pdf')) return `${fallbackBaseName}.pdf`
  return `${fallbackBaseName}.xlsx`
}

export const ReportService = {
  exportOperator: async (payload: ReportExportRequest): Promise<ReportExportResult> => {
    try {
      const response = await axios.get<ArrayBuffer>(
        `${API_BASE_URL}${ENDPOINTS.REPORT.OPERATOR_EXPORT}`,
        {
          headers: {
            Authorization: `Bearer ${payload.token}`,
            ...buildRequestSecurityHeaders({}),
          },
          params: {
            from: payload.from,
            to: payload.to,
            period: payload.period,
          },
          responseType: 'arraybuffer',
          timeout: 30000,
        },
      )

      const contentDisposition = response.headers['content-disposition'] as string | undefined
      const contentType = (response.headers['content-type'] as string | undefined) ?? ''
      const filename = getFilenameFromHeaders(contentDisposition, contentType, payload)
      const defaultDirectory = app.getPath('downloads')
      const defaultPath = path.join(defaultDirectory, filename)

      await writeFile(defaultPath, Buffer.from(response.data))

      return {
        filename,
        savedPath: defaultPath,
      }
    } catch (error) {
      if (axios.isAxiosError(error)) {
        const statusCode = error.response?.status
        const apiMessage =
          (error.response?.data as { message?: string } | undefined)?.message ||
          error.message ||
          'Gagal mengunduh laporan operator'

        throw new Error(statusCode ? `HTTP_STATUS_${statusCode}::${apiMessage}` : apiMessage)
      }
      throw error
    }
  },
}
```

## 5. Considerations & Best Practices

- ArrayBuffer responseType must be set in Axios to avoid UTF-8 string conversion corrupting binary spreadsheets or PDF files.
- Content-Disposition header parsing must support RFC 5987 percent-encoded UTF-8 filenames (filename*=UTF-8'') before standard quotes filename= attributes.
- Writing directly to native OS downloads path via app.getPath('downloads') avoids renderer memory exhaustion and bypasses browser download dialog restrictions.

## 6. Related Knowledge

- Electron Binary Stream File Writer
- Rfc Content Disposition Parser

## 7. Source

- Harvest 1787127443075 F8fd0716.json
