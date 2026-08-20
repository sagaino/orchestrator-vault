---
title: "Hybrid Client-Side Storage Adapter Pattern (IndexedDB + OPFS)"
type: pattern
tags: [pattern, frontend, storage, indexeddb, opfs, offline-first, architecture]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787107893306-92ee562e
sources: ["Harvest 1787107893306 92ee562e.json"]
---

# Hybrid Client-Side Storage Adapter Pattern (IndexedDB + OPFS)

Pemisahan persistensi metadata dan binary file pada browser menggunakan IndexedDB dan Origin Private File System (OPFS) dengan proteksi transaksi kompensasi dan kuota.

## 1. Overview & Architecture

Pattern ini mengintegrasikan IndexedDB untuk data metadata terstruktur (proyek, scenes, tracks) dan Origin Private File System (OPFS) untuk aset biner berukuran besar (video, gambar, audio) dengan antarmuka StorageAdapter yang seragam. Dilengkapi dengan mekanisme rollback kompensasi dan deteksi storage quota.

## 2. Implementation & Code Structure

src/services/storage/
├── types.ts (StorageAdapter interface & domain schemas)
├── indexeddb-adapter.ts (IndexedDB generic adapter)
├── opfs-adapter.ts (OPFS binary storage adapter)
├── quota.ts (Storage quota error detection & wrapper)
└── services.ts (StorageService orchestrator with compensation logic)

## 3. Key Implementation Points

- Definisi kontrak generic StorageAdapter<T> untuk operasi CRUD storage terstandarisasi.
- Penyimpanan biner langsung ke Origin Private File System (OPFS) untuk performa I/O tinggi dan menghindari konsumsi RAM berlebih.
- Penyimpanan skema metadata relasional/dokumen ke IndexedDB dengan isolasi transaksi readonly dan readwrite.
- Pola compensating transaction: jika penulisan metadata gagal, file biner di OPFS segera dihapus (rollback).
- Deteksi StorageQuotaExceededError untuk memberitahu batas kapasitas browser secara elegan.

## 4. Code Examples

### Storage Service Orchestration dengan Compensating Rollback & Quota Guard

```typescript
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
    try {
      await mediaAssetsAdapter.remove(mediaAsset.id); // Compensating rollback: clean up orphaned binary
    } catch {
      // Preserve original error
    }
    if (this.isQuotaExceededError({ error })) {
      throw new StorageQuotaExceededError({ requiredBytes: mediaAsset.file.size });
    }
    throw error;
  }
}
```

### Generic IndexedDB Adapter Implementation

```typescript
export class IndexedDBAdapter<T> implements StorageAdapter<T> {
  private dbName: string;
  private storeName: string;
  private version: number;

  private async getDB(): Promise<IDBDatabase> {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, this.version);
      request.onerror = () => reject(request.error);
      request.onsuccess = () => resolve(request.result);
      request.onupgradeneeded = (event) => {
        const db = (event.target as IDBOpenDBRequest).result;
        if (!db.objectStoreNames.contains(this.storeName)) {
          db.createObjectStore(this.storeName, { keyPath: "id" });
        }
      };
    });
  }

  async set({ key, value }: { key: string; value: T }): Promise<void> {
    const db = await this.getDB();
    const transaction = db.transaction([this.storeName], "readwrite");
    const store = transaction.objectStore(this.storeName);
    return new Promise((resolve, reject) => {
      const request = store.put({ id: key, ...value });
      request.onerror = () => reject(request.error);
      request.onsuccess = () => resolve();
    });
  }
}
```

## 5. Considerations & Best Practices

- Browser compatibility: OPFS memerlukan browser modern (Chromium 86+, Safari 15.2+, Firefox 111+).
- Pembersihan object URL via URL.revokeObjectURL saat unmount untuk mencegah memory leaks.
- Kapasitas penyimpanan lokal dibatasi kuota browser, sehingga penanganan StorageQuotaExceededError mutlak diperlukan.

## 6. Related Knowledge

- `IndexedDB API`
- `Origin Private File System (OPFS)`
- `Compensating Transactions`
- `Storage Quota API`

## 7. Source

- Harvest 1787107893306 92ee562e.json
