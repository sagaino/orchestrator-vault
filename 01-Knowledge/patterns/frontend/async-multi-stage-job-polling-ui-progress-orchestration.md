---
title: "Async Multi-Stage Job Polling & UI Progress Orchestration"
type: pattern
tags: [pattern, frontend, react-query, polling, async-job, toast-notifications, file-download]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787111041173-fb05651c
sources: ["Harvest 1787111041173 Fb05651c.json"]
---

# Async Multi-Stage Job Polling & UI Progress Orchestration

An async orchestration pattern combining TanStack Query mutations and polling queries with interactive toast progress and automated artifact downloading.

## 1. Overview & Architecture

A two-phase async workflow coordinating multipart job submission via TanStack Query mutation with polling query status checks, interactive progress toast, and automatic file download.

## 2. Implementation & Code Structure

src/services/generate.ts (API endpoints) -> src/services/hooks/useGenerate.tsx (TanStack Query hooks & toast orchestration)

## 3. Key Implementation Points

- Use useMutation for initiating multipart async jobs and useQuery for polling job status.
- Conditionally terminate refetchInterval when job status reaches terminal states.
- Expose an imperative jumpTo100 API from toast progress component via ref callback.
- Trigger automatic file download upon successful job completion using blob URLs.

## 4. Code Examples

### React Query polling pattern with conditional refetch interval and mutation trigger

```typescript
function useCheckStatus(promptId: string | null) {
  return useQuery<CheckStatusResponse>({
    queryKey: ['generate-status', promptId],
    queryFn: () => GenerateService.checkStatus(promptId!),
    enabled: !!promptId,
    refetchInterval: (query) => {
      const data = query.state.data;
      if (data?.status === 'completed' || data?.status === 'failed') return false;
      return 2000;
    },
  });
}

export function useGenerateVideo() {
  const generate = useGenerate();
  const [promptId, setPromptId] = useState<string | null>(null);
  const statusQuery = useCheckStatus(promptId);

  const handleGenerate = async (project: TProject) => {
    const { metadata } = buildMetadata(project);
    const images = await loadImages(project);
    const result = await generate.mutateAsync({ metadata, images });
    setPromptId(result.prompt_id);
  };

  return { handleGenerate, isGenerating: generate.isPending || !!promptId };
}
```

## 5. Considerations & Best Practices

- Clear promptId on network errors or component unmounting.
- Handle browser tab backgrounding throttling polling intervals gracefully.

## 6. Related Knowledge

- `TanStack Query Polling Strategies`
- `Sonner Custom Toast Components`
- `Programmatic Blob Downloads`

## 7. Source

- Harvest 1787111041173 Fb05651c.json
