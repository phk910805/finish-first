# 09. 코드 저장소와 샘플

## 그룹핑, 유사도

- JohannesBuchner/imagehash — dHash·pHash·aHash·wHash Python 구현. 로직 참고용 — https://github.com/JohannesBuchner/imagehash
- google-ai-edge/mediapipe-samples, image_embedder Android — https://github.com/google-ai-edge/mediapipe-samples/tree/main/examples/image_embedder/android
- google-ai-edge/mediapipe-samples, image_embedder iOS — https://github.com/google-ai-edge/mediapipe-samples/blob/main/examples/image_embedder/ios/ImageEmbedder/Services/ImageEmbedderService.swift
- RazgrizHsu/immich-deduper — CLIP 임베딩 기반 유사 사진 탐지, 임계값 조절 UI — https://github.com/RazgrizHsu/immich-deduper
- immich-stack — 버스트 시퀀스 스택 CLI — https://majorfi.github.io/immich-stack/
- titanxxh/immich PR #45 — 버스트 중 가장 선명한 사진 선택 구현 — https://github.com/titanxxh/immich/pull/45

## 베스트 컷 선별 (가장 가까운 참고)

- Miska-raligun/photo-picker — 선명도, 눈 뜸(OCEC 분류기), 노출, 심미(LAION 헤드) 가중 합산. 가중치 예시와 XMP 별점 출력 — https://github.com/Miska-raligun/photo-picker
- Princeton Photo Triage — 데이터셋·모델·벤치마크 — https://gfx.cs.princeton.edu/pubs/Chang_2016_ATF/index.php
- dennisbrian/retouch PR #9 — 눈 감음 플래그, 얼굴 선명도 우선 — https://github.com/dennisbrian/retouch/pull/9

## 얼굴 검출·인식 (온디바이스)

- googlesamples/mlkit, vision-quickstart — 얼굴 검출 옵션 전부 시연 — https://github.com/googlesamples/mlkit/blob/master/android/vision-quickstart/README.md
- deepinsight/insightface — ArcFace 공식, 사전학습 모델 — https://github.com/deepinsight/insightface
- JuheonYi/TFLiteFaceExample — Android TFLite, RetinaFace + MobileFaceNet — https://github.com/JuheonYi/TFLiteFaceExample
- shubham0204/OnDevice-Face-Recognition-Android — Android 온디바이스 얼굴 인식 — https://github.com/shubham0204/OnDevice-Face-Recognition-Android
- syaringan357/Android-MobileFaceNet-MTCNN-FaceAntiSpoofing — https://github.com/syaringan357/Android-MobileFaceNet-MTCNN-FaceAntiSpoofing
- syaringan357/iOS-MobileFaceNet-MTCNN-FaceAntiSpoofing — iOS TFLite — https://github.com/syaringan357/iOS-MobileFaceNet-MTCNN-FaceAntiSpoofing
- dlib 얼굴 클러스터링 예제 (Chinese Whispers) — http://dlib.net/face_clustering.py.html

## 품질·심미

- idealo/image-quality-assessment — NIMA Keras 구현 — https://github.com/idealo/image-quality-assessment
- christophschuhmann/improved-aesthetic-predictor — CLIP + MLP 심미 예측기 — https://github.com/christophschuhmann/improved-aesthetic-predictor
- shunk031/simple-aesthetics-predictor — 위 모델의 transformers 스타일 래퍼 — https://github.com/shunk031/simple-aesthetics-predictor

## 플랫폼 문서

- Apple Vision — https://developer.apple.com/documentation/vision
- Apple PhotoKit — https://developer.apple.com/documentation/photokit
- ML Kit Face Detection Android — https://developers.google.com/ml-kit/vision/face-detection/android
- MediaPipe Image Embedder — https://developers.google.com/edge/mediapipe/solutions/vision/image_embedder
- Android MediaStore — https://developer.android.com/training/data-storage/shared/media

## 사용 지침

- 1사이클에 쓰는 것: 없음. dHash는 `01` 문서의 20줄로 직접 쓴다. ML Kit는 문서만.
- 2사이클: insightface에서 MobileFaceNet 가중치, JuheonYi 또는 shubham0204로 Android 연동 패턴.
- 3사이클: photo-picker의 가중치와 Photo Triage 데이터셋.
- 라이선스는 가져다 쓸 때 확인. insightface 모델 일부는 비상업 제한이 있다.
