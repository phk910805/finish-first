# 03. 유저 얼굴 식별과 주인공 판별 (2사이클)

규칙: 베스트 컷의 1순위는 유저의 얼굴. 유저가 없으면 사진의 주인공을 정하고 그 얼굴이 잘 나온 장을 고른다.

## A. 유저 얼굴 등록과 매칭

### 등록
- 앱 첫 실행 시 셀카 3장 (정면, 살짝 좌, 살짝 우). 임베딩 3개의 평균을 유저 템플릿으로 저장. 기기 밖으로 안 나감.
- 대안(자동): 전면 카메라로 찍은 사진(EXIF `LensModel`에 "front")에서 가장 자주 나오는 얼굴 클러스터를 유저로 추정하고 확인만 받는다.

### 임베딩 모델
- **MobileFaceNet** (ArcFace 손실로 학습). 입력 112×112 정렬 얼굴, 출력 128~512차원, 모델 약 4MB, 모바일에서 수십 ms.
- 정렬에 5점 랜드마크(양 눈, 코, 입 양끝)가 필요. ML Kit 랜드마크 / Vision 랜드마크로 얻어 어파인 변환.
- 매칭: 코사인 유사도. 임계값은 모델마다 달라 0.5~0.65 구간에서 직접 측정해 잡는다. 내 얼굴 50장, 남 얼굴 200장으로 ROC 그리면 10분.
- 애플 Vision과 ML Kit 모두 얼굴 "인식" 임베딩을 제공하지 않는다. 직접 모델을 넣어야 한다. TFLite(LiteRT)는 양쪽 다 돌고, iOS는 Core ML 변환도 가능.
- **MagFace**를 쓰면 같은 임베딩의 노름이 얼굴 품질이 된다. 인식과 품질을 한 모델로.

### 프라이버시
- 얼굴 임베딩과 템플릿은 로컬 저장소. 백업 제외 권장.
- 서버로 올리는 것은 `05` 문서의 피처 벡터뿐이며 얼굴 임베딩은 제외.

## B. 자주 나오는 얼굴 = 중요한 사람

라이브러리 전체 얼굴 임베딩을 클러스터링하면 "가족"이 자연스럽게 큰 클러스터가 된다.

- 알고리즘: Chinese Whispers (dlib가 쓰는 방식) 또는 DBSCAN (eps는 코사인 거리 기준 매칭 임계값과 동일).
- 클러스터 크기 순위를 `importance_prior`로 사용. 상위 3개 클러스터가 보통 본인·배우자·아이.
- 수천 장 기준 임베딩 계산이 수 분 걸리므로 백그라운드에서 점진적으로.

## C. 주인공 점수 (유저가 없을 때)

```
protagonist(face) = 2.0 * importance_prior_rank_score
                  + 1.5 * face_size_norm
                  + 1.0 * centeredness
                  + 1.0 * relative_sharpness
                  + 0.5 * saliency_overlap
```

- VIP 논문(CVPR 2015)이 신원 없이 쓰는 단서: 크기, 위치, 선명도, 얼굴 방향, 타인과의 상대 관계. 우리는 여기에 빈도 사전확률을 더한다. 빈도가 가장 강한 신호다.
- 주인공이 정해지면 그 얼굴의 `face_quality`가 `f15`가 되고, 가중치는 유저 얼굴 다음으로 크게.
- 단체 사진에서 유저 얼굴이 모두 비슷하면 나머지 인원의 `eyes_open_ratio`가 타이브레이커.

## D. 실패 모드와 대응

| 실패 | 대응 |
|---|---|
| 마스크·선글라스로 매칭 실패 | 그룹 안에서 한 장이라도 매칭되면 같은 위치의 얼굴을 추적 (박스 IoU) |
| 아이가 자라 임베딩이 변함 | 클러스터를 주기적으로 재계산, 템플릿은 최근 가중 평균 |
| 쌍둥이 | 포기. 둘 다 주인공으로 취급 |
| 뒷모습·측면 | 얼굴 없음으로 처리해 전체 선명도만 사용 |

## 출처

- Schroff 외, "FaceNet", CVPR 2015 — https://arxiv.org/abs/1503.03832
- Deng 외, "ArcFace", CVPR 2019 — https://arxiv.org/abs/1801.07698
- Chen 외, "MobileFaceNets", 2018 — https://arxiv.org/abs/1804.07573
- Meng 외, "MagFace", CVPR 2021 — https://arxiv.org/abs/2103.06627
- Bazarevsky 외, "BlazeFace", 2019 — https://arxiv.org/abs/1907.05047
- Mathialagan, Gallagher, Batra, "VIP: Finding Important People in Images", CVPR 2015 — https://arxiv.org/abs/1502.05678
- Oh 외, "Person Recognition in Personal Photo Collections", ICCV 2015 / TPAMI — https://arxiv.org/abs/1710.03224
- Biemann, "Chinese Whispers", 2006. dlib 예제 — http://dlib.net/face_clustering.py.html
- insightface (ArcFace 공식) — https://github.com/deepinsight/insightface
