---
title: "Riverpod 2.x CodeGen Layered State Architecture"
type: pattern
tags: [pattern, mobile, flutter, riverpod, state-management, clean-architecture]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787125853633-86cbed7a
sources: ["Harvest 1787125853633 86cbed7a.json"]
---

# Riverpod 2.x CodeGen Layered State Architecture

Pola implementasi state management Riverpod 2.x berbasis code generation (@riverpod) dengan pemisahan immutable state dan dependency injection otomatis antar layer.

## 1. Overview & Architecture

Arsitektur state management layered berbasis Riverpod 2.x dengan code-generation (@riverpod) yang memisahkan UI Presentation, Business Logic Notifiers, dan Data Repositories secara type-safe.

## 2. Implementation & Code Structure

lib/
├── data/
│   ├── api/service/<feature>/<feature>_service.dart
│   └── repository/<feature>/<feature>_repository_impl.dart
└── presenter/<feature>/
    ├── notifier/<feature>_notifier.dart
    ├── component/
    └── screen_<feature>.dart

## 3. Key Implementation Points

- Dependency Injection satu arah: BaseHttpClient -> Service -> Repository -> Notifier -> UI Widget
- Anotasi @riverpod dengan dynamic family argument via method build(String id)
- Penggunaan state immutable untuk memicu re-render widget secara presisi

## 4. Code Examples

### Riverpod Notifier with code generation, repository injection in build(), and immutable state transitions

```dart
@riverpod
class CardNotifier extends _$CardNotifier {
  late final MatchRepository _matchRepository;
  late final MailboxRepository _mailboxRepository;
  bool isMatch = false;
  bool? isLike;
  bool isLoadingMailbox = false;
  bool isMailboxSent = false;

  @override
  CardState build(String targetUserId) {
    _matchRepository = ref.watch(matchRepositoryProvider);
    _mailboxRepository = ref.watch(mailboxRepositoryProvider);
    return CardState(
        isMatch: isMatch,
        isLoadingMailbox: isLoadingMailbox,
        isMailboxSent: isMailboxSent);
  }

  void dislike(String content) async {
    try {
      final result = await _matchRepository.dislike(targetUserId: targetUserId);
      isMatch = result?.isMatched ?? false;
      isLike = false;
      state = CardState(
          isMatch: isMatch,
          isLike: isLike,
          isLoadingMailbox: isLoadingMailbox,
          isMailboxSent: isMailboxSent);
    } on ErrorData catch (e) {
      isLike = null;
      isMatch = false;
      state = CardState(
          isMatch: isMatch,
          isLike: isLike,
          isLoadingMailbox: isLoadingMailbox,
          isMailboxSent: isMailboxSent);
    }
  }
}
```

## 5. Considerations & Best Practices

- Selalu jalankan build_runner saat menambahkan atau memperbarui provider anotasi @riverpod
- Gunakan auto-dispose untuk presenter screen agar memori terbebas saat navigasi pop

## 6. Related Knowledge

- [[01-Knowledge/patterns/mobile/feature-first-clean-modular-architecture-with-reactive-bindings.md]]
- State Management

## 7. Source

- Harvest 1787125853633 86cbed7a.json
