# 04. 취향 학습 (3사이클)

운전 데이터 학습과 같은 구조. 시스템이 먼저 고르고, 사용자가 다르게 고르면 그 차이가 학습 데이터.

## 로그 스키마 (1사이클부터 수집)

```
group_log
  group_id        TEXT
  created_at      INTEGER   # epoch ms
  shown_with_rec  INTEGER   # 1이면 추천 배지를 보여줌, 0이면 숨김(편향 없는 평가용)
  rec_index       INTEGER   # 시스템 추천 인덱스
  chosen_index    INTEGER   # 사용자가 고른 인덱스, 미선택이면 NULL
  n_photos        INTEGER
  features_json   TEXT      # [[f1..fk], [f1..fk], ...] 그룹 내 z-score 정규화 전 원값
  weights_version TEXT      # 추천에 쓴 w 버전
```

원값을 저장하고 정규화는 학습 시점에 다시 한다. 피처를 추가해도 과거 로그를 못 쓰게 되지 않도록 키 이름을 같이 저장.

## 모델: 쌍 비교 로지스틱 (Bradley-Terry / RankNet)

사용자가 그룹에서 c를 골랐으면, 안 고른 각 r에 대해 "c > r" 쌍이 생긴다. 5장 그룹이면 4쌍.

```
P(c > r) = sigmoid( w · (f(c) - f(r)) )
loss     = -log P(c > r) + (λ/2) * ||w - w0||²
```

온라인 업데이트 (SGD):

```python
import numpy as np

def sigmoid(x): return 1.0 / (1.0 + np.exp(-x))

def update(w, w0, fc, fr, lr=0.05, lam=0.01):
    d = fc - fr                      # 그룹 내 z-score 정규화된 피처 차
    p = sigmoid(w @ d)
    grad = (1.0 - p) * d - lam * (w - w0)
    return w + lr * grad

def train_group(w, w0, feats, chosen, weight=1.0):
    fc = feats[chosen]
    for i, fr in enumerate(feats):
        if i == chosen: continue
        for _ in range(int(weight)):
            w = update(w, w0, fc, fr)
    return w
```

- `w0`: `02` 문서의 손 가중치. λ 항이 w를 w0 쪽으로 당겨서 샘플이 적을 때 폭주를 막는다.
- 샘플 20~50개면 개인차가 드러난다. 그 전엔 거의 w0.
- 학습은 기기 안에서 밀리초. 서버 불필요.

## 신호의 가중치

| 사용자 행동 | 신호 | 학습 가중치 |
|---|---|---|
| 추천과 다른 사진을 고름 | 개입. 가장 비싼 데이터 | 3 |
| 추천 배지 없이 고름 (`shown_with_rec=0`) | 편향 없는 선택 | 2 |
| 추천 그대로 수락 | 약한 신호. 앵커링 가능성 | 1 또는 샘플링 |
| 그룹을 열고 아무것도 안 고름 | 무시 | 0 |
| 나중에 즐겨찾기·공유·편집 | 암묵적 긍정 | 1 (수집 가능할 때) |

## 앵커링 편향

추천 배지를 보여주면 사용자가 그걸 따라간다. 검색 클릭 데이터의 위치 편향과 같은 문제.
- 대응 1: 5그룹 중 1그룹은 배지 없이 보여주고 `shown_with_rec=0`으로 기록. 이 로그만으로 평가하면 편향이 없다.
- 대응 2: 그룹 내 사진 순서를 촬영 시각이 아니라 무작위로 섞는다 (순서 편향 제거).
- 이론: Joachims 외 SIGIR 2005 (클릭 해석), Joachims·Swaminathan·Schnabel WSDM 2017 (편향 보정 학습).

## 평가

- 지표: **top-1 일치율** = 시스템 추천 == 사용자 선택 비율. `shown_with_rec=0` 로그로만 계산.
- 학습 곡선: 로그 수에 따른 일치율. 올라가야 한다.
- 섀도 모드: 새 가중치 w'로 추천을 계산하되 보여주지 않고, 실제 선택과의 일치율을 현재 w와 비교. w'가 20개 연속 그룹에서 더 높으면 교체.
- 오프라인 검증: Photo Triage 데이터셋(5,953 시리즈, 사람 선호 라벨)에 같은 피처를 뽑아 돌려보면 w0의 상한을 미리 안다.

## 피처로 안 잡히는 취향

"자연스러운 표정 대 포즈", "구도" 같은 취향은 위 피처에 없다. 그때:
1. 사진 임베딩(Vision feature print, MediaPipe embedder)을 피처 벡터 뒤에 붙인다.
2. 같은 쌍 비교 학습. 단, 임베딩 부분의 λ를 10배 크게 (차원이 많아 과적합).
3. 임베딩 모델은 고정, 선형 헤드만 학습. LAION aesthetic predictor가 정확히 이 구조.
4. 콜드 스타트: Photo Triage 데이터셋으로 헤드의 w0를 미리 학습.

비전 모델(LLM)에 매번 물어보는 구조는 필요 없다. 비용과 프라이버시 문제가 같이 사라진다.

## 출처

- Bradley, Terry, "Rank Analysis of Incomplete Block Designs", Biometrika 1952
- Burges 외, "Learning to Rank using Gradient Descent" (RankNet), ICML 2005
- Christiano 외, "Deep RL from Human Preferences", NeurIPS 2017 — https://arxiv.org/abs/1706.03741
- Evgeniou, Pontil, "Regularized Multi-Task Learning", KDD 2004
- Li 외, "Ditto: Fair and Robust Federated Learning Through Personalization", ICML 2021 — https://arxiv.org/abs/2012.04221
- Ren 외, "Personalized Image Aesthetics", ICCV 2017 — 일반 점수 + 개인 오프셋 구조
- Joachims 외, "Accurately Interpreting Clickthrough Data as Implicit Feedback", SIGIR 2005
- Joachims, Swaminathan, Schnabel, "Unbiased Learning-to-Rank with Biased Feedback", WSDM 2017 — https://arxiv.org/abs/1608.04468
- Ross, Gordon, Bagnell, "DAgger", AISTATS 2011 — https://arxiv.org/abs/1011.0686
- Chang 외, "Automatic Triage for a Photo Series", SIGGRAPH 2016 — https://gfx.cs.princeton.edu/pubs/Chang_2016_ATF/index.php
