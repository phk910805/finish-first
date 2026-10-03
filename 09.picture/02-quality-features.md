# 02. 품질 피처 벡터

점수는 `score(p) = w · f(p)`. 이 문서는 `f(p)`의 명세와 각 피처를 iOS/Android에서 얻는 방법.

## 피처 명세

| # | 이름 | 의미 | iOS | Android | 사이클 |
|---|---|---|---|---|---|
| f1 | `global_sharpness` | 전체 선명도. 512px 회색조 라플라시안 분산 | Core Image `CIConvolution3X3` 또는 vImage | OpenCV `Laplacian().var()` 또는 직접 루프 | 1 |
| f2 | `exposure` | 평균 밝기의 0.5 거리, 클리핑 비율 | 히스토그램 직접 계산 | 동일 | 1 |
| f3 | `n_faces` | 검출된 얼굴 수 | `VNDetectFaceRectanglesRequest` | ML Kit `FaceDetector` | 1 |
| f4 | `eyes_open_min` | 주요 얼굴들의 눈 뜸 확률 최소값 | 랜드마크로 EAR 계산 또는 f8 사용 | `leftEyeOpenProbability`, `rightEyeOpenProbability` | 1 |
| f5 | `smile_mean` | 주요 얼굴 미소 평균 | 직접 모델 필요. 2사이클 이후 | `smilingProbability` | 1 (Android) |
| f6 | `face_size` | 최대 얼굴 면적 / 이미지 면적 | `VNFaceObservation.boundingBox` | `Face.boundingBox` | 1 |
| f7 | `face_centered` | 최대 얼굴 중심과 이미지 중심 거리 | 동일 | 동일 | 1 |
| f8 | `face_quality` | 얼굴 캡처 품질 0~1 | `VNDetectFaceCaptureQualityRequest` | 얼굴 크롭 라플라시안 + 눈 뜸 + 고개 각도 조합 | 1 |
| f9 | `facing_camera` | 고개가 카메라를 향하는 정도 | `VNFaceObservation.yaw/pitch/roll` | `headEulerAngleY/X/Z` | 1 |
| f10 | `face_sharpness` | 최대 얼굴 크롭의 라플라시안 분산 | 크롭 후 f1과 동일 | 동일 | 1 |
| f11 | `eyes_open_ratio` | 눈 뜬 얼굴 수 / 전체 얼굴 수 | 동일 | 동일 | 1 |
| f12 | `is_favorite` | 사용자가 이미 즐겨찾기/편집 | `PHAsset.isFavorite` | 신뢰할 소스 없음 | 1 (iOS) |
| f13 | `user_face_present` | 유저 얼굴 있음 | `03` 문서 | `03` 문서 | 2 |
| f14 | `user_face_quality` | 유저 얼굴의 f8 | 〃 | 〃 | 2 |
| f15 | `protagonist_face_quality` | 주인공 얼굴의 f8 | 〃 | 〃 | 2 |
| f16 | `embedding` | 사진 임베딩 (수백~천 차원) | Vision feature print | MediaPipe embedder | 4+ |

## 중요한 설계 원칙

**그룹 안에서 z-score 정규화한다.** 학습과 비교가 모두 같은 그룹 안에서 일어나므로 절대값은 의미가 없다. 각 피처를 그룹 평균 0, 표준편차 1로 바꾼 뒤 점수를 매긴다. 그룹이 2장이면 부호만 남는다. 이렇게 하면 "밝은 실내"와 "어두운 야외"의 노출 차이가 가중치를 오염시키지 않는다.

## 1사이클 초기 가중치 (손으로 정함)

```
w0 = {
  face_quality:      3.0,
  eyes_open_min:     2.5,
  face_sharpness:    2.0,
  facing_camera:     1.0,
  smile_mean:        1.0,
  global_sharpness:  1.0,
  eyes_open_ratio:   1.0,
  face_size:         0.5,
  face_centered:     0.3,
  exposure:         -0.5,   # 0.5에서 멀수록 감점
  is_favorite:       5.0,
}
얼굴이 하나도 없는 그룹: global_sharpness 3.0, exposure -1.0만 사용.
2사이클에서 user_face_quality 10.0, user_face_present 8.0 추가.
```

이 숫자는 출발점이다. 3사이클 학습이 덮어쓴다.

## 라플라시안 분산

