# 실전 iOS 개발 가이드 2025
## 3개 앱을 만들며 배우는 실무 중심 Swift/SwiftUI

---

> **"The best way to learn is to build real things that people actually use."**

이 책은 철학이 아닌 **결과물**에 집중합니다. 3개의 완전한 앱을 처음부터 App Store 출시까지, 실제 사용자가 다운받을 수 있는 수준으로 만들어봅니다.

---

## 🎯 이 책의 약속

### ✅ 실제로 만들 것들
1. **WeatherNow** - 실시간 날씨 앱 (API, 위치, 위젯, 알림)
2. **TaskMaster** - 할일 관리 앱 (Core Data, 동기화, 공유)
3. **RecipeBox** - 레시피 앱 (카메라, ML, 소셜 기능)

### ✅ 보장하는 것들
- **모든 코드가 2025년 기준 실제 작동함** (Xcode 16, iOS 18, Swift 6)
- **App Store 출시까지 완주** (심사 가이드, 스크린샷, 마케팅)
- **한국 시장 특화** (결제, 현지화, 법적 요구사항)
- **실무 레벨 품질** (테스트, 성능, 디버깅)

### ❌ 하지 않는 것들
- 추상적 철학 논의
- 당장 쓸 수 없는 이론
- 실행되지 않는 예제 코드
- 현실성 없는 복잡한 설계

---

## 📋 학습 경로

### 🚀 빠른 시작 (1주)
실무에 바로 써야 하는 개발자용:
- **L0**: Hello World → 실제 앱 (1일)
- **L1**: WeatherNow 완성 (3일) 
- **L2**: TaskMaster 완성 (3일)

### 🏗️ 완벽 마스터 (1개월)
제대로 배우고 싶은 개발자용:
- **L0-L3**: 3개 앱 모두 완성
- **L4**: App Store 출시 및 운영
- **L5**: 고급 최적화 및 확장

### 🇰🇷 한국 시장 특화 (추가 1주)
국내 서비스 개발자용:
- 결제 시스템 연동 (토스, 카카오페이)
- 정부 API 활용 (공공데이터, 기상청)
- 국내 법적 요구사항 (개인정보보호법, 위치정보법)

---

## 📖 상세 목차

### [L0: Hello World → 실제 앱](L0_hello_to_real/)
**목표**: "배고파" 예제를 벗어나 실제 앱 만들기

- **[00. 진짜 Hello World](L0_hello_to_real/00_real_hello.md)**
  ```swift
  // 90%의 책이 여기서 멈춤
  Text("Hello, World!")
  
  // 우리는 여기서 시작
  struct WeatherApp: App {
      @main var body: some Scene { ... }
  }
  ```

- **[01. 첫 화면 만들기](L0_hello_to_real/01_first_screen.md)**
  - NavigationView와 기본 레이아웃
  - 실제 앱 같은 UI 구조
  - 아이콘과 색상 시스템

- **[02. 데이터와 연결](L0_hello_to_real/02_connect_data.md)**
  - API 호출의 기초
  - JSON 파싱과 모델
  - 로딩 상태 처리

**결과물**: 실제 날씨 데이터를 보여주는 앱 (시뮬레이터에서 실행 가능)

---

### [L1: WeatherNow - 실시간 날씨 앱](L1_weather_app/)
**목표**: 프로덕션 수준의 완성된 날씨 앱

- **[00. 프로젝트 설정](L1_weather_app/00_project_setup.md)**
  ```bash
  # 실제 명령어 제공
  xcodebuild -version  # Xcode 16.0 이상 확인
  swift --version      # Swift 6.0 이상 확인
  ```

- **[01. 날씨 API 연동](L1_weather_app/01_weather_api.md)**
  - OpenWeatherMap API 키 발급
  - HTTPClient 구현
  - 에러 처리와 재시도

- **[02. 위치 기반 서비스](L1_weather_app/02_location_service.md)**
  - CoreLocation 권한 처리
  - 현재 위치 자동 감지
  - 도시 검색 기능

- **[03. UI/UX 완성](L1_weather_app/03_complete_ui.md)**
  - 날씨에 따른 배경 애니메이션
  - 다크 모드 대응
  - 접근성 (VoiceOver) 지원

- **[04. 위젯과 알림](L1_weather_app/04_widget_notification.md)**
  - WidgetKit으로 홈화면 위젯
  - 일기예보 푸시 알림
  - Live Activities (iOS 16+)

