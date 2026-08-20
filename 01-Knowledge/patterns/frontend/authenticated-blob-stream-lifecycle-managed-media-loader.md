---
title: "Authenticated Blob Stream & Lifecycle-Managed Media Loader"
type: pattern
tags: [pattern, frontend, media, blob, memory-management, file-download, react]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787112018130-d8cc3a8b
sources: ["Harvest 1787112018130 D8cc3a8b.json"]
---

# Authenticated Blob Stream & Lifecycle-Managed Media Loader

Authenticated blob media loader and download stream with explicit memory deallocation and progress tracking.

## 1. Overview & Architecture

Pola pemuatan gambar dan file unduhan privat yang memerlukan header otentikasi HTTP dengan mengunduh binary Blob via Axios, mengonversi ke Object URL lokal, dan membersihkan alokasi memori secara eksplisit.

## 2. Implementation & Code Structure

src/
├── services/post-media.ts (Blob download method with progress tracking & Object URL cleanup)
└── pages/Gallery/components/ThumbnailImage.tsx (Component with authenticated Blob loader & URL.revokeObjectURL lifecycle hook)

## 3. Key Implementation Points

- Fetching authenticated binary streams with `responseType: 'blob'`.
- Creating localized object references via `URL.createObjectURL(blob)`.
- Guaranteed memory deallocation in component cleanup handlers via `URL.revokeObjectURL()`.
- Progress tracking for large media downloads using `onDownloadProgress`.

## 4. Code Examples

### Authenticated thumbnail component with Blob streaming and strict memory revocation on unmount.

```typescript
// Image thumbnail fetcher with Blob and Object URL cleanup
export const ThumbnailImage: React.FC<ThumbnailImageProps> = ({
  photoId,
  alt = '',
  className = '',
}) => {
  const [imageUrl, setImageUrl] = useState<string | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<boolean>(false);

  useEffect(() => {
    let isMounted = true;
    let createdUrl: string | null = null;

    const loadThumbnail = async () => {
      setLoading(true);
      setError(false);
      try {
        const endpoint = PostMediaService.getThumbnailUrl(photoId);
        const response = await axios.get(endpoint, { responseType: 'blob' });

        if (isMounted) {
          const rawContentType = response.headers['content-type'];
          const contentType = typeof rawContentType === 'string' ? rawContentType : 'image/jpeg';
          const blob = new Blob([response.data], { type: contentType });
          createdUrl = URL.createObjectURL(blob);
          setImageUrl(createdUrl);
          setLoading(false);
        }
      } catch {
        if (isMounted) {
          const baseUrl = import.meta.env.VITE_API_URL || '';
          setImageUrl(`${baseUrl}/post-media/${photoId}/thumbnail`);
          setError(true);
          setLoading(false);
        }
      }
    };

    if (photoId) {
      loadThumbnail();
    } else {
      setLoading(false);
      setError(true);
    }

    return () => {
      isMounted = false;
      if (createdUrl) {
        URL.revokeObjectURL(createdUrl);
      }
    };
  }, [photoId]);

  if (loading) return <div className="absolute inset-0 animate-pulse bg-muted/60" />;
  return <img src={imageUrl || ''} alt={alt} className={className} onError={() => setError(true)} />;
};
```

### Download service leveraging Axios blob response, download progress callback, and programmatic anchor trigger.

```typescript
// Authenticated file download with progress tracking
downloadPhoto: async (
  photoId: string,
  filename?: string,
  onProgress?: (progress: number) => void
): Promise<void> => {
  const response = await axios.get(`/post-media/${photoId}/download`, {
    responseType: "blob",
    onDownloadProgress: (progressEvent) => {
      if (progressEvent.total) {
        const percentCompleted = Math.round(
          (progressEvent.loaded * 100) / progressEvent.total
        );
        onProgress?.(percentCompleted);
      }
    },
  });
  const rawContentType = response.headers["content-type"];
  const contentType = typeof rawContentType === "string" ? rawContentType : "image/jpeg";
  const blob = new Blob([response.data], { type: contentType });
  const url = URL.createObjectURL(blob);
  const link = document.createElement("a");
  link.href = url;
  link.download = filename || `photo-${photoId}.jpg`;
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
  URL.revokeObjectURL(url);
};
```

## 5. Considerations & Best Practices

- Always revoke Object URLs (`URL.revokeObjectURL`) to prevent browser heap memory bloat when loading numerous images.
- Requires Axios client credentials rather than direct HTML `src` links when accessing protected static assets.

## 6. Related Knowledge

- `Blob API`
- `URL.createObjectURL / URL.revokeObjectURL`
- `Axios onDownloadProgress`
- `React useEffect cleanup`

## 7. Source

- Harvest 1787112018130 D8cc3a8b.json