```python
import cv2
gray = cv2.cvtColor(cv2.resize(img, (512, 384)), cv2.COLOR_BGR2GRAY)
sharpness = cv2.Laplacian(gray, cv2.CV_64F).var()
```

38개 버스트 그룹 중 37개에서 사람이 고른 가장 선명한 사진과 일치했다는 보고가 있다 (Medium, "Picking the Sharpest Photo from a Burst"). 같은 장면끼리만 비교하기 때문에 잘 맞는다. 장면이 다르면 쓸모없다.

모바일 구현: 3×3 커널 `[0 1 0; 1 -4 1; 0 1 0]` 컨볼루션 후 분산. Android는 `Bitmap`을 ByteArray로 받아 직접 루프를 돌려도 512px에서 수 ms.

## 플랫폼 API 메모

### iOS Vision
- `VNDetectFaceRectanglesRequest`: 얼굴 박스, yaw/roll (revision 3부터 pitch).
- `VNDetectFaceLandmarksRequest`: 눈·입 등 랜드마크. 눈 뜸은 랜드마크로 EAR(eye aspect ratio) 계산.
- `VNDetectFaceCaptureQualityRequest`: `VNFaceObservation.faceCaptureQuality` 0~1. 애플 문서가 명시하듯 "같은 사람의 여러 캡처를 비교"하는 용도. 조명·선명도·눈 감음·가림이 반영된다. 우리 문제에 정확히 맞는 API.
- `VNGenerateAttentionBasedSaliencyImageRequest`: 주목 영역. 주인공 판별 보조.
- 모든 요청은 `VNImageRequestHandler`에 묶어 한 번에 수행 가능.

### Android ML Kit Face Detection
- `FaceDetectorOptions`: `setClassificationMode(CLASSIFICATION_MODE_ALL)`로 미소·눈 뜸 확률, `setLandmarkMode`, `setPerformanceMode(PERFORMANCE_MODE_ACCURATE)`.
- `Face.smilingProbability`, `leftEyeOpenProbability`, `rightEyeOpenProbability`: 0~1. 눈 뜸 < 0.4면 감은 것으로 처리.
- `headEulerAngleY` (좌우), `headEulerAngleX` (상하), `headEulerAngleZ` (기울기). |Y| < 15도면 정면.
- 인식(누구인지)은 안 한다. 그건 `03` 문서.
- 16KB 페이지 정렬 이슈 등 네이티브 라이브러리 버그 리포트가 있으니 최신 버전 사용.

## 2사이클 이후 추가 가능한 모델

- **MagFace**: ArcFace 계열 임베딩의 벡터 크기가 곧 얼굴 품질. `03`에서 임베딩을 뽑으면 공짜로 얻는다.
- **NIMA**: 심미·기술 품질 점수. idealo 구현이 Keras. 모바일 변환 필요. 연속 촬영 비교에는 라플라시안보다 나을 게 별로 없다는 게 중론.
- **LAION aesthetic predictor**: CLIP 임베딩 위 선형 헤드. `04`의 임베딩 경로와 같은 구조.
- **Photo Triage 모델**: 시리즈 내 상대 품질을 직접 학습한 유일한 공개 모델. `08-papers.md`.

## 출처

- Pertuz, Puig, Garcia, "Analysis of focus measure operators for shape-from-focus", Pattern Recognition 2013
- Schlett 외, "Face Image Quality Assessment: A Literature Survey", ACM Computing Surveys 2022 — https://dl.acm.org/doi/10.1145/3507901
- Meng 외, "MagFace", CVPR 2021 — https://arxiv.org/abs/2103.06627
- Talebi, Milanfar, "NIMA", IEEE TIP 2018 — https://arxiv.org/abs/1709.05424
- Apple: Detecting face capture quality — https://developer.apple.com/documentation/vision/vndetectfacecapturequalityrequest
- ML Kit Face Detection Android — https://developers.google.com/ml-kit/vision/face-detection/android
- Medium, "Picking the Sharpest Photo from a Burst — No ML, Just Math" — https://medium.com/@nonikaleksandr/picking-the-sharpest-photo-from-a-burst-no-ml-just-math-6af787949ba5
- Miska-raligun/photo-picker 가중치 예시 (얼굴 초점 0.40, 눈 뜸 0.25 등) — https://github.com/Miska-raligun/photo-picker
