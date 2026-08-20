---
title: "Biometric Face Liveness & EXIF Similarity Engine"
type: pattern
tags: [pattern, mobile, flutter, biometrics, face-recognition, liveness, native-sdk]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787125853633-86cbed7a
sources: ["Harvest 1787125853633 86cbed7a.json"]
---

# Biometric Face Liveness & EXIF Similarity Engine

Pola integrasi native face biometrics dan kalkulasi skor kemiripan wajah dengan normalisasi orientasi EXIF kamera.

## 1. Overview & Architecture

Pola integrasi SDK biometrik native untuk deteksi liveness wajah, ekstraksi landmark template, dan penghitungan skor kesamaan (similarity) antara foto selfie dan foto profil.

## 2. Implementation & Code Structure

facesdk_plugin/
lib/
├── presenter/face_recognition/
│   ├── notifier/face_recognition_notifier.dart
│   └── screen_face_recognition.dart
└── model/face_model.dart

## 3. Key Implementation Points

- Inisialisasi dan aktivasi native face SDK per-platform (Android/iOS license keys)
- Rotasi otomatis EXIF gambar sebelum ekstraksi feature landmarks wajah
- Penyimpanan vector embedding selfie ke secure keychain dan komparasi cosine similarity dengan foto profil galeri

## 4. Code Examples

### Facial similarity calculation engine matching candidate face templates against verified selfie template

```dart
Future<FilesMeta> calculateSimilarity({
  required List<FaceModel> faceModel,
  PickPhotoModel? pickPhotoModel,
  PickPhotoProfileModel? pickPhotoProfileModel,
  int? key,
}) async {
  List<FilesMeta> listFilesMeta = List.empty(growable: true);
  final Uint8List? templateSelfie =
      await CacheUtil.getUint8List(Strings.keyTemplateSelfie);

  if (templateSelfie == null) {
    return FilesMeta(
      filename: pickPhotoModel?.photo.name ?? pickPhotoProfileModel?.photo?.name,
      order: key,
      similarity: 0.1,
    );
  }

  for (var el in faceModel) {
    double similarity = (await _facesdkPlugin.similarityCalculation(
      el.templates!,
      templateSelfie,
    )) ?? 0.1;
    listFilesMeta.add(FilesMeta(
      filename: pickPhotoModel?.photo.name ?? pickPhotoProfileModel?.photo?.name,
      order: key,
      similarity: similarity,
      meta: MetaKBY.fromJson(el.toJson()),
    ));
  }

  final highestValue = listFilesMeta.reduce((value, element) =>
      (value.similarity ?? 0.1) > (element.similarity ?? 0.1) ? value : element);
  return highestValue;
}
```

## 5. Considerations & Best Practices

- Orientasi gambar harus dinormalisasi dengan FlutterExifRotation sebelum diekstraksi template wajahnya untuk mencegah rotasi gambar miring pada sensor kamera Android/iOS tertentu

## 6. Related Knowledge

- [[01-Knowledge/patterns/mobile/biometric-first-local-auth-fallback-pin-session-gate.md]]

## 7. Source

- Harvest 1787125853633 86cbed7a.json
