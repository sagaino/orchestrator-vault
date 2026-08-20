---
title: "Origin Private File System (OPFS) Direct Stream Adapter"
type: pattern
tags: [pattern, frontend, storage, opfs, file-system, native-integration, binary-stream]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127791371-cc78a949
sources: ["Harvest 1787127791371 Cc78a949.json"]
---

# Origin Private File System (OPFS) Direct Stream Adapter

Origin Private File System (OPFS) Direct Stream Adapter Pattern with Storage Quota Isolation

## 1. Overview & Architecture

Provides direct streaming access to the Origin Private File System (OPFS) for zero-copy binary blob/media file storage, avoiding IndexedDB/localStorage memory overhead.

## 2. Implementation & Code Structure

OPFSAdapter implements StorageAdapter<File> with getDirectory, get, set, remove, list, clear, and isSupported static checker.

## 3. Key Implementation Points

- Direct stream pipeline via createWritable() to write binary files directly to disk without main-thread blocking.
- Safe feature detection via isSupported() to check navigator.storage.getDirectory availability.
- Resilient error handling mapping NotFoundError DOMException to null or safe no-op.

## 4. Code Examples

### OPFS direct stream file adapter implementation

```typescript
export class OPFSAdapter implements StorageAdapter<File> {
  private directoryName: string;

  constructor(directoryName = "media") {
    this.directoryName = directoryName;
  }

  private async getDirectory(): Promise<FileSystemDirectoryHandle> {
    const opfsRoot = await navigator.storage.getDirectory();
    return await opfsRoot.getDirectoryHandle(this.directoryName, {
      create: true,
    });
  }

  async get(key: string): Promise<File | null> {
    try {
      const directory = await this.getDirectory();
      const fileHandle = await directory.getFileHandle(key);
      return await fileHandle.getFile();
    } catch (error) {
      if ((error as Error).name === "NotFoundError") {
        return null;
      }
      throw error;
    }
  }

  async set({ key, value: file }: { key: string; value: File }): Promise<void> {
    const directory = await this.getDirectory();
    const fileHandle = await directory.getFileHandle(key, { create: true });
    const writable = await fileHandle.createWritable();

    await writable.write(file);
    await writable.close();
  }

  async remove(key: string): Promise<void> {
    try {
      const directory = await this.getDirectory();
      await directory.removeEntry(key);
    } catch (error) {
      if ((error as Error).name !== "NotFoundError") {
        throw error;
      }
    }
  }

  static isSupported(): boolean {
    return "storage" in navigator && "getDirectory" in navigator.storage;
  }
}
```

## 5. Considerations & Best Practices

- OPFS is bound to the browser origin and sandboxed private directory.
- Exclusive write lock exists during FileSystemWritableFileStream usage until close() is invoked.

## 6. Related Knowledge

- `W3C File System Access API & OPFS specification`
- `Storage Quota Exceeded error handling and rollback semantics`

## 7. Source

- Harvest 1787127791371 Cc78a949.json
