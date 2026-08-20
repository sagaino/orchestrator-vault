---
title: "Multi-Engine Polyglot Persistence Store"
type: pattern
tags: [pattern, frontend, local-db, indexeddb, opfs, sharding, offline-storage]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127791371-cc78a949
sources: ["Harvest 1787127791371 Cc78a949.json"]
---

# Multi-Engine Polyglot Persistence Store

Multi-Engine Polyglot Persistence Store with Sharded Transactional Teardown

## 1. Overview & Architecture

Coordinates a hybrid multi-engine persistence strategy combining IndexedDB for queryable structured scene/metadata with OPFS for heavy binary media assets, partitioned by project.

## 2. Implementation & Code Structure

StorageService -> Global Projects IndexedDB Store -> Sharded Media Metadata IndexedDB + Sharded OPFS Directory per Project -> Cascade Deletion Pipeline.

## 3. Key Implementation Points

- Polyglot partitioning separating structured JSON metadata (IndexedDB) from raw binary media chunks (OPFS).
- Project-level DB sharding allowing single-call teardown via deleteDatabase() native request.
- Type deserialization guards converting stringified ISO dates back to Date instances with duration safety.

## 4. Code Examples

### Polyglot multi-engine storage orchestrator with sharded database isolation and transactional teardown

```typescript
class StorageService {
  private projectsAdapter: IndexedDBAdapter<SerializedProject>;
  private config: StorageConfig;

  constructor() {
    this.config = {
      projectsDb: "video-generator-projects",
      mediaDb: "video-generator-media",
    };
    this.projectsAdapter = new IndexedDBAdapter<SerializedProject>({
      dbName: this.config.projectsDb,
      storeName: "projects",
      version: 1,
    });
  }

  private getProjectMediaAdapters({ projectId }: { projectId: string }) {
    const mediaMetadataAdapter = new IndexedDBAdapter<MediaAssetData>({
      dbName: `${this.config.mediaDb}-${projectId}`,
      storeName: "media-metadata",
    });
    const mediaAssetsAdapter = new OPFSAdapter(`media-files-${projectId}`);
    return { mediaMetadataAdapter, mediaAssetsAdapter };
  }

  async saveMediaAsset({ projectId, mediaAsset }: { projectId: string; mediaAsset: MediaAsset }): Promise<void> {
    const { mediaMetadataAdapter, mediaAssetsAdapter } = this.getProjectMediaAdapters({ projectId });
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
      await mediaAssetsAdapter.set({ key: mediaAsset.id, value: mediaAsset.file });
      await mediaMetadataAdapter.set({ key: mediaAsset.id, value: metadata });
    } catch (error) {
      try { await mediaAssetsAdapter.remove(mediaAsset.id); } catch {}
      if (this.isQuotaExceededError({ error })) {
        throw new StorageQuotaExceededError({ requiredBytes: mediaAsset.file.size });
      }
      throw error;
    }
  }

  async deleteProject({ id }: { id: string }): Promise<void> {
    await this.projectsAdapter.remove(id);
    await deleteDatabase({ dbName: `${this.config.mediaDb}-${id}` });
  }
}
```

## 5. Considerations & Best Practices

- Object URLs generated from loaded files must be revoked when no longer needed to prevent memory leaks.
- Schema version upgrades in IndexedDB require consistent onupgradeneeded handlers.

## 6. Related Knowledge

- `W3C IndexedDB 3.0 API and Transactional Object Stores`
- `Database Sharding & Cascading Deletion Strategy for Offline-First Web Apps`

## 7. Source

- Harvest 1787127791371 Cc78a949.json
