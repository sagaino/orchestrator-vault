---
title: "Preemptive Buffer Infinite Card Pagination Machine"
type: pattern
tags: [pattern, mobile, flutter, pagination, infinite-scroll, swipeable-cards, performance]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787125853633-86cbed7a
sources: ["Harvest 1787125853633 86cbed7a.json"]
---

# Preemptive Buffer Infinite Card Pagination Machine

Pola pagination infinite card dengan sistem lookahead buffer prefetching dan optimistik penghapusan item kartu pada aksi swipe.

## 1. Overview & Architecture

Pola pagination swipeable cards dengan prefetching otomatis ketika jumlah buffer kartu mendekati ambang batas tertentu (< 3 kartu), mencegah layar kosong saat user melakukan swipe cepat.

## 2. Implementation & Code Structure

lib/
├── data/repository/feed/feed_repository_impl.dart
└── presenter/home/
    ├── notifier/for_you_page_notifier.dart
    ├── notifier/card_notifier.dart
    └── component/page_for_you.dart

## 3. Key Implementation Points

- Preemptive background fetching ketika selisih buffer kartu tersisa < 3 item
- Optimistic UI update: kartu langsung dihapus dari deck saat swipe dilakukan tanpa menunggu respon backend
- Deteksi batas data (isFullLoaded) untuk menghentikan pemanggilan pagination yang tidak diperlukan

## 4. Code Examples

### Preemptive buffer pagination with buffer < 3 prefetch trigger and optimistic item removal

```dart
@riverpod
class ForYouPageNotifier extends _$ForYouPageNotifier {
  late final FeedRepository _feedRepository;
  final List<FeedUserData> _listUserData = List.empty(growable: true);
  bool _isFullLoaded = false;
  int _limit = 10;
  int _offSet = 1;

  void loadMore(int newPage) {
    if (_isFullLoaded) return;
    if ((_listUserData.length - newPage) < 3) {
      _offSet++;
      getForYouPageData();
    }
  }

  void removeUserFeed(String targetUserId) {
    int tempLength = _listUserData.length;
    _listUserData.removeWhere((data) => data.id == targetUserId);
    if (tempLength != _listUserData.length) {
      state = AsyncValue.data(ForYouPageState(
          listFeedUser: _listUserData,
          isFullLoaded: _isFullLoaded,
          isMaxSeen: false,
          page: _offSet));
    }
  }
}
```

## 5. Considerations & Best Practices

- Pastikan flag _isFullLoaded aktif saat API mengembalikan data lebih sedikit dari limit untuk mencegah request berulang saat data habis
- Reset offset dan clear list secara bersih pada aksi refresh pull-to-refresh

## 6. Related Knowledge

- [[01-Knowledge/patterns/mobile/riverpod-2-0-code-gen-async-state-pattern.md]]

## 7. Source

- Harvest 1787125853633 86cbed7a.json