- **[05. 성능과 테스트](L1_weather_app/05_performance_test.md)**
  - 네트워크 캐싱 전략
  - 단위 테스트 작성
  - UI 테스트 자동화

**결과물**: App Store 출시 가능한 완성된 날씨 앱

---

### [L2: TaskMaster - 할일 관리 앱](L2_todo_app/)
**목표**: 데이터 저장과 동기화가 완벽한 앱

- **[00. Core Data 설계](L2_todo_app/00_core_data_design.md)**
  - 데이터 모델 설계
  - 관계형 데이터 처리
  - 마이그레이션 전략

- **[01. CRUD 기능 완성](L2_todo_app/01_crud_complete.md)**
  - 할일 생성/수정/삭제
  - 카테고리와 태그 시스템
  - 검색과 필터링

- **[02. 동기화와 백업](L2_todo_app/02_sync_backup.md)**
  - CloudKit 자동 동기화
  - 오프라인 우선 설계
  - 충돌 해결 전략

- **[03. 공유와 협업](L2_todo_app/03_share_collaborate.md)**
  - 할일 목록 공유
  - 실시간 협업 기능
  - 권한 관리 시스템

- **[04. 위젯과 Shortcuts](L2_todo_app/04_widget_shortcuts.md)**
  - 홈화면 위젯으로 빠른 추가
  - Siri Shortcuts 지원
  - Control Center 확장

**결과물**: Todoist/Things 급의 완성된 할일 관리 앱

---

### [L3: RecipeBox - 레시피 앱](L3_recipe_app/)
**목표**: AI/ML과 소셜 기능이 포함된 고급 앱

- **[00. 카메라와 갤러리](L3_recipe_app/00_camera_gallery.md)**
  - 음식 사진 촬영
  - 갤러리에서 이미지 선택
  - 이미지 편집과 크롭

- **[01. AI 재료 인식](L3_recipe_app/01_ai_ingredient.md)**
  - Vision 프레임워크 활용
  - CoreML 모델 통합
  - 재료 자동 인식과 추출

- **[02. 레시피 생성과 편집](L3_recipe_app/02_recipe_editor.md)**
  - 단계별 요리 과정
  - 타이머와 알림
  - 영양 정보 계산

- **[03. 소셜 기능](L3_recipe_app/03_social_features.md)**
  - 레시피 공유와 평점
  - 팔로우와 피드
  - 댓글과 좋아요

- **[04. 추천과 검색](L3_recipe_app/04_recommendation.md)**
  - AI 기반 레시피 추천
  - 고급 검색 필터
  - 개인 맞춤 알고리즘

**결과물**: Instagram + 만개의레시피 수준의 소셜 레시피 앱

---

### [L4: App Store 출시](L4_app_store/)
**목표**: 3개 앱을 실제 App Store에 출시

- **[00. 출시 준비](L4_app_store/00_launch_prep.md)**
  - App Store Connect 설정
  - 앱 메타데이터 작성
  - 스크린샷과 프리뷰 제작

- **[01. 심사 통과 전략](L4_app_store/01_review_strategy.md)**
  - 심사 가이드라인 체크
  - 거부 사유 미리 방지
  - 재심사 대응 방법

- **[02. 마케팅과 ASO](L4_app_store/02_marketing_aso.md)**
  - App Store 최적화
  - 키워드 전략
  - 초기 사용자 확보

- **[03. 런칭 후 운영](L4_app_store/03_post_launch.md)**
  - 사용자 피드백 대응
  - 크래시 모니터링
  - 업데이트 전략

**결과물**: 실제 다운로드 가능한 3개의 App Store 앱

---

### [L5: 한국 시장 특화](L5_korean_market/)
**목표**: 국내 서비스에 최적화된 기능

- **[00. 결제 시스템](L5_korean_market/00_payment_system.md)**
  - 토스페이먼츠 연동
  - 카카오페이 결제
  - 구독 관리 시스템

- **[01. 정부 API 활용](L5_korean_market/01_government_api.md)**
  - 공공데이터포털 연동
  - 기상청 API 활용
  - 지역별 맞춤 정보

- **[02. 법적 요구사항](L5_korean_market/02_legal_requirements.md)**
  - 개인정보보호법 대응
  - 위치정보보호법 준수
  - 청소년보호 정책

**결과물**: 한국 시장에 완전 최적화된 앱

---

### [L6: 고급 최적화](L6_advanced/) (선택사항)
**목표**: 대규모 사용자를 위한 최적화

