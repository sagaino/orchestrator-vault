---
title: "Hybrid Client Storage (IndexedDB Metadata + OPFS Binary Assets)"
type: pattern
tags: [pattern, frontend, storage, indexeddb, opfs, offline-first, media-storage]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787111041173-fb05651c
sources: ["Harvest 1787111041173 Fb05651c.json"]
---

# Hybrid Client Storage (IndexedDB Metadata + OPFS Binary Assets)

A hybrid client persistence pattern combining IndexedDB for structured JSON metadata queries and OPFS for zero-overhead binary media blobs with quota guards and rollback.

## 1. Overview & Architecture

A dual-layer client persistence architecture that offloads large binary assets (blobs, images, audio, video) to the Origin Private File System (OPFS) while maintaining queryable structured metadata and project manifests in IndexedDB.

## 2. Implementation & Code Structure

src/services/storage/types.ts (StorageAdapter interface) -> src/services/storage/indexeddb-adapter.ts & opfs-adapter.ts (Implementations) -> src/services/storage/quota.ts (Quota classification) -> src/services/storage/services.ts (StorageService orchestrator)

## 3. Key Implementation Points

- Implement generic StorageAdapter<T> interface defining get, set, remove, list, and clear methods.
- Store binary files directly in OPFS via FileSystemDirectoryHandle with streaming writers.
- Store asset metadata (dimensions, size, mime type, thumbnail) in IndexedDB for fast indexing.
- Perform atomic compensation/rollback if saving to one adapter fails.
- Provide quota detection via StorageQuotaExceededError with byte requirements.

## 4. Code Examples

### StorageService coordinating dual persistence to OPFS and IndexedDB with quota detection and rollback

```typescript
class StorageService {
  private getProjectMediaAdapters({ projectId }: { projectId: string }) {
    const mediaMetadataAdapter = new IndexedDBAdapter<MediaAssetData>({
      dbName: `${this.config.mediaDb}-${projectId}`,
      storeName: "media-metadata",
    });
    const mediaAssetsAdapter = new OPFSAdapter(`media-files-${projectId}`);
    return { mediaMetadataAdapter, mediaAssetsAdapter };
  }

  async saveMediaAsset({
    projectId,
    mediaAsset,
  }: {
    projectId: string;
    mediaAsset: MediaAsset;
  }): Promise<void> {
    const { mediaMetadataAdapter, mediaAssetsAdapter } =
      this.getProjectMediaAdapters({ projectId });

    const metadata: MediaAssetData = {
      id: mediaAsset.id,
      name: mediaAsset.name,
      type: mediaAsset.type,
      size: mediaAsset.file.size,
      width: mediaAsset.width,
      height: mediaAsset.height,
      thumbnailUrl: mediaAsset.thumbnailUrl,
    };

    try {
      await mediaAssetsAdapter.set({
        key: mediaAsset.id,
        value: mediaAsset.file,
      });
      await mediaMetadataAdapter.set({
        key: mediaAsset.id,
        value: metadata,
      });
    } catch (error) {
      try {
        await mediaAssetsAdapter.remove(mediaAsset.id);
      } catch {
        // Ignore cleanup failures so original error is preserved
      }
      if (this.isQuotaExceededError({ error })) {
        throw new StorageQuotaExceededError({
          requiredBytes: mediaAsset.file.size,
        });
      }
      throw error;
    }
  }
}
```

## 5. Considerations & Best Practices

- OPFS is supported in modern browsers (Chrome, Safari 15.2+, Firefox 111+); fallback strategy needed for legacy environments.
- Object URLs created from OPFS files must be revoked when no longer needed to prevent memory leaks.
- IndexedDB database names are partitioned by project ID to allow clean atomic project deletion.

## 6. Related Knowledge

- `StorageAdapter Pattern`
- `Browser Storage Quota Management`
- `Object URL Lifecycle Management`

## 7. Source

- Harvest 1787111041173 Fb05651c.json
