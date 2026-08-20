---
title: "Atomic File-Based Persistence with Schema Gating & Concurrency Control"
type: pattern
tags: [pattern, backend, persistence, atomicity, schema-validation, nodejs]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787130041232-c9802673
sources: ["Harvest 1787130041232 C9802673.json"]
---

# Atomic File-Based Persistence with Schema Gating & Concurrency Control

Pola penyimpanan state berbasis file atomik dengan pre-write schema validation dan POSIX atomic rename untuk mencegah partial writes dan status korup.

## 1. Overview & Architecture

Pola persistensi file atomik menyediakan penyimpanan data lokal yang tangguh dan tahan terhadap crash (crash-resilient) tanpa memerlukan database eksternal. Pola ini menggabungkan validasi skema sebelum penulisan ke disk dengan operasi rename POSIX atomik.

## 2. Implementation & Code Structure

src/
├── schema.mjs           # Central validation contracts for manifests and jobs
├── job-queue.mjs        # Atomic job state mutations and schema checks
└── run-manager.mjs      # Run manifest state machine and transactional updates

## 3. Key Implementation Points

- Write data to a unique temporary file (`<path>.<pid>.<uuid>.tmp`) before swapping into place.
- Validate data against explicit schema rules before writing to disk to prevent corrupted state files.
- Use POSIX-compliant `fs.renameSync` for atomic file replacement.
- Apply strict filesystem permissions (`0o600`) on runtime metadata files.

## 4. Code Examples

### Atomic file write with schema pre-flight validation and POSIX atomic rename

```javascript
import fs from "node:fs";
import path from "node:path";
import { randomUUID } from "node:crypto";
import { validateJob } from "./schema.mjs";

export function writeAtomic(filePath, value) {
  const validation = validateJob(value);
  if (!validation.valid) {
    throw new Error(`Job schema invalid: ${validation.errors.join(", ")}`);
  }
  fs.mkdirSync(path.dirname(filePath), { recursive: true });
  const temporaryPath = `${filePath}.${process.pid}.${randomUUID().slice(0, 8)}.tmp`;
  fs.writeFileSync(temporaryPath, `${JSON.stringify(value, null, 2)}\n`, {
    encoding: "utf8",
    mode: 0o600,
  });
  fs.renameSync(temporaryPath, filePath);
}
```

### Strict data validation guarding runtime state mutations

```javascript
export function validateManifest(manifest) {
  const errors = [];
  if (!manifest || typeof manifest !== "object") {
    return { valid: false, errors: ["Manifest must be a non-null object"] };
  }
  if (!manifest.schemaVersion) errors.push("Missing schemaVersion");
  if (!manifest.runId || typeof manifest.runId !== "string") errors.push("Missing or invalid runId");
  if (!manifest.state || !Object.values(RUN_STATES).includes(manifest.state)) {
    errors.push(`Invalid manifest state: ${manifest.state}`);
  }
  if (!manifest.project?.id) errors.push("Missing project.id");
  if (!manifest.project?.repository) errors.push("Missing project.repository");
  if (!manifest.task?.path) errors.push("Missing task.path");
  return { valid: errors.length === 0, errors };
}
```

## 5. Considerations & Best Practices

- Atomic rename (`fs.renameSync`) is only atomic within the same filesystem mount/volume. Ensure temporary files are created in the same parent directory.
- Always set restrictive file modes (e.g. `0o600`) when persisting sensitive job or token metadata.
- Schema validation must occur synchronously before opening file handles or writing temp files.

## 6. Related Knowledge

- File Based Persistence
- Data Integrity Guards

## 7. Source

- Harvest 1787130041232 C9802673.json