- **[00. 성능 최적화](L6_advanced/00_performance.md)**
- **[01. 메모리 관리](L6_advanced/01_memory.md)**
- **[02. 배터리 최적화](L6_advanced/02_battery.md)**

### [L7: 개발 철학](L7_philosophy/) (선택사항)
**목표**: 왜 이렇게 개발하는가에 대한 사고

- **[00. Apple의 설계 철학](L7_philosophy/00_apple_philosophy.md)**
- **[01. 사용자 중심 설계](L7_philosophy/01_user_centered.md)**

---

## 🛠️ 실습 환경

### 필수 요구사항
- **Xcode 16.0+** (2024년 9월 릴리스)
- **iOS 18.0+** 시뮬레이터
- **Swift 6.0+**
- **macOS Sonoma 14.0+**

### 권장사항
- **GitHub 계정** (코드 백업용)
- **TestFlight** 접근 (베타 테스트용)
- **Apple Developer Program** ($99/년, 실제 출시용)

---

## 📊 학습 성과 측정

### L0 완료 후
```swift
// 이 코드를 이해하고 수정할 수 있음
struct ContentView: View {
    @State private var weather: Weather?
    var body: some View { ... }
}
```

### L1 완료 후
- 실제 API를 사용하는 앱을 만들 수 있음
- 시뮬레이터에서 완전히 작동하는 앱 보유
- 기본적인 iOS 개발 워크플로우 이해

### L2 완료 후
- Core Data 기반 앱을 만들 수 있음
- 클라우드 동기화 구현 가능
- 복잡한 상태 관리 능력

### L3 완료 후
- AI/ML 기능 통합 가능
- 소셜 기능 구현 능력
- 고급 iOS 개발자 수준

### L4 완료 후
- App Store에 실제 앱 출시 경험
- 마케팅과 운영 지식
- 풀스택 모바일 개발자

---

## 🎯 이 책의 차별점

### 1. **100% 실행 가능한 코드**
모든 예제가 Xcode 16에서 실제로 컴파일되고 실행됩니다.

### 2. **완성된 앱 3개**
책을 다 읽으면 포트폴리오에 추가할 수 있는 완성된 앱 3개를 갖게 됩니다.

### 3. **App Store 출시 경험**
단순히 코딩만이 아닌, 실제 사용자에게 배포하는 전체 과정을 경험합니다.

### 4. **한국 시장 특화**
국내 결제, API, 법적 요구사항까지 실무에 바로 적용 가능합니다.

### 5. **2025년 최신 기술**
Swift 6, iOS 18, Xcode 16 기준으로 모든 내용이 업데이트되어 있습니다.

---

## 👥 대상 독자

### ✅ 완벽한 대상
- **Swift 기본 문법을 아는 iOS 개발자**
- **실제 앱을 만들어서 출시하고 싶은 사람**
- **포트폴리오용 완성된 앱이 필요한 취업 준비생**
- **회사에서 iOS 앱 개발을 맡게 된 개발자**

### ⚠️ 이 책이 맞지 않는 사람
- Swift 문법을 전혀 모르는 완전 초보
- 철학적 토론을 원하는 사람
- 당장 결과물이 필요하지 않은 사람

---

## 🚀 바로 시작하기

**1분 테스트**: 다음 코드가 이해된다면 바로 시작하세요!

```swift
struct ContentView: View {
    @State private var message = "Hello"
    
    var body: some View {
        VStack {
            Text(message)
            Button("Change") {
                message = "World"
            }
        }
    }
}
```

이해된다면 → **[L0: Hello World → 실제 앱](L0_hello_to_real/00_real_hello.md)**으로!

이해 안 된다면 → Swift 기본서부터 공부 후 돌아오세요.

---

## 📞 지원과 커뮤니티

### 문제 해결
- **GitHub Issues**: 코드 관련 문제
- **Discord 채널**: 실시간 질답
- **YouTube 채널**: 영상 보충 설명

### 성공 사례 공유
완성한 앱을 App Store에 출시하면 알려주세요! 성공 사례로 소개해드립니다.

---

**준비되셨나요? 3개월 후 여러분의 앱이 App Store에 있을 겁니다.**

→ **[L0: Hello World → 실제 앱 시작하기](L0_hello_to_real/00_real_hello.md)**

---

*"Code is poetry, but shipped apps are symphonies."*

© 2025 실전 iOS 개발 가이드 | 모든 코드 예제는 실제 작동합니다.