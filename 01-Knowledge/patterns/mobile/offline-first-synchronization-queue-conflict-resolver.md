---
title: "Offline-First Synchronization & Queue Conflict Resolver"
type: pattern
tags: [pattern, mobile, flutter, drift, sqlite, offline-first, sync, queue, conflict-resolution]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787110568549-448b547b
sources: ["Harvest 1787110568549 448b547b.json"]
---

# Offline-First Synchronization & Queue Conflict Resolver

Offline-First Synchronization & Queue Conflict Resolver pattern in Flutter with Drift SQLite and ConnectivityPlus.

## 1. Overview & Architecture

Pola arsitektur offline-first untuk transaksi finansial mobile menggunakan local SQLite queue, pendeteksi konektivitas reaktif, dan strategi rekonsiliasi data (conflict resolution) dengan rollback otomatis pada kegagalan.

```text
[User Action / Transfer] ────(Offline)────► [Insert PendingTransfers Table]
        │                                             │
     (Online)                                         ▼
        │                                   [Optimistic Local Balance]
        ▼                                             │
[Submit Remote API] ◄──(Connectivity Resumed)─────────┘
        ├─► (Success) ──► Mark 'synced' + Insert Transactions Table
        └─► (Failure) ──► Mark 'failed' + Rollback Balance + Notify User
```

## 2. Implementation & Code Structure

lib/core/service/sync_service.dart
lib/core/storage/database.dart
lib/core/storage/local_storage.dart
lib/core/sync/conflict_resolver.dart
lib/core/sync/account_conflict_resolver.dart
lib/core/sync/transaction_conflict_resolver.dart
lib/core/sync/contact_conflict_resolver.dart

## 3. Key Implementation Points

- Drift SQLite table for pending transfers acting as persistent offline queue
- Reactive Connectivity listener triggers automatic sync when device transitions from offline to online
- Server-Wins and timestamp-based conflict reconciliation via generic ConflictResolver abstraction
- Optimistic balance mutation with automatic rollback on server rejection

## 4. Code Examples

### Generic ConflictResolver base class for reconciling local Drift SQLite data with remote API models

```dart
abstract class ConflictResolver<TLocal, TRemote, TCompanion> {
  /// Resolves conflict between local and remote data.
  /// Returns the companion to persist locally.
  TCompanion resolve(TLocal local, TRemote remote);

  /// Creates a companion from remote data when no local data exists.
  TCompanion fromRemote(TRemote remote);
}
```

### Sync service pending queue processor with automatic rollback on error

```dart
Future<void> _syncPendingTransfers({int? specificId}) async {
  final db = Get.find<LocalStorage>().db;
  var query = db.select(db.pendingTransfers)
    ..where((t) => t.status.equals('pending'));
  if (specificId != null) {
    query.where((t) => t.id.equals(specificId));
  }
  final pending = await query.get();
  if (pending.isEmpty) return;
  final api = Get.find<TransferApiDatasource>();

  for (final item in pending) {
    try {
      final response = await api.submitTransfer(
        TransferRequest(
          fromAccountId: item.fromAccountId,
          toContactId: item.toContactId,
          amount: item.amount,
          note: item.note ?? '',
        ),
      );
      await (db.update(db.pendingTransfers)..where((t) => t.id.equals(item.id)))
          .write(const PendingTransfersCompanion(status: Value('synced')));
    } on ErrorData catch (e) {
      final accLocal = Get.find<AccountLocalDatasource>();
      final acc = await accLocal.getAccount(item.fromAccountId);
      if (acc != null) {
        await accLocal.updateBalance(acc.serverId, acc.balance + item.amount);
      }
      await (db.update(db.pendingTransfers)..where((t) => t.id.equals(item.id)))
          .write(PendingTransfersCompanion(status: const Value('failed'), errorMessage: Value<String?>(e.message)));
    }
  }
}
```

## 5. Considerations & Best Practices

- Pending queue must be FIFO to maintain correct sequence of financial transactions
- Balance compensation must rollback immediately when offline submission fails at remote backend
- Connectivity stream must be disposed on service close to prevent memory leaks

## 6. Related Knowledge

- Offline First Architecture
- Drift Sqlite Database

## 7. Source

- Harvest 1787110568549 448b547b.json
