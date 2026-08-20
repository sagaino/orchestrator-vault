---
title: "dual-phase long-running job pipeline with multipart asset packing & progress emulation"
type: pattern
tags: [pattern, frontend, long-polling, react-query, multipart, async-jobs, media-generation]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127791371-cc78a949
sources: ["Harvest 1787127791371 Cc78a949.json"]
---

# dual-phase long-running job pipeline with multipart asset packing & progress emulation

Dual-Phase Long-Running Job Pipeline with Multipart Asset Packing & Progress Emulation orchestrates background asynchronous tasks with dynamic polling intervals and simulated progress UX.

## 1. Overview & Architecture

An asynchronous job lifecycle coordinator designed for long-running AI video/media generation pipelines combining multi-modal binary compilation, reactive status polling, and UX progress emulation.

## 2. Implementation & Code Structure

src/services/generate.ts -> Multi-part compiler, polling endpoint & blob downloader. src/services/hooks/useGenerate.tsx -> Polling state coordinator, simulated progress toast & auto-download trigger.

## 3. Key Implementation Points

- Parallel OPFS asset retrieval combining image and audio binaries with serialized JSON metadata into FormData.
- Conditional refetchInterval lifecycle governed by reactive React Query query state.
- Separation of concerns between low-level HTTP transport (GenerateService) and reactive UI state coordination (useGenerateVideo).

## 4. Code Examples

### Dual-phase long-running job pipeline with multipart compilation, dynamic polling interval, and animated progress emulation

```typescript
// 1. Service Definition (src/services/generate.ts)
export interface GenerateRequest {
  global_prompt: string
  segments: GenerateSegment[]
  audio_segments?: AudioSegment[]
  duration_seconds: number
  fps: number
  seed?: number
  inpaint_audio: boolean
}

export interface CheckStatusResponse {
  prompt_id: string
  status: 'queued' | 'processing' | 'completed' | 'failed'
  progress: number | null
  output_filename: string | null
}

export const GenerateService = {
  generate: async ({ metadata, images, audios }: { metadata: GenerateRequest; images: File[]; audios?: File[] }) => {
    const formData = new FormData()
    formData.append('metadata', JSON.stringify(metadata))
    images.forEach((file) => formData.append('images', file))
    audios?.forEach((file) => formData.append('audios', file))
    const response = await axios.post<GenerateResponse>(ENDPOINTS.GENERATE, formData, {
      headers: { 'Content-Type': 'multipart/form-data' },
    })
    return response.data
  },
  checkStatus: async (promptId: string) => {
    const url = ENDPOINTS.CHECK_STATUS.replace(':prompt_id', promptId)
    const response = await axios.get<CheckStatusResponse>(url)
    return response.data
  },
  download: async (filename: string) => {
    const url = ENDPOINTS.DOWNLOAD.replace(':filename', filename)
    const response = await axios.get(url, { responseType: 'blob' })
    const blob = new Blob([response.data])
    const downloadUrl = URL.createObjectURL(blob)
    const a = document.createElement('a')
    a.href = downloadUrl
    a.download = filename
    a.click()
    URL.revokeObjectURL(downloadUrl)
  }
}

// 2. React Query Polling & Progress Emulation Hook (src/services/hooks/useGenerate.tsx)
export function useGenerateVideo() {
  const generate = useMutation({ mutationFn: GenerateService.generate })
  const [promptId, setPromptId] = useState<string | null>(null)
  
  const statusQuery = useQuery<CheckStatusResponse>({
    queryKey: ['generate-status', promptId],
    queryFn: () => GenerateService.checkStatus(promptId!),
    enabled: !!promptId,
    refetchInterval: (query) => {
      const data = query.state.data
      if (data?.status === 'completed' || data?.status === 'failed') return false
      return 2000
    },
  })

  // Smooth progress emulation with instant 100% completion jump
  useEffect(() => {
    const data = statusQuery.data
    if (!data || !promptId) return
    if (data.status === 'completed') {
      progressApiRef.current?.jumpTo100()
      if (data.output_filename) GenerateService.download(data.output_filename)
      toast.success('Video generation complete!')
    }
  }, [statusQuery.data, promptId])

  return { handleGenerate, isGenerating }
}
```

## 5. Considerations & Best Practices

- Uses dynamic refetchInterval in TanStack Query to automatically stop polling once status is 'completed' or 'failed', preventing zombie intervals.
- ProgressToast component provides continuous visual feedback via asymptotic easing up to 80% while waiting for long async generative jobs, then jumps instantly to 100% on completion.
- Automatic blob object URL lifecycle management (creation, click simulation, revocation) ensures clean memory disposal.

## 6. Related Knowledge

- [[01-Knowledge/patterns/frontend/origin-private-file-system-opfs-direct-stream-adapter.md]]
- [[01-Knowledge/patterns/frontend/multi-engine-polyglot-persistence-store.md]]

## 7. Source

- Harvest 1787127791371 Cc78a949.json
