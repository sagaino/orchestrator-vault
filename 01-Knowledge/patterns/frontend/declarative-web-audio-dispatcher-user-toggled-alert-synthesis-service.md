---
title: "declarative-web-audio-dispatcher-user-toggled-alert-synthesis-service"
type: pattern
tags: [pattern, frontend, audio, media, notification, services, user-settings]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787132260416-42b894f9
sources: ["Harvest 1787132260416 42b894f9.json"]
---

# declarative-web-audio-dispatcher-user-toggled-alert-synthesis-service

Pola audio alert synthesizer mandiri dengan volume normalization dan user-toggled gesture compliance untuk notifikasi insiden real-time.

## 1. Overview & Architecture

Pola arsitektur audio notification dispatcher terisolasi yang mengelola siklus hidup HTML5 Audio buffer, pembatasan volume, dan penanganan kebijakan autoplay browser. Pola ini terintegrasi secara reaktif dengan sistem notifikasi real-time WebSocket dan dikontrol oleh preferensi audio pengguna.

## 2. Implementation & Code Structure

src/lib/playAlarmSound.ts (Audio synthesis utility & HTML5 Audio buffer lifecycle)
├── src/components/common/Header.tsx (Audio dispatch orchestration & toggle state)
├── src/pages/Alert/NotificationDropdown.tsx (Sound mute/unmute control UI)
└── src/pages/Alert/hooks/useStompNotifications.ts (Real-time trigger source)

## 3. Key Implementation Points

- Isolasi modul audio synthesizer mandiri dengan volume clamp dan error catch guard.
- Koneksi decoupled antara push notification WebSocket listener dan audio trigger melalui callback hook (`onNewNotification`).
- Kontrol preferensi suara pengguna via state toggle di header/dropdown notification.
- Safe handling kegagalan browser autoplay tanpa merusak flow notifikasi data.

## 4. Code Examples

### Audio synthesis service with volume normalization and promise catch guard.

```typescript
// src/lib/playAlarmSound.ts
export const playAlarmSound = () => {
  try {
    const audio = new Audio("/assets/alarm.mp3");
    audio.volume = 0.5;
    audio.play().catch((error) => {
      console.error("Failed to play alarm sound:", error);
    });
  } catch (error) {
    console.error("Error creating audio object:", error);
  }
};
```

### Header component orchestrating audio playback based on user preference toggle upon real-time alert dispatch.

```typescript
// src/components/common/Header.tsx
const Header: React.FC = () => {
  const [isSoundOn, setIsSoundOn] = useState(false);

  const handleNewNotification = useCallback(() => {
    if (isSoundOn) {
      playAlarmSound();
    }
  }, [isSoundOn]);

  const { notifications, handleMarkAllRead } = useStompNotifications({
    onNewNotification: handleNewNotification,
  });

  return (
    <header ...>
      <NotificationDropdown
        notifications={notifications}
        onMarkAllRead={handleMarkAllRead}
        onSoundToggle={setIsSoundOn}
        isSoundOn={isSoundOn}
      />
    </header>
  );
};
```

## 5. Considerations & Best Practices

- Browser Autoplay Policy memblokir pemutaran audio otomatis tanpa interaksi pengguna awal (user gesture). Penggunaan user toggle (`isSoundOn`) memenuhi syarat inisiasi gesture.
- Audio instance dibuat on-demand untuk mencegah memory leak audio context yang tertahan lama di browser.
- Volume audio dinormalisasi (0.5) untuk mencegah audio clipping pada speaker hardware client.

## 6. Related Knowledge

- `Web Audio API / HTML5 Audio Element`
- `Browser Autoplay Policy Lifecycle`
- `Real-time Audio Cue Dispatching`

## 7. Source

- Harvest 1787132260416 42b894f9.json
