# 01. 연속 촬영 사진 그룹핑

목표: "같은 장면을 연달아 찍은 사진"을 하나의 그룹으로 묶는다. 완전 중복이 아니라 유사샷이 대상.

## 2단계 구조

### A. 시간 창으로 후보 묶기

- 촬영 시각 기준으로 정렬하고, 인접 사진 간격이 N초 이내면 같은 후보군.
- N 기본값 10초. 3~30초 사이에서 조정. 결혼식·여행은 길게, 일상은 짧게.
- 같은 카메라(전면/후면)와 같은 방향(세로/가로)이어야 같은 후보군.
- 시각 소스: iOS `PHAsset.creationDate`, Android `MediaStore.Images.Media.DATE_TAKEN` (없으면 `DATE_ADDED`).
- iOS는 `PHAsset.burstIdentifier`가 있으면 그 자체가 그룹. 셔터를 길게 눌러 찍은 진짜 버스트만 해당.

### B. 시각 유사도로 확정

후보군 안에서 모든 쌍의 유사도를 계산하고, 임계값 이상이면 같은 그룹 (union-find).

**옵션 1. dHash (1사이클 권장)**
- 9×8 회색조로 축소, 가로 인접 픽셀 밝기 비교 → 64비트.
- 해밍 거리 ≤ 10이면 유사. 5 이하는 거의 동일, 15 이상은 다른 장면. 손으로 라벨링해서 조정.
- 장점: 의존성 0, 밀리초. 단점: 구도가 조금만 바뀌어도 거리 커짐. 연속 촬영에는 충분.
- pHash(DCT 기반)는 회전·크롭에 조금 더 강하지만 1사이클엔 불필요.

**옵션 2. 플랫폼 임베딩 (2사이클 이후)**
- iOS: `VNGenerateImageFeaturePrintRequest` → `VNFeaturePrintObservation.computeDistance(_:to:)`.
  실무 임계값: 0.35 (ShutterSlim, 수백 쌍 라벨링으로 결정), 0.4~0.6 구간 (다른 개발자 보고). 낮을수록 유사.
- Android: MediaPipe `ImageEmbedder` (MobileNetV3) + `ImageEmbedder.cosineSimilarity`. L2 정규화 켜기.
- CLIP 계열 참고치 (Immich): 코사인 ≥ 0.97 완전 중복, 0.60~0.70 연속 촬영 유사샷.

### C. 그룹 후처리

- 그룹 크기 상한 (예: 30장). 넘으면 시간으로 쪼갬.
- 1장짜리 그룹은 표시 안 함.
- 그룹 내 정렬은 촬영 시각.

## 구현 메모

- 해시·임베딩은 반드시 축소본(긴 변 256~512px)으로 계산. HEIC 원본 디코드는 느리다.
- 결과를 asset id 키로 캐시. 재스캔 때 새 사진만 계산.
- iCloud 최적화로 기기에 없는 사진은 1사이클에서 건너뜀 (`PHImageRequestOptions.isNetworkAccessAllowed = false`). Clever Cleaner 불만 사례의 원인.
- Live Photo는 정지 이미지만 사용.
- 백그라운드: iOS `BGProcessingTask`, Android `WorkManager`.

## dHash 참고 구현 (Python, 검증용)

```python
from PIL import Image

def dhash(path, size=8):
    img = Image.open(path).convert("L").resize((size + 1, size))
    px = list(img.getdata())
    bits = 0
    for row in range(size):
        for col in range(size):
            left = px[row * (size + 1) + col]
            right = px[row * (size + 1) + col + 1]
            bits = (bits << 1) | (1 if left > right else 0)
    return bits

def hamming(a, b):
    return bin(a ^ b).count("1")
```

Kotlin/Swift로 옮길 때도 같은 로직. 라이브러리 불필요.

## 임계값 잡는 법

1. 내 사진첩에서 시간 창 후보군 200쌍을 뽑는다.
2. "같은 장면인가"를 손으로 O/X 라벨링한다.
3. 거리 분포를 그려 O와 X가 갈리는 지점을 임계값으로 잡는다.
4. 2사이클 이후 다른 사람 사진첩으로 재검증.

## 출처

- dHash 원문: Hacker Factor, "Kind of Like That" (2013)
- pHash 벤치마크: Zauner, "Implementation and Benchmarking of Perceptual Image Hash Functions" (2010, 석사논문)
- ShutterSlim 블로그: Vision 프레임워크 유사도 0.35 — https://shutterslim.com/blog/2026/01/18/vision-framework-image-similarity/
- Immich 중복 처리 가이드 — https://www.bigiron.cc/guides/immich-duplicate-handling-at-100k-photos-what-actually-works
- MediaPipe Image Embedder Android — https://developers.google.com/edge/mediapipe/solutions/vision/image_embedder/android
- Apple Vision `VNGenerateImageFeaturePrintRequest` — https://developer.apple.com/documentation/vision/vngenerateimagefeatureprintrequest
