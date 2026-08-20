---
title: "Project-Partitioned Multi-Tier Local Storage Isolation with Rollback-Guarded Quota Resilience"
type: pattern
tags: [pattern, frontend, security, storage-isolation, opfs, indexeddb, quota-management]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127791371-cc78a949
sources: ["Harvest 1787127791371 Cc78a949.json"]
---

# Project-Partitioned Multi-Tier Local Storage Isolation with Rollback-Guarded Quota Resilience

Project-Partitioned Multi-Tier Local Storage Isolation with Rollback-Guarded Quota Resilience.

## 1. Overview & Architecture

A partitioned client-side multi-tier storage isolation architecture that pairs fast binary media storage in the Origin Private File System (OPFS) with structured metadata in project-namespaced IndexedDB databases. Includes defensive write-rollback compensation and domain-level storage quota error classification.

## 2. Implementation & Code Structure

src/
└── services/
    └── storage/
        ├── services.ts            # StorageService facade coordinating partitioned stores
        ├── indexeddb-adapter.ts   # IndexedDB key-value metadata store
        ├── opfs-adapter.ts        # Origin Private File System binary asset adapter
        ├── quota.ts               # StorageQuotaExceededError and quota detector
        └── types.ts               # Storage interfaces and serialization schemas

## 3. Key Implementation Points

- Project-partitioned namespacing (`mediaDb-${projectId}` and `media-files-${projectId}`) providing complete tenant isolation within client storage.
- Dual-tier storage segregation separating high-speed raw file binary streams (OPFS) from queryable JSON metadata (IndexedDB).
- Defensive compensating rollback removing partial binary writes when downstream metadata persistence fails.
- Browser DOMException mapping (`QuotaExceededError`) converting low-level storage limits into actionable domain exceptions with byte calculation.
- Atomic project lifecycle eviction destroying metadata catalogs and database connections upon project deletion.

## 4. Code Examples

### StorageService implementing project-partitioned IndexedDB & OPFS adapters with quota rollback

```typescript
import type { TProject } from "@/types/project";
import { IndexedDBAdapter, deleteDatabase } from "./indexeddb-adapter";
import type { MediaAsset, MediaAssetData, SerializedProject, StorageConfig } from "./types";
import { OPFSAdapter } from "./opfs-adapter";
import { isStorageQuotaExceededError, StorageQuotaExceededError } from "./quota";

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

  // Factory providing project-partitioned storage adapters
  private getProjectMediaAdapters({ projectId }: { projectId: string }) {
    const mediaMetadataAdapter = new IndexedDBAdapter<MediaAssetData>({
      dbName: `${this.config.mediaDb}-${projectId}`,
      storeName: "media-metadata",
    });

    const mediaAssetsAdapter = new OPFSAdapter(`media-files-${projectId}`);

    return { mediaMetadataAdapter, mediaAssetsAdapter };
  }

  isQuotaExceededError({ error }: { error: unknown }): boolean {
    return isStorageQuotaExceededError({ error });
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
      // Step 1: Write raw blob to Origin Private File System
      await mediaAssetsAdapter.set({
        key: mediaAsset.id,
        value: mediaAsset.file,
      });

      // Step 2: Write structured metadata to project-scoped IndexedDB
      await mediaMetadataAdapter.set({
        key: mediaAsset.id,
        value: metadata,
      });
    } catch (error) {
      // Rollback: delete orphaned binary file on failure
      try {
        await mediaAssetsAdapter.remove(mediaAsset.id);
      } catch {
        // Ignore cleanup failures to preserve original error
      }

      if (this.isQuotaExceededError({ error })) {
        throw new StorageQuotaExceededError({
          requiredBytes: mediaAsset.file.size,
        });
      }

      throw error;
    }
  }

  async deleteProject({ id }: { id: string }): Promise<void> {
    await this.projectsAdapter.remove(id);
    await deleteDatabase({ dbName: `${this.config.mediaDb}-${id}` });
  }
}

export const storageService = new StorageService();
```

### Storage quota error classifier and domain exception definition

```typescript
export class StorageQuotaExceededError extends Error {
  readonly requiredBytes?: number;

  constructor({ requiredBytes }: { requiredBytes?: number } = {}) {
    super("Storage quota exceeded. Please free up space.");
    this.name = "StorageQuotaExceededError";
    this.requiredBytes = requiredBytes;
  }
}

export const isStorageQuotaExceededError = ({ error }: { error: unknown }): boolean => {
  if (error instanceof StorageQuotaExceededError) return true;
  if (error instanceof DOMException && (error.name === "QuotaExceededError" || error.code === 22)) {
    return true;
  }
  return false;
};
```

## 5. Considerations & Best Practices

- OPFS is only supported in modern secure contexts (HTTPS/localhost); fallback strategies (IndexedDB blobs or memory) should be provided for legacy browsers.
- Deleting a project requires deleting both the IndexedDB database instance and purging the OPFS folder hierarchy to prevent orphaned storage leaks.
- Concurrent writes to the same project asset must handle OPFS file handle locks carefully.

## 6. Related Knowledge

- [[01-Knowledge/patterns/frontend/centralized-domain-store-with-asynchronous-storage-sidecar-sync.md]]
- Opfs Vs Indexeddb
- Compensating Transactions

## 7. Source

- Harvest 1787127791371 Cc78a949.json
