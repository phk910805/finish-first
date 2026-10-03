# 06. 플랫폼 API, 권한, 배포

## Android (1사이클)

### 권한
- Android 13+: `READ_MEDIA_IMAGES`. Android 14+: 사용자가 "선택한 사진만" 허용 가능 (`READ_MEDIA_VISUAL_USER_SELECTED`). 12 이하: `READ_EXTERNAL_STORAGE`.
- Play 스토어 정책(2024~): 광범위한 사진 권한을 쓰는 앱은 "Photo and Video Permissions" 선언 양식을 제출해야 한다. 갤러리·사진 관리 앱은 핵심 용도로 인정되는 범주. 사이드로드 APK엔 해당 없음.
- Photo Picker는 권한 없이 쓸 수 있지만 삭제 권한이 없어 우리 용도엔 부적합.

### 읽기
```kotlin
val projection = arrayOf(
    MediaStore.Images.Media._ID,
    MediaStore.Images.Media.DATE_TAKEN,
    MediaStore.Images.Media.DATE_ADDED,
    MediaStore.Images.Media.ORIENTATION,
    MediaStore.Images.Media.WIDTH,
    MediaStore.Images.Media.HEIGHT,
)
val sort = "${MediaStore.Images.Media.DATE_TAKEN} DESC"
// Bundle 기반 query()로 LIMIT 500. 축소본은 contentResolver.loadThumbnail(uri, Size(512, 512), null)
```

### 삭제 (2사이클)
```kotlin
val pi = MediaStore.createDeleteRequest(contentResolver, uris)  // API 30+
startIntentSenderForResult(pi.intentSender, REQ, null, 0, 0, 0)
// 시스템 확인창이 뜬다. 묶음 삭제 가능. 앱이 조용히 지울 방법은 없다.
```

### 백그라운드
- `WorkManager`로 해시·얼굴 피처 계산. 충전 중 + 유휴 제약.

### 1사이클 배포
- `./gradlew assembleDebug` → `app-debug.apk`를 메신저로 전송. 받는 쪽은 "출처를 알 수 없는 앱 허용".
- 서명·스토어·개발자 계정 전부 불필요.

## iOS (4사이클 이후)

### 권한
- `Info.plist`에 `NSPhotoLibraryUsageDescription`. 삭제까지 하려면 `PHPhotoLibrary.requestAuthorization(for: .readWrite)`.
- iOS 14+ "선택한 사진만" 모드가 있다. 제한 모드면 `PHPhotoLibrary.shared().presentLimitedLibraryPicker`로 추가 선택 유도.

### 읽기
```swift
let opts = PHFetchOptions()
opts.sortDescriptors = [NSSortDescriptor(key: "creationDate", ascending: false)]
opts.fetchLimit = 500
let assets = PHAsset.fetchAssets(with: .image, options: opts)
// PHImageManager.default().requestImage(for:targetSize:contentMode:options:)
// PHImageRequestOptions.isNetworkAccessAllowed = false  → iCloud 미다운로드 사진 건너뜀
// PHAsset.burstIdentifier, representsBurst, isFavorite, creationDate
```

### 삭제
```swift
PHPhotoLibrary.shared().performChanges {
    PHAssetChangeRequest.deleteAssets(assets as NSFastEnumeration)
}
// "N장 삭제 허용" 시스템 팝업. 삭제본은 최근 삭제됨에 30일 보관.
```

### 배포
- TestFlight 내부 테스터(최대 100명)는 베타 심사 없이 빌드 업로드 후 바로 초대 가능. Apple Developer Program 연 99달러와 Mac·Xcode 필요.

## 모델 런타임

| 런타임 | iOS | Android | 용도 |
|---|---|---|---|
| Vision / Core ML | O | X | 얼굴 검출·품질·feature print. Core ML 변환은 `coremltools` |
| ML Kit | O | O | 얼굴 검출, 눈·미소 확률, 고개 각도 |
| MediaPipe Tasks | O | O | 이미지 임베더, 얼굴 검출(BlazeFace) |
| LiteRT (TFLite) | O | O | MobileFaceNet 같은 직접 넣는 모델 |

## 출처

- Android: Photo picker / media permissions — https://developer.android.com/training/data-storage/shared/media
- Android: `MediaStore.createDeleteRequest` — https://developer.android.com/reference/android/provider/MediaStore#createDeleteRequest(android.content.ContentResolver,%20java.util.Collection%3Candroid.net.Uri%3E)
- Google Play: Photo and Video Permissions policy — https://support.google.com/googleplay/android-developer/answer/14115180
- Apple PhotoKit — https://developer.apple.com/documentation/photokit
- Apple: Delivering a limited library — https://developer.apple.com/documentation/photokit/delivering-an-enhanced-privacy-experience-in-your-photos-app
- Apple Vision — https://developer.apple.com/documentation/vision
- MediaPipe Image Embedder iOS — https://developers.google.com/edge/mediapipe/solutions/vision/image_embedder/ios
