---
title: "Operator Constraints for starter-app"
type: decision
tags: [decision, operator-constraint, starter-app]
created: 2026-09-02
updated: 2026-09-02
sources: ["02-Projects/starter-app/project.md"]
provenance_schema: 1
confidence: 1.0
owner: "local:sagaino"
orchestrator_run: "FE-034"
review_by: 2027-03-01
supersession: "ACTIVE"
---

# Operator Constraints for starter-app

## 1. Konteks Koreksi (Review Feedback)

### Feedback pada Task `FE-030` (Create UserProfileBadge Component):
> komponen src/components/UserProfileBadge.tsx tidak di pasang di src/pages/Dashboard/index.tsx

### Feedback pada Task `FE-034` (Create UserProfileBadge Component):
> Wajib gunakan varian badge "outline" dan tambahkan animasi titik hijau (animate-pulse) untuk status online. Jangan gunakan warna solid.

## 2. Aturan & Batasan Wajib (Mandatory Constraint)

- Seluruh task berikutnya pada project `starter-app` wajib mematuhi instruksi di atas.
- Hindari pola atau arsitektur yang telah ditolak oleh operator.
