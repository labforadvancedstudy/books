# iOS 실전 프로젝트 - App Store 출시 가능한 3개 앱

## 🚀 프로젝트 개요

이 폴더는 즉시 App Store에 출시 가능한 3개의 완전한 iOS 앱 프로젝트를 포함합니다.

### 📱 포함된 앱

1. **WeatherNow** - 실시간 날씨 앱
2. **TaskMaster** - 스마트 할일 관리 앱  
3. **RecipeBox** - AI 기반 레시피 앱

---

## 1️⃣ WeatherNow - 실시간 날씨 앱

### 주요 기능
- ✅ 실시간 날씨 정보 (OpenWeatherMap API)
- ✅ 시간별/일별/주간 예보
- ✅ 여러 도시 저장 및 관리
- ✅ 홈 화면 위젯 (소형/중형/대형)
- ✅ 잠금 화면 위젯
- ✅ Live Activities (Dynamic Island)
- ✅ 날씨 변화 알림
- ✅ 오프라인 캐싱
- ✅ 다크 모드 완벽 지원
- ✅ 한국어/영어 지원

### 기술 스택
- SwiftUI 6.0
- WidgetKit
- ActivityKit
- Core Location
- URLSession
- UserNotifications

### 프로젝트 구조
```
weathernow/
├── WeatherNow.xcodeproj
├── WeatherNow/
│   ├── App/
│   ├── Features/
│   │   ├── Weather/
│   │   ├── Location/
│   │   └── Settings/
│   ├── Core/
│   │   ├── Network/
│   │   ├── Storage/
│   │   └── Extensions/
│   └── Resources/
├── WeatherWidget/
└── WeatherNowTests/
```

---

## 2️⃣ TaskMaster - 스마트 할일 관리 앱

### 주요 기능
- ✅ 작업 생성/편집/삭제
- ✅ 프로젝트별 작업 분류
- ✅ 우선순위 및 마감일 설정
- ✅ 하위 작업 관리
- ✅ 태그 시스템
- ✅ 반복 작업 설정
- ✅ CloudKit 실시간 동기화
- ✅ 팀 협업 (공유 프로젝트)
- ✅ Siri Shortcuts
- ✅ 위젯 및 App Intents
- ✅ 캘린더 통합
- ✅ 알림 및 리마인더

### 기술 스택
- SwiftUI 6.0
- Core Data
- CloudKit
- App Intents
- EventKit
- UserNotifications

### 프로젝트 구조
```
taskmaster/
├── TaskMaster.xcodeproj
├── TaskMaster/
│   ├── App/
│   ├── Presentation/
│   │   ├── Views/
│   │   ├── ViewModels/
│   │   └── Components/
│   ├── Domain/
│   │   ├── Entities/
│   │   ├── UseCases/
│   │   └── Repositories/
│   ├── Data/
│   │   ├── Local/
│   │   └── Remote/
│   └── Core/
├── TaskMasterWidget/
└── TaskMasterTests/
```

---

## 3️⃣ RecipeBox - AI 기반 레시피 앱

### 주요 기능
- ✅ 카메라로 재료 인식 (Vision API)
- ✅ AI 레시피 추천 (Core ML)
- ✅ 레시피 검색 및 필터링
- ✅ 즐겨찾기 및 컬렉션
- ✅ 요리 타이머 및 단계별 가이드
- ✅ 쇼핑 리스트 자동 생성
- ✅ 영양 정보 분석
- ✅ 소셜 공유 기능
- ✅ 사용자 리뷰 및 평점
- ✅ 인앱 구매 (프리미엄 레시피)

### 기술 스택
- SwiftUI 6.0
- Core ML
- Vision
- AVFoundation
- StoreKit 2
- SwiftData
- PhotosUI

### 프로젝트 구조
```
recipebox/
├── RecipeBox.xcodeproj
├── RecipeBox/
│   ├── App/
│   ├── Features/
│   │   ├── Camera/
│   │   ├── Recipes/
│   │   ├── AI/
│   │   └── Social/
│   ├── Models/
│   ├── Services/
│   └── Resources/
├── RecipeBoxWidget/
└── RecipeBoxTests/
```

---

## 🛠 개발 환경 설정

### 필수 요구사항
- Xcode 16.0+
- iOS 18.0+ SDK
- macOS Sonoma 14.0+
- Swift 6.0+

### 설치 방법

1. **프로젝트 클론**
```bash
git clone https://github.com/yourusername/ios-practical-projects.git
cd ios-practical-projects
```

2. **각 앱별 설정**

#### WeatherNow
```bash
cd weathernow
open WeatherNow.xcodeproj

# Info.plist에 API 키 추가
# OPENWEATHER_API_KEY: "your-api-key"
```

