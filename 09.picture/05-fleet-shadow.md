# 05. 플릿 학습과 섀도 모드 (4사이클 이후)

테슬라 데이터 엔진의 축소판. 차이는 "사진은 절대 안 올린다"는 것.

## 운전 데이터와의 대응

| 운전 | 이 앱 |
|---|---|
| 개입(운전자가 핸들 잡음)이 트리거 | 추천을 뒤집은 선택이 트리거 |
| 트리거 시 클립 업로드 | 피처 벡터 + 선택 인덱스만 업로드. 이미지·얼굴 임베딩 제외 |
| 중앙에서 모델 학습, 차량에 배포 | 중앙에서 w0 학습, 신규 유저의 초기값으로 배포 |
| 섀도 모드로 새 모델을 조용히 비교 | 새 w'의 추천을 숨기고 실제 선택과 일치율 비교 |
| 개입 없는 주행은 약한 신호 | 추천 수락은 약한 신호 |

## 무엇이 기기 밖으로 나가나

- 나가는 것: `features_json`(숫자 벡터), `rec_index`, `chosen_index`, `shown_with_rec`, 앱 버전. 익명 디바이스 ID.
- 안 나가는 것: 사진, 축소본, 얼굴 임베딩, 유저 템플릿, 파일명, 촬영 시각 원값.
- 피처 벡터만으로는 사진을 복원할 수 없다. 그래도 사용자 동의 토글과 설명은 필수. Pixel Screenshots가 클라우드로 옮겼다가 받은 반발을 기억할 것.

## 전역 가중치 w0

- 모든 유저의 쌍을 모아 같은 쌍 비교 로지스틱으로 학습. 수천 쌍이면 충분.
- 개인 가중치 = w0 + δ. δ는 기기에서 학습하고 λ로 w0에 묶는다 (Ditto 구조).
- 임베딩 헤드까지 플릿 학습으로 가면 그때 FedAvg와 차등 프라이버시를 검토. 그 전엔 과하다.

## 섀도 모드 프로토콜

1. 후보 w'를 만든다 (새 피처 추가, 다른 λ, 전역 w0 갱신 등).
2. 앱은 현재 w로 추천을 보여주고, 동시에 w'의 추천을 로그에만 기록.
3. `shown_with_rec=0` 그룹에서 w와 w'의 top-1 일치율을 비교.
4. w'가 N개 그룹에서 통계적으로 우세하면 교체. N은 50부터.
5. 두 추천기를 같은 화면에 섞어 보여주는 인터리빙은 사진 선택 UI엔 부자연스러워 쓰지 않는다.

## 반사실 평가 한 줄

로그는 "현재 w가 보여준 상태"에서 수집됐으므로 다른 w'를 평가하면 편향이 있다. `shown_with_rec=0` 로그가 그 편향을 끊는 무작위화다. 더 정교하게 하려면 역성향 가중(IPS). 사이클 6 이전엔 불필요.

## 출처

- Karpathy, Tesla AI 워크숍 강연, CVPR 2021 (데이터 엔진, 섀도 모드)
- McMahan 외, "Communication-Efficient Learning of Deep Networks from Decentralized Data" (FedAvg), AISTATS 2017 — https://arxiv.org/abs/1602.05629
- Abadi 외, "Deep Learning with Differential Privacy", CCS 2016 — https://arxiv.org/abs/1607.00133
- Bottou 외, "Counterfactual Reasoning and Learning Systems", JMLR 2013 — https://arxiv.org/abs/1209.2355
- Chapelle 외, "Large-Scale Validation and Analysis of Interleaved Search Evaluation", TOIS 2012
- Li 외, "Ditto", ICML 2021 — https://arxiv.org/abs/2012.04221
