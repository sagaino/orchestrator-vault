---
title: "Riverpod 2.0 Code-Gen & Async State Pattern"
type: pattern
tags: [pattern, mobile, flutter, riverpod, state-management, architecture]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787116476503-650c4b2e
sources: ["Harvest 1787116476503 650c4b2e.json"]
---

# Riverpod 2.0 Code-Gen & Async State Pattern

Manajemen state reaktif menggunakan Riverpod 2.0 code-gen Notifier dengan dukungan AsyncValue dan pagination prefetching.

## 1. Overview & Architecture

Pola manajemen state terstruktur berbasis Riverpod 2.0 dengan code-generation (build_runner). Notifier mengelola async lifecycle, pagination prefetching, dan data mutation secara reaktif tanpa tight coupling dengan UI Flutter.

## 2. Implementation & Code Structure

lib/
├── presenter/
│   └── mailbox/
│       ├── component/
│       ├── notifier/
│       │   ├── mailbox_notifier.dart
│       │   └── mailbox_notifier.g.dart
│       ├── mailbox.dart
│       └── page_mailbox.dart
└── data/
    └── repository/
        └── mailbox/
            ├── mailbox_repository.dart
            └── mailbox_repository_impl.dart

## 3. Key Implementation Points

- Deklarasi Notifier dengan anotasi @riverpod dan generator _$Notifier.
- Penggunaan AsyncValue untuk mengelola transisi state: loading, data, dan error secara deklaratif.
- Prefetching / infinite scroll pagination logic terisolasi di notifier layer.

## 4. Code Examples

### Penerapan Riverpod 2.0 Code-Gen Notifier dengan pagination dan AsyncValue state handling.

```dart
@riverpod
class MailboxNotifier extends _$MailboxNotifier {
  late final MailboxRepository _mailboxRepository;
  int offset = 1;
  int limit = 50;
  bool isMaxMailbox = false;
  bool isFetching = false;
  List<MailboxData> mailboxList = [];

  @override
  MailboxState build() {
    _mailboxRepository = ref.watch(mailboxRepositoryProvider);
    return MailboxState(
      data: const AsyncValue.loading(),
      totalData: 0,
    );
  }

  Future<void> getMailboxList({bool isRefresh = false}) async {
    try {
      isFetching = true;
      final result = await _mailboxRepository.mailboxList(limit: limit, offset: offset);
      if ((result.dataSet ?? []).length < limit) {
        isMaxMailbox = true;
      } else {
        offset++;
      }
      if (isRefresh) mailboxList.clear();
      mailboxList.addAll(result.dataSet ?? []);
      isFetching = false;
      state = MailboxState(
        data: AsyncValue.data(mailboxList),
        totalData: result.meta?.totalData ?? 0,
      );
    } on ErrorData catch (e) {
      isFetching = false;
      state = MailboxState(
        data: AsyncValue.error(e.message ?? '', StackTrace.current),
        totalData: 0,
      );
    }
  }
}
```

## 5. Considerations & Best Practices

- Gunakan ref.watch pada build() untuk dependency injection repository.
- Pisahkan state mutation asynchronous dari controller UI dan bungkus ke AsyncValue.data / AsyncValue.error.
- Manajemen state loading/pagination menggunakan flag boolean isFetching dan isMaxMailbox untuk mencegah race conditions saat prefetching.

## 6. Related Knowledge

- Riverpod State Management
- Async Value Pagination

## 7. Source

- Harvest 1787116476503 650c4b2e.json
