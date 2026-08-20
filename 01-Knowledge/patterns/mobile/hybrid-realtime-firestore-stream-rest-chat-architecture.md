---
title: "Hybrid Realtime Firestore Stream & REST Chat Architecture"
type: pattern
tags: [pattern, mobile, flutter, firestore, realtime-chat, firebase, streams]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787125853633-86cbed7a
sources: ["Harvest 1787125853633 86cbed7a.json"]
---

# Hybrid Realtime Firestore Stream & REST Chat Architecture

Pola arsitektur chat realtime hibrida menggunakan Cloud Firestore reactive streams dan REST API untuk push notification dan audit logging.

## 1. Overview & Architecture

Arsitektur chat realtime hibrida yang mengombinasikan Cloud Firestore Stream untuk syncing pesan instan tanpa WebSocket server terpisah, dengan REST API untuk audit dan notifikasi.

## 2. Implementation & Code Structure

lib/
├── data/
│   ├── api/service/message/message_service.dart
│   └── repository/message/message_repository_impl.dart
└── presenter/message/
    ├── notifier/message_notifier.dart
    ├── notifier/room_notifier.dart
    └── screen_message.dart

## 3. Key Implementation Points

- Listening realtime snapshot dokumen Firestore menggunakan StreamTransformer untuk injeksi date separator otomatis
- Operasi kirim pesan dwifungsi: menulis ke Firestore subcollection dan memanggil REST endpoint untuk audit & push notification
- Dukungan gamification polymorphic message types (text, voice, match-off game, tarot cards)

## 4. Code Examples

### Hybrid Firestore reactive stream with in-memory date separators and REST notification trigger on send

```dart
@riverpod
class MessageNotifier extends _$MessageNotifier {
  late final MessageRepository _messageRepository;

  @override
  Stream<List<MessageData>> build(String roomId) {
    _messageRepository = ref.watch(messageRepositoryProvider);
    final stream = _messageRepository.fetchMessages(roomId);
    return stream.map<List<MessageData>>((event) {
      final List<MessageData> allMessage = List.empty(growable: true);
      int currentTimeEpoch = 0;
      for (final element in event.docs) {
        final message = MessageData.fromJson(element.data());
        if (currentTimeEpoch == 0) currentTimeEpoch = message.timeStamp!;
        final isSameDay = DateUtil.isToday(
            DateTime.fromMillisecondsSinceEpoch(currentTimeEpoch),
            DateTime.fromMillisecondsSinceEpoch(message.timeStamp!));
        if (!isSameDay) {
          allMessage.add(MessageData(
              messageId: '',
              senderId: '',
              type: MessageType.dateSeparator,
              content: DateUtil.getStringDateFromEpoch(currentTimeEpoch,
                  outputFormat: DateUtil.clientDateFormat3)));
          currentTimeEpoch = message.timeStamp!;
        }
        allMessage.add(message);
      }
      return allMessage;
    });
  }

  void sendMessage(String content) {
    CacheUtil.getString(Strings.keyUserId).then((userId) {
      _messageRepository.sendMessage(
          MessageData(
            content: content,
            messageId: const Uuid().v4(),
            type: MessageType.text,
            timeStamp: DateUtil.timeStamp(),
            senderId: userId,
          ),
          roomId,
          ref.read(roomNotifierProvider.call(roomId).notifier).targetUserId);
    });
  }
}
```

## 5. Considerations & Best Practices

- Query Firestore subcollection harus di-index berdasarkan time_stamp descending untuk performa tinggi
- Batalkan stream listener saat screen didispose untuk menghemat baterai dan konsumsi kuota Firestore

## 6. Related Knowledge

- [[01-Knowledge/patterns/mobile/offline-first-synchronization-queue-conflict-resolver.md]]

## 7. Source

- Harvest 1787125853633 86cbed7a.json
