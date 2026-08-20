---
title: "Riverpod Code-Gen Notifier Architecture"
type: pattern
tags: [pattern, mobile, flutter, riverpod, state-management, code-generation]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787116045805-ce1291de
sources: ["Harvest 1787116045805 Ce1291de.json"]
---

# Riverpod Code-Gen Notifier Architecture

Pola implementasi state management Riverpod 2 berbasis code generation (@riverpod) dengan pemisahan immutable state dan dependency injection otomatis.

## 1. Overview & Architecture

Pola arsitektur state management menggunakan Riverpod 2 dengan anotasi generator (@riverpod). Menyatukan Business Logic Controller, Dependency Injection, dan Reactive State Binding ke widget Flutter secara type-safe dan testable.

## 2. Implementation & Code Structure

lib/
├── presenter/
│   └── <feature>/
│       ├── component/
│       ├── notifier/
│       │   ├── <feature>_notifier.dart
│       │   └── <feature>_notifier.g.dart
│       └── page_<feature>.dart
└── data/
    └── repository/
        └── <feature>/

## 3. Key Implementation Points

- Anotasi @riverpod memicu generasi boilerplate code secara otomatis.
- Method build() menerima parameter dinamis (family pattern) dan menginisialisasi repository.
- Mutasi state selalu menghasilkan instans state baru yang memicu update reaktif pada UI.

## 4. Code Examples

### Riverpod notifier class with code generation, parameter injection in build(), and immutable state emission.

```dart
import 'package:lovelet_lookin/data/repository/match/match_repository.dart';
import 'package:lovelet_lookin/data/repository/match/match_repository_impl.dart';
import 'package:lovelet_lookin/data/repository/mailbox/mailbox_repository.dart';
import 'package:lovelet_lookin/data/repository/mailbox/mailbox_repository_impl.dart';
import 'package:lovelet_lookin/model/error_data.dart';
import 'package:riverpod_annotation/riverpod_annotation.dart';

part 'card_notifier.g.dart';

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
      isMailboxSent: isMailboxSent,
    );
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
        isMailboxSent: isMailboxSent,
      );
    } on ErrorData catch (e) {
      isLike = null;
      isMatch = false;
      state = CardState(
        isMatch: isMatch,
        isLike: isLike,
        isLoadingMailbox: isLoadingMailbox,
        isMailboxSent: isMailboxSent,
      );
    }
  }
}
```

## 5. Considerations & Best Practices

- Gunakan immutable state class untuk memicu re-render widget secara presisi.
- Gunakan ref.watch() di dalam build() untuk dependency injection dari provider lain.
- Gunakan code-generator build_runner secara konsisten saat menambahkan state provider baru.

## 6. Related Knowledge

- State Management
- Clean Architecture Flutter

## 7. Source

- Harvest 1787116045805 Ce1291de.json
