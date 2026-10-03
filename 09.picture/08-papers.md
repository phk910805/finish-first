# 08. 논문 서지

사이클 열이 비어 있으면 "그 이후". 링크는 arXiv 또는 공식 페이지.

## 이 문제를 직접 다룬 것

| 논문 | 요지 | 사이클 |
|---|---|---|
| Chang, Yu, Wang, Ashley, Finkelstein, **Automatic Triage for a Photo Series**, SIGGRAPH 2016 — https://gfx.cs.princeton.edu/pubs/Chang_2016_ATF/index.php | 연속 촬영 시리즈 내 상대 선호를 쌍 비교로 수집. 15,545장, 5,953 시리즈, 모델·벤치마크 공개. w0 초기화와 오프라인 평가에 사용 | 3 |
| Google Research, **Top Shot on Pixel 3**, 2018 — https://research.google/blog/top-shot-on-pixel-3/ | MobileNet 온디바이스로 블러·눈·미소·표정 판정. 블로그지만 설계 참고 | |

## 얼굴: 검출, 임베딩, 품질

| 논문 | 요지 | 사이클 |
|---|---|---|
| Bazarevsky 외, **BlazeFace**, 2019 — https://arxiv.org/abs/1907.05047 | 모바일 얼굴 검출. MediaPipe | |
| Schroff 외, **FaceNet**, CVPR 2015 — https://arxiv.org/abs/1503.03832 | 얼굴 임베딩의 출발점 | |
| Deng 외, **ArcFace**, CVPR 2019 — https://arxiv.org/abs/1801.07698 | 각도 마진 손실. 현재 표준 | |
| Chen 외, **MobileFaceNets**, 2018 — https://arxiv.org/abs/1804.07573 | 모바일용 얼굴 인식, 약 4MB | 2 |
| Meng 외, **MagFace**, CVPR 2021 — https://arxiv.org/abs/2103.06627 | 임베딩 노름 = 품질. 인식과 품질을 한 모델로 | 2 |
| Terhörst 외, **SER-FIQ**, CVPR 2020 — https://arxiv.org/abs/2003.09373 | 비지도 얼굴 품질 | |
| Schlett 외, **Face Image Quality Assessment: A Literature Survey**, ACM CSUR 2022 — https://dl.acm.org/doi/10.1145/3507901 | 서베이 | |
| Song 외, **Eyes closeness detection** (CEW 데이터셋), Pattern Recognition 2014 | 눈 감음 데이터셋 | |

## 주인공 판별

| 논문 | 요지 | 사이클 |
|---|---|---|
| Mathialagan, Gallagher, Batra, **VIP: Finding Important People in Images**, CVPR 2015 — https://arxiv.org/abs/1502.05678 | 신원 없이 시각 단서로 중요 인물 예측 | 2 |
| Li, Hong, Zheng, **Learning to Learn Relation for Important People Detection**, CVPR 2020 | 관계 기반 후속 | |
| Oh 외, **Person Recognition in Personal Photo Collections**, ICCV 2015 / TPAMI — https://arxiv.org/abs/1710.03224 | 개인 사진첩 맥락의 인물 인식 | |
| Biemann, **Chinese Whispers**, 2006 | 그래프 클러스터링. dlib 얼굴 클러스터링 | 2 |

## 사진 품질과 심미

| 논문 | 요지 | 사이클 |
|---|---|---|
| Pertuz, Puig, Garcia, **Analysis of focus measure operators for shape-from-focus**, Pattern Recognition 2013 | 라플라시안 분산 포함 초점 척도 비교 | |
| Pech-Pacheco 외, **Diatom autofocusing in brightfield microscopy**, ICPR 2000 | 라플라시안 분산 원전 | |
| Murray, Marchesotti, Perronnin, **AVA**, CVPR 2012 | 심미 데이터셋 | |
| Talebi, Milanfar, **NIMA**, IEEE TIP 2018 — https://arxiv.org/abs/1709.05424 | 심미·기술 품질 분포 예측 | |
| Schuhmann, **LAION Aesthetic Predictor**, 2022 — https://github.com/christophschuhmann/improved-aesthetic-predictor | CLIP 임베딩 + 선형/MLP 헤드 | 4+ |
| Wang, Chan, Loy, **CLIP-IQA**, AAAI 2023 — https://arxiv.org/abs/2207.12396 | CLIP으로 품질 평가 | |
| Ren 외, **Personalized Image Aesthetics**, ICCV 2017 | 일반 점수 + 개인 오프셋, 소수 샘플 | 3 |
| Yang 외, **Scaling Up Personalized Image Aesthetic Assessment**, ECCV 2024 — https://arxiv.org/abs/2407.07176 | PIAA 최신 | |
| Radford 외, **CLIP**, ICML 2021 — https://arxiv.org/abs/2103.00020 | 임베딩 백본 | 4+ |

## 선호 학습과 편향

| 논문 | 요지 | 사이클 |
|---|---|---|
| Bradley, Terry, **Rank Analysis of Incomplete Block Designs**, Biometrika 1952 | 쌍 비교 모델 | 3 |
| Burges 외, **Learning to Rank using Gradient Descent** (RankNet), ICML 2005 | 쌍 비교 로지스틱 + SGD. 우리 업데이트 식 | 3 |
| Christiano 외, **Deep RL from Human Preferences**, NeurIPS 2017 — https://arxiv.org/abs/1706.03741 | 같은 모델을 보상 학습에 사용 | |
| Evgeniou, Pontil, **Regularized Multi-Task Learning**, KDD 2004 | 공통 가중치로 당기는 정규화 | |
| Li 외, **Ditto**, ICML 2021 — https://arxiv.org/abs/2012.04221 | 개인화 연합학습. w0 + δ | |
| Joachims 외, **Accurately Interpreting Clickthrough Data as Implicit Feedback**, SIGIR 2005 | 제시 편향 | 3 |
| Joachims, Swaminathan, Schnabel, **Unbiased Learning-to-Rank with Biased Feedback**, WSDM 2017 — https://arxiv.org/abs/1608.04468 | 편향 보정 학습 | |
| Ross, Gordon, Bagnell, **DAgger**, AISTATS 2011 — https://arxiv.org/abs/1011.0686 | 교정에서 배우는 구조의 이론 | |

## 플릿 학습, 평가, 프라이버시

| 논문 | 요지 | 사이클 |
|---|---|---|
| McMahan 외, **FedAvg**, AISTATS 2017 — https://arxiv.org/abs/1602.05629 | 연합학습 | |
| Abadi 외, **Deep Learning with Differential Privacy**, CCS 2016 — https://arxiv.org/abs/1607.00133 | DP-SGD | |
| Bottou 외, **Counterfactual Reasoning and Learning Systems**, JMLR 2013 — https://arxiv.org/abs/1209.2355 | 로그 기반 반사실 평가 | |
| Chapelle 외, **Large-Scale Validation and Analysis of Interleaved Search Evaluation**, TOIS 2012 | 인터리빙 | |

## 유사 이미지

| 논문 | 요지 | 사이클 |
|---|---|---|
| Zauner, **Implementation and Benchmarking of Perceptual Image Hash Functions**, 2010 (석사논문) | pHash 등 비교 | 1 |

## 스크린샷 (보류)

| 논문 | 요지 |
|---|---|
| Mottelson, **Why do people take Screenshots on their Smartphones?**, DIS 2023 — https://dl.acm.org/doi/10.1145/3563657.3596067 | 9가지 동기 |