#### TaskMaster
```bash
cd taskmaster
open TaskMaster.xcodeproj

# CloudKit 컨테이너 설정
# Signing & Capabilities에서 CloudKit 활성화
```

#### RecipeBox
```bash
cd recipebox
open RecipeBox.xcodeproj

# Core ML 모델 다운로드
# Resources/ML Models 폴더에 추가
```

---

## 📦 빌드 및 배포

### 개발 빌드
```bash
# 각 프로젝트 폴더에서
xcodebuild -scheme AppName -configuration Debug
```

### TestFlight 배포
```bash
# Fastlane 사용
fastlane beta
```

### App Store 출시
```bash
# Fastlane 사용
fastlane release
```

---

## 💰 수익화 전략

### WeatherNow
- **무료 + 광고**: 기본 기능 무료, 배너 광고
- **프리미엄 구독**: ₩3,300/월
  - 광고 제거
  - 무제한 도시
  - 레이더 지도
  - 1시간 단위 알림

### TaskMaster
- **Freemium**: 기본 기능 무료
- **Pro 구독**: ₩5,500/월
  - 무제한 프로젝트
  - 팀 협업
  - 고급 자동화
  - 우선 지원

### RecipeBox
- **무료 + IAP**: 기본 레시피 무료
- **프리미엄 레시피**: ₩1,100/레시피
- **Chef 구독**: ₩8,800/월
  - 모든 프리미엄 레시피
  - AI 개인화 추천
  - 영양사 상담

---

## 🧪 테스트

### 유닛 테스트
```swift
// 각 프로젝트의 Tests 폴더 참조
swift test
```

### UI 테스트
```swift
// UITests 폴더 참조
xcodebuild test -scheme AppNameUITests
```

### 성능 테스트
- Memory Graph Debugger 사용
- Instruments 프로파일링
- MetricKit 통합

---

## 📱 스크린샷

### WeatherNow
- 메인 화면: 현재 날씨와 예보
- 위젯: 홈 화면 날씨 정보
- Live Activity: Dynamic Island 날씨

### TaskMaster
- 작업 목록: 우선순위별 정렬
- 프로젝트 뷰: 칸반 보드
- 캘린더: 일정 통합

### RecipeBox
- 카메라: 재료 인식
- 레시피 상세: 단계별 가이드
- AI 추천: 개인화된 레시피

---

## 🚀 출시 체크리스트

### 모든 앱 공통
- [ ] 앱 아이콘 (1024x1024)
- [ ] 스크린샷 (모든 디바이스)
- [ ] 앱 설명 (한국어/영어)
- [ ] 키워드 최적화
- [ ] 개인정보 처리방침
- [ ] 이용약관
- [ ] 지원 URL
- [ ] 연령 등급 설정

### 앱별 특별 요구사항

#### WeatherNow
- [ ] 위치 권한 설명
- [ ] 네트워크 사용 설명
- [ ] API 키 보안

#### TaskMaster
- [ ] CloudKit 설정
- [ ] 캘린더 권한 설명
- [ ] 알림 권한 설명

#### RecipeBox
- [ ] 카메라 권한 설명
- [ ] 사진 라이브러리 권한
- [ ] 인앱 구매 설정

---

## 📈 마케팅 전략

### ASO (App Store Optimization)
1. 키워드 연구 및 최적화
2. 현지화 (10개국 언어)
3. A/B 테스트 (아이콘, 스크린샷)
4. 정기 업데이트

### 사용자 획득
1. 소셜 미디어 마케팅
2. 인플루언서 협업
3. 앱 리뷰 사이트
4. Apple Search Ads

### 리텐션 전략
1. 온보딩 최적화
2. 푸시 알림 개인화
3. 이메일 마케팅
4. 인앱 이벤트

---

## 🤝 기여 가이드

1. Fork the repository
2. Create feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open Pull Request

---

## 📄 라이선스

MIT License - 자유롭게 사용하되 출처를 명시해주세요.

---

## 📞 지원

- 이메일: support@yourcompany.com
- 웹사이트: https://yourcompany.com
- 트위터: @yourcompany

---

## 🎉 성공 지표

### 목표 (첫 3개월)
- **다운로드**: 각 앱 10,000+
- **DAU**: 3,000+
- **평점**: 4.5+ ⭐️
- **수익**: $5,000/월

### 장기 목표 (1년)
- **다운로드**: 각 앱 100,000+
- **DAU**: 30,000+
- **평점**: 4.7+ ⭐️
- **수익**: $50,000/월

---

**이제 App Store에서 성공할 준비가 되었습니다! 🚀**