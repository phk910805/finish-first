# 07. 시장: 기본 앱, 경쟁 앱, 페인포인트

2026-10-03 조사. 회고 때 "뭘 다르게 할 것인가"를 정할 때 연다.

## 기본 앱이 이미 하는 것

| 앱 | 완전 중복 삭제 | 유사 사진 묶기 | 유사 중 베스트 남기고 삭제 | 흐림/스크린샷/대용량 |
|---|---|---|---|---|
| 애플 사진 iOS 16+ | O. 해상도·포맷만 다른 사본까지 병합 | X | X | X |
| 구글 포토 | 업로드 시 동일 파일만 차단 | O. 스택(2023.11~), 기본 오프 | O. "이것만 남기고 삭제" | O. 저장공간 관리 |
| 삼성 갤러리 | O. 바이트 동일만 | O. 유사 이미지 그룹화, 보기 전용 | X | 디바이스 케어 |
| Files by Google | O. 동일 파일만 | X | X | O |

- 애플 중복 앨범은 "완전 사본이거나 해상도·포맷·메타데이터만 다른 같은 사진"만. 연사·각도 차이는 못 잡는다는 불만이 꾸준함. iOS 26에서도 동일.
- 구글 포토: 2025년 6월 APK 분석에서 "삭제할 만한 사진을 찾았습니다" 추천 기능 준비 중. 빈자리가 좁아지는 중.
- 구글 Top Shot(Pixel 3~), 삼성 Best Face(One UI 7~)는 촬영 시점에 프레임을 고르는 기능. 이미 찍힌 사진첩 정리와는 별개.

## 경쟁 앱: Clever Cleaner (iOS, CleverFiles)

- 2025년 초 "광고·구독·유료 해금 없음"으로 출시, 평점 4.8에 리뷰 8만. 3.0부터 자동 정리 도구에 일일 한도, Premium 주 6.99달러 또는 평생 39.99달러. 3.0 이전 설치자는 무제한 유지.
- 유저 불만: 한도 때문에 정리 도중 멈춤. 유사 판정이 95%쯤 맞지만 일러스트·애니 놓침, 분위기만 다른 사진을 묶음. AI가 고른 베스트가 취향과 다름. iCloud 미다운로드 사진 누락. 유사 기능이 전부 훑어야 삭제 활성화되게 바뀜. 오삭제 공포. 전체 사진 접근 권한 불안.
- Android 버전 없음.

그 외 iOS: CleanMy®Phone(구 Gemini Photos), Remo, PicPicks(1대1 비교), Siftly(스와이프), NeatPix(토너먼트). 대부분 주 단위 구독. Android는 광고 범벅 앱이 많고 품질 낮음.

## 베스트 컷 기능의 페인포인트

1. AI 선택이 취향과 다름. 기술 점수 높은 사진이 "순간"을 놓침.
2. 눈 감음 판정이 맥락 무시. 배경 인물도 감점.
3. 그룹핑이 사진을 숨김. "사진이 사라졌다"는 착각 (구글 포토 스택).
4. 연속 두 장이 각각 남길 만한데 중복 처리.
5. 결국 전부 다시 봐서 시간 절약 체감 없음 (프로 컬링 툴 공통).
6. 그래서 최근 앱은 자동 삭제 대신 스와이프·비교·토너먼트 UI. AI는 후보를 줄이고 결정은 사람.

→ 우리 답: 유저 얼굴 우선 규칙 + 사용자 선택으로 취향 학습 + 삭제는 항상 사람이.

## 스크린샷 (보류, 별개 제품)

- Pixel Screenshots(2024, Pixel 9 전용, Gemini Nano OCR·요약·검색), 삼성 S26 Screenshot Analyzer(갤러리 내 자동 분류), iOS 26 Visual Intelligence(캘린더 추가·검색). 플랫폼이 직접 들어온 영역.
- 서드파티: Sorti, Captr, SnapStash, RemindShot(리마인더 결합).
- 페인포인트: 왜 찍었는지 기억 안 남, 검색 안 됨, 유효기간 지나도 남음, 기존 수천 장 가져오기 안 됨, 클라우드 처리 불안, AI 환각, 구독.
- 연구: Mottelson, "Why do people take Screenshots on their Smartphones?", DIS 2023. 52명 1,679장, 9가지 동기. 증거·감정 용도가 섞여 있어 일괄 삭제 위험.

## 출처

- Apple Support, 중복 앨범 — https://support.apple.com/en-us/102260
- 9to5Google, Google Photos Stacks — https://9to5google.com/2023/11/15/google-photos-stacks/
- Android Authority, Google Photos 삭제 추천 APK 분석 — https://www.androidauthority.com/google-photos-delete-suggestions-apk-3570803/
- Android Police, 삼성 갤러리 중복 삭제 — https://www.androidpolice.com/the-samsung-gallery-feature-i-wish-google-photos-would-copy/
- Clever Cleaner 가격 변화 — https://www.handyrecovery.com/best-iphone-storage-cleaner-apps/
- Clever Cleaner 리뷰 — https://justuseapp.com/en/app/1666645584/clever-cleaner-ai-cleanup-app/reviews
- Google Research, Top Shot — https://research.google/blog/top-shot-on-pixel-3/
- FilterPixel, AI 컬링 툴 비교 — https://filterpixel.com/ai-photo-culling-software
- Android Authority, Pixel Screenshots 1,800장 — https://www.androidauthority.com/pixel-screenshots-review-3476772/
- Mottelson, DIS 2023 — https://dl.acm.org/doi/10.1145/3563657.3596067
