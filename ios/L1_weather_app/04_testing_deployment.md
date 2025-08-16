# L1-04: 테스팅과 App Store 출시
## 실제 사용자에게 배포하기

---

> **"Real artists ship."**

완벽한 테스트와 최적화를 거쳐 App Store에 출시해봅시다.

---

## 🎯 목표

**완성 후 결과물**:
- 완전한 테스트 커버리지
- 성능 최적화 완료
- App Store 심사 통과
- 실제 사용자 다운로드 가능

---

## 🚀 실습: 완벽한 테스팅

### 1단계: 유닛 테스트

**WeatherNowTests/WeatherServiceTests.swift**:

```swift
import XCTest
@testable import WeatherNow

final class WeatherServiceTests: XCTestCase {
    var weatherService: WeatherService!
    var mockHTTPClient: MockHTTPClient!
    
    override func setUp() {
        super.setUp()
        weatherService = WeatherService()
        mockHTTPClient = MockHTTPClient()
    }
    
    override func tearDown() {
        weatherService = nil
        mockHTTPClient = nil
        super.tearDown()
    }
    
    // MARK: - Weather Data Tests
    
    func testWeatherDataParsing() throws {
        // Given
        let jsonData = """
        {
            "coord": {"lon": 126.9780, "lat": 37.5665},
            "weather": [{
                "id": 800,
                "main": "Clear",
                "description": "clear sky",
                "icon": "01d"
            }],
            "main": {
                "temp": 23.5,
                "feels_like": 24.0,
                "temp_min": 20.0,
                "temp_max": 26.0,
                "pressure": 1013,
                "humidity": 65
            },
            "visibility": 10000,
            "wind": {"speed": 2.5, "deg": 180},
            "clouds": {"all": 0},
            "dt": 1609459200,
            "sys": {
                "country": "KR",
                "sunrise": 1609459200,
                "sunset": 1609498800
            },
            "timezone": 32400,
            "id": 1835848,
            "name": "Seoul",
            "cod": 200
        }
        """.data(using: .utf8)!
        
        // When
        let response = try JSONDecoder().decode(WeatherResponse.self, from: jsonData)
        let weatherData = WeatherData(from: response)
        
        // Then
        XCTAssertEqual(weatherData.cityName, "Seoul")
        XCTAssertEqual(weatherData.country, "KR")
        XCTAssertEqual(weatherData.temperature, 24)
        XCTAssertEqual(weatherData.condition, "Clear")
        XCTAssertEqual(weatherData.humidity, 65)
    }
    
    func testWeatherFetchSuccess() async throws {
        // Given
        mockHTTPClient.mockResponse = WeatherResponse.mock
        
        // When
        await weatherService.fetchCurrentWeather(for: "Seoul")
        
        // Then
        XCTAssertNotNil(weatherService.currentWeather)
        XCTAssertEqual(weatherService.currentWeather?.cityName, "Seoul")
        XCTAssertFalse(weatherService.isLoading)
        XCTAssertNil(weatherService.errorMessage)
    }
    
    func testWeatherFetchFailure() async {
        // Given
        mockHTTPClient.shouldFail = true
        mockHTTPClient.error = NetworkError.notFound
        
        // When
        await weatherService.fetchCurrentWeather(for: "InvalidCity")
        
        // Then
        XCTAssertNil(weatherService.currentWeather)
        XCTAssertNotNil(weatherService.errorMessage)
        XCTAssertFalse(weatherService.isLoading)
    }
    
    func testCacheValidation() {
        // Given
        let weather = WeatherData.sample
        let cacheManager = WeatherCacheManager.shared
        
        // When
        cacheManager.cacheWeather(weather)
        let cachedWeather = cacheManager.getCachedWeather(for: weather.cityName)
        
        // Then
        XCTAssertNotNil(cachedWeather)
        XCTAssertEqual(cachedWeather?.cityName, weather.cityName)
        XCTAssertEqual(cachedWeather?.temperature, weather.temperature)
    }
    
    func testLocationBasedWeather() async throws {
        // Given
        let latitude = 37.5665
        let longitude = 126.9780
        mockHTTPClient.mockResponse = WeatherResponse.mock
        
        // When
        await weatherService.fetchCurrentWeather(latitude: latitude, longitude: longitude)
        
        // Then
        XCTAssertNotNil(weatherService.currentWeather)
        XCTAssertEqual(weatherService.currentWeather?.coordinates.latitude, latitude)
        XCTAssertEqual(weatherService.currentWeather?.coordinates.longitude, longitude)
    }
    
    // MARK: - Network Tests
    
    func testNetworkErrorHandling() async {
        // Given
        let testCases: [(NetworkError, String)] = [
            (.invalidURL, "잘못된 URL입니다."),
            (.unauthorized, "API 키를 확인해주세요."),
            (.notFound, "요청한 정보를 찾을 수 없습니다."),
            (.tooManyRequests, "너무 많은 요청입니다. 잠시 후 다시 시도해주세요."),
            (.serverError, "서버에 문제가 발생했습니다.")
        ]
        
        for (error, expectedMessage) in testCases {
            // When
            mockHTTPClient.error = error
            mockHTTPClient.shouldFail = true
            await weatherService.fetchCurrentWeather(for: "Seoul")
            
            // Then
            XCTAssertEqual(weatherService.errorMessage, expectedMessage)
        }
    }
    
    // MARK: - Performance Tests
    
    func testWeatherFetchPerformance() {
        measure {
            let expectation = XCTestExpectation(description: "Weather fetch")
            
            Task {
                await weatherService.fetchCurrentWeather(for: "Seoul")
                expectation.fulfill()
            }
            
            wait(for: [expectation], timeout: 5.0)
        }
    }
}

// MARK: - Mock Objects

class MockHTTPClient: HTTPClient {
    var mockResponse: Codable?
    var shouldFail = false
    var error: NetworkError?
    var requestCount = 0
    
    override func fetch<T>(_ type: T.Type, from url: URL) async throws -> T where T : Codable {
        requestCount += 1
        
        if shouldFail {
            throw error ?? NetworkError.networkError(NSError(domain: "test", code: -1))
        }
        
        guard let response = mockResponse as? T else {
            throw NetworkError.decodingFailed
        }
        
        return response
    }
}

extension WeatherResponse {
    static var mock: WeatherResponse {
        WeatherResponse(
            coord: Coordinates(lon: 126.9780, lat: 37.5665),
            weather: [WeatherCondition(id: 800, main: "Clear", description: "clear sky", icon: "01d")],
            base: "stations",
            main: MainWeatherData(
                temp: 23.5,
                feelsLike: 24.0,
                tempMin: 20.0,
                tempMax: 26.0,
                pressure: 1013,
                humidity: 65,
                seaLevel: nil,
                grndLevel: nil
            ),
            visibility: 10000,
            wind: WindData(speed: 2.5, deg: 180, gust: nil),
            clouds: CloudData(all: 0),
            dt: 1609459200,
            sys: SystemData(type: 1, id: 8105, country: "KR", sunrise: 1609459200, sunset: 1609498800),
            timezone: 32400,
            id: 1835848,
            name: "Seoul",
            cod: 200
        )
    }
}
```

### 2단계: UI 테스트

**WeatherNowUITests/WeatherNowUITests.swift**:

```swift
import XCTest

final class WeatherNowUITests: XCTestCase {
    var app: XCUIApplication!
    
    override func setUpWithError() throws {
        continueAfterFailure = false
        app = XCUIApplication()
        app.launchArguments = ["UI_TESTING"]
        app.launch()
    }
    
    override func tearDownWithError() throws {
        app = nil
    }
    
    // MARK: - Onboarding Tests
    
    func testOnboardingFlow() {
        // 온보딩 화면 확인
        XCTAssertTrue(app.staticTexts["날씨를 한눈에"].exists)
        
        // 다음 버튼 탭
        app.buttons["다음"].tap()
        XCTAssertTrue(app.staticTexts["위치 기반 알림"].exists)
        
        // 다음 버튼 탭
        app.buttons["다음"].tap()
        XCTAssertTrue(app.staticTexts["위젯으로 더 빠르게"].exists)
        
        // 시작하기 버튼 탭
        app.buttons["시작하기"].tap()
        
        // 메인 화면 도달 확인
        XCTAssertTrue(app.navigationBars["WeatherNow"].exists)
    }
    
    // MARK: - Main Screen Tests
    
    func testSearchCity() {
        // 검색 필드 찾기
        let searchField = app.textFields["도시 검색..."]
        XCTAssertTrue(searchField.exists)
        
        // 도시명 입력
        searchField.tap()
        searchField.typeText("Seoul")
        
        // 키보드 닫기
        app.keyboards.buttons["return"].tap()
        
        // 결과 확인 (약간의 지연 필요)
        let seoulText = app.staticTexts["서울"]
        XCTAssertTrue(seoulText.waitForExistence(timeout: 5))
    }
    
    func testNavigationTabs() {
        // 날씨 탭
        XCTAssertTrue(app.tabBars.buttons["날씨"].isSelected)
        
        // 지도 탭
        app.tabBars.buttons["지도"].tap()
        XCTAssertTrue(app.tabBars.buttons["지도"].isSelected)
        
        // 저장 탭
        app.tabBars.buttons["저장"].tap()
        XCTAssertTrue(app.tabBars.buttons["저장"].isSelected)
        
        // 설정 탭
        app.tabBars.buttons["설정"].tap()
        XCTAssertTrue(app.tabBars.buttons["설정"].isSelected)
    }
    
    func testPullToRefresh() {
        // 날씨 화면에서
        let weatherView = app.scrollViews.firstMatch
        
        // 아래로 스와이프 (pull to refresh)
        weatherView.swipeDown()
        
        // 로딩 인디케이터 확인
        let progressView = app.activityIndicators.firstMatch
        XCTAssertTrue(progressView.exists)
    }
    
    // MARK: - Settings Tests
    
    func testSettingsNavigation() {
        // 설정 탭으로 이동
        app.tabBars.buttons["설정"].tap()
        
        // 설정 항목들 확인
        XCTAssertTrue(app.staticTexts["단위 설정"].exists)
        XCTAssertTrue(app.staticTexts["알림"].exists)
        XCTAssertTrue(app.staticTexts["외관"].exists)
        XCTAssertTrue(app.staticTexts["데이터"].exists)
        XCTAssertTrue(app.staticTexts["정보"].exists)
    }
    
    func testTemperatureUnitChange() {
        // 설정으로 이동
        app.tabBars.buttons["설정"].tap()
        
        // 온도 단위 선택
        app.cells["온도"].tap()
        
        // 화씨 선택
        app.buttons["화씨 (°F)"].tap()
        
        // 날씨 탭으로 돌아가기
        app.tabBars.buttons["날씨"].tap()
        
        // 온도 표시 확인 (°F 포함)
        let fahrenheitTemp = app.staticTexts.matching(NSPredicate(format: "label CONTAINS '°F'")).firstMatch
        XCTAssertTrue(fahrenheitTemp.exists)
    }
    
    // MARK: - Accessibility Tests
    
    func testAccessibilityLabels() {
        // VoiceOver 라벨 확인
        XCTAssertTrue(app.buttons["날씨"].isAccessibilityElement)
        XCTAssertEqual(app.buttons["날씨"].label, "날씨")
        
        // 날씨 정보 접근성
        let weatherCard = app.otherElements["현재 날씨 카드"]
        XCTAssertTrue(weatherCard.isAccessibilityElement)
    }
    
    // MARK: - Performance Tests
    
    func testLaunchPerformance() throws {
        if #available(iOS 13.0, *) {
            measure(metrics: [XCTApplicationLaunchMetric()]) {
                XCUIApplication().launch()
            }
        }
    }
    
    func testScrollPerformance() {
        measure {
            let scrollView = app.scrollViews.firstMatch
            scrollView.swipeUp()
            scrollView.swipeDown()
        }
    }
}

// MARK: - Test Helpers

extension XCUIElement {
    func clearAndTypeText(_ text: String) {
        guard let stringValue = self.value as? String else {
            XCTFail("Tried to clear and type text into a non string value")
            return
        }
        
        self.tap()
        let deleteString = String(repeating: XCUIKeyboardKey.delete.rawValue, count: stringValue.count)
        self.typeText(deleteString)
        self.typeText(text)
    }
}
```

### 3단계: 성능 최적화

**Performance/PerformanceTests.swift**:

```swift
import XCTest
@testable import WeatherNow

final class PerformanceTests: XCTestCase {
    
    func testMemoryUsage() {
        let info = ProcessInfo.processInfo
        let physicalMemory = info.physicalMemory
        let memoryUsage = getMemoryUsage()
        
        // 메모리 사용량이 100MB 이하인지 확인
        XCTAssertLessThan(memoryUsage, 100 * 1024 * 1024)
    }
    
    func testCPUUsage() {
        let cpuUsage = getCPUUsage()
        
        // CPU 사용률이 50% 이하인지 확인
        XCTAssertLessThan(cpuUsage, 50.0)
    }
    
    func testNetworkRequestTime() {
        measure {
            let expectation = XCTestExpectation(description: "Network request")
            
            Task {
                let start = CFAbsoluteTimeGetCurrent()
                await WeatherService().fetchCurrentWeather(for: "Seoul")
                let end = CFAbsoluteTimeGetCurrent()
                
                // 2초 이내 응답 확인
                XCTAssertLessThan(end - start, 2.0)
                expectation.fulfill()
            }
            
            wait(for: [expectation], timeout: 5.0)
        }
    }
    
    func testImageLoadingPerformance() {
        measure {
            // 이미지 캐싱 성능 테스트
            let imageCache = ImageCache.shared
            
            for i in 0..<100 {
                let imageName = "weather_icon_\(i % 10)"
                _ = imageCache.getImage(named: imageName)
            }
        }
    }
    
    private func getMemoryUsage() -> Int64 {
        var info = mach_task_basic_info()
        var count = mach_msg_type_number_t(MemoryLayout<mach_task_basic_info>.size) / 4
        
        let result = withUnsafeMutablePointer(to: &info) {
            $0.withMemoryRebound(to: integer_t.self, capacity: 1) {
                task_info(mach_task_self_,
                         task_flavor_t(MACH_TASK_BASIC_INFO),
                         $0,
                         &count)
            }
        }
        
        return result == KERN_SUCCESS ? Int64(info.resident_size) : 0
    }
    
    private func getCPUUsage() -> Double {
        var info = mach_task_basic_info()
        var count = mach_msg_type_number_t(MemoryLayout<mach_task_basic_info>.size) / 4
        
        let result = withUnsafeMutablePointer(to: &info) {
            $0.withMemoryRebound(to: integer_t.self, capacity: 1) {
                task_info(mach_task_self_,
                         task_flavor_t(MACH_TASK_BASIC_INFO),
                         $0,
                         &count)
            }
        }
        
        if result == KERN_SUCCESS {
            let userTime = info.user_time.seconds + Double(info.user_time.microseconds) / 1_000_000
            let systemTime = info.system_time.seconds + Double(info.system_time.microseconds) / 1_000_000
            return (userTime + systemTime) * 100
        }
        
        return 0
    }
}
```

### 4단계: App Store 출시 준비

**AppStore/AppStoreAssets.md**:

```markdown
# App Store 출시 체크리스트

## 1. 앱 정보

### 앱 이름
- 한국어: WeatherNow - 실시간 날씨
- 영어: WeatherNow - Live Weather

### 부제목
- 한국어: 정확한 날씨 예보와 알림
- 영어: Accurate Forecast & Alerts

### 카테고리
- Primary: 날씨
- Secondary: 라이프스타일

## 2. 앱 설명

### 한국어 설명
```
WeatherNow는 가장 정확하고 아름다운 날씨 앱입니다.

주요 기능:
• 실시간 날씨 정보와 상세한 예보
• 시간별, 일별, 주간 날씨 예보
• 아름다운 위젯으로 홈 화면에서 바로 확인
• Live Activities로 Dynamic Island 지원
• 날씨 변화 실시간 알림
• 여러 도시 저장 및 관리
• 다크 모드 완벽 지원
• 한국 주요 도시 최적화

특별한 기능:
• 미세먼지 및 공기질 정보
• 일출/일몰 시간
• 체감 온도 및 습도
• 바람 속도 및 방향
• 기압 변화 추적
• 가시거리 정보

프리미엄 기능:
• 광고 제거
• 무제한 도시 저장
• 레이더 및 위성 지도
• 상세한 기상 차트
• 1시간 단위 알림 설정

WeatherNow와 함께 완벽한 하루를 계획하세요!
```

### 영어 설명
```
WeatherNow is the most accurate and beautiful weather app.

Key Features:
• Real-time weather information and detailed forecasts
• Hourly, daily, and weekly weather forecasts
• Beautiful widgets for your home screen
• Live Activities with Dynamic Island support
• Real-time weather change notifications
• Save and manage multiple cities
• Full dark mode support
• Optimized for major cities

Special Features:
• Air quality information
• Sunrise/sunset times
• Feels-like temperature and humidity
• Wind speed and direction
• Pressure tracking
• Visibility information

Premium Features:
• Ad-free experience
• Unlimited city saves
• Radar and satellite maps
• Detailed weather charts
• Hourly notification settings

Plan your perfect day with WeatherNow!
```

## 3. 키워드

### 한국어
날씨, 일기예보, 날씨예보, 기상청, 미세먼지, 날씨위젯, 실시간날씨, 주간날씨, 오늘날씨, 내일날씨

### 영어
weather, forecast, weather widget, live weather, weather radar, weather app, local weather, weather alerts, weather map, temperature

## 4. 스크린샷 (필수)

### iPhone 6.7" (1290 x 2796)
1. 메인 화면 - 현재 날씨
2. 시간별 예보
3. 주간 예보
4. 날씨 지도
5. 위젯 갤러리
6. 설정 화면

### iPhone 5.5" (1242 x 2208)
동일한 구성

### iPad Pro 12.9" (2048 x 2732)
1. 메인 대시보드
2. 분할 화면
3. 상세 차트
4. 멀티 도시 뷰

## 5. 앱 아이콘

### 필수 크기
- 1024 x 1024 (App Store)

### 디자인 가이드라인
- 그라데이션 배경 (파랑 → 하늘색)
- 중앙에 태양과 구름 아이콘
- 모서리 둥글게 처리 안 함 (시스템이 자동 처리)

## 6. 연령 등급

### 등급: 4+
- 폭력적 콘텐츠: 없음
- 성적 콘텐츠: 없음
- 공포 테마: 없음
- 의료 정보: 없음
- 도박: 없음

## 7. 가격 및 판매

### 가격
- 기본: 무료
- 인앱 구매:
  - 프리미엄 월간: ₩3,300
  - 프리미엄 연간: ₩33,000
  - 평생 이용권: ₩66,000

### 판매 지역
- 전 세계 (155개국)
- 주요 타겟: 한국, 미국, 일본, 중국

## 8. 앱 심사 정보

### 심사 노트
```
테스트 계정은 필요하지 않습니다.
위치 권한은 현재 위치의 날씨를 표시하기 위해 사용됩니다.
알림 권한은 날씨 변화를 알리기 위해 사용됩니다.
모든 데이터는 OpenWeatherMap API를 통해 제공됩니다.
```

### 연락처
- 이메일: support@weathernow.app
- 전화: +82-10-1234-5678

## 9. 출시 전 체크리스트

- [ ] 모든 크래시 수정
- [ ] 성능 최적화 완료
- [ ] 접근성 테스트 통과
- [ ] 모든 텍스트 현지화
- [ ] 개인정보 처리방침 URL 준비
- [ ] 이용약관 URL 준비
- [ ] 지원 URL 준비
- [ ] 마케팅 URL 준비

## 10. 버전 히스토리

### v1.0.0 (최초 출시)
- 실시간 날씨 정보
- 위젯 지원
- Live Activities
- 다크 모드

### v1.1.0 (계획)
- Apple Watch 앱
- Siri 단축어
- 날씨 공유 기능
```

### 5단계: 자동화된 배포

**fastlane/Fastfile**:

```ruby
default_platform(:ios)

platform :ios do
  
  # 개발 빌드
  desc "Development build"
  lane :dev do
    build_app(
      scheme: "WeatherNow",
      configuration: "Debug",
      export_method: "development"
    )
  end
  
  # 테스트 실행
  desc "Run all tests"
  lane :test do
    run_tests(
      scheme: "WeatherNow",
      devices: ["iPhone 15 Pro"],
      code_coverage: true
    )
  end
  
  # TestFlight 배포
  desc "Deploy to TestFlight"
  lane :beta do
    # 버전 번호 증가
    increment_build_number(
      build_number: latest_testflight_build_number + 1
    )
    
    # 빌드
    build_app(
      scheme: "WeatherNow",
      configuration: "Release",
      export_method: "app-store"
    )
    
    # TestFlight 업로드
    upload_to_testflight(
      skip_waiting_for_build_processing: true
    )
    
    # Slack 알림
    slack(
      message: "새 버전이 TestFlight에 업로드되었습니다! 🚀"
    )
  end
  
  # App Store 배포
  desc "Deploy to App Store"
  lane :release do
    # 스크린샷 생성
    capture_screenshots
    
    # 빌드
    build_app(
      scheme: "WeatherNow",
      configuration: "Release",
      export_method: "app-store"
    )
    
    # App Store Connect 업로드
    upload_to_app_store(
      skip_metadata: false,
      skip_screenshots: false,
      submit_for_review: true,
      automatic_release: false,
      force: true,
      
      # 메타데이터
      app_identifier: "com.yourname.weathernow",
      name: {
        "ko" => "WeatherNow - 실시간 날씨",
        "en-US" => "WeatherNow - Live Weather"
      },
      subtitle: {
        "ko" => "정확한 날씨 예보와 알림",
        "en-US" => "Accurate Forecast & Alerts"
      },
      description: {
        "ko" => File.read("./metadata/ko/description.txt"),
        "en-US" => File.read("./metadata/en-US/description.txt")
      },
      keywords: {
        "ko" => "날씨,일기예보,날씨예보,기상청,미세먼지",
        "en-US" => "weather,forecast,weather widget,live weather"
      },
      
      # 앱 정보
      primary_category: "WEATHER",
      secondary_category: "LIFESTYLE",
      
      # 가격
      price_tier: 0,
      
      # 심사 정보
      app_review_information: {
        first_name: "John",
        last_name: "Smith",
        phone_number: "+821012345678",
        email_address: "review@weathernow.app",
        demo_user: "",
        demo_password: "",
        notes: "No login required. Location permission is used for current weather."
      }
    )
    
    # 성공 알림
    slack(
      message: "WeatherNow가 App Store에 제출되었습니다! 🎉"
    )
  end
  
  # 크래시 리포트 확인
  desc "Check crash reports"
  lane :crashes do
    download_dsyms
    upload_symbols_to_crashlytics
  end
  
end
```

---

## 🎯 여기서 배운 것

### 1. **완벽한 테스팅**
- 유닛 테스트
- UI 테스트
- 성능 테스트
- 접근성 테스트

### 2. **App Store 준비**
- 메타데이터 작성
- 스크린샷 준비
- 심사 대응
- 현지화

### 3. **자동화 배포**
- Fastlane 설정
- CI/CD 파이프라인
- 버전 관리
- 크래시 모니터링

---

## 🎉 최종 체크리스트

### 출시 전 필수 확인

**기능 테스트**:
- [ ] 모든 API 정상 작동
- [ ] 오프라인 모드 테스트
- [ ] 위젯 업데이트 확인
- [ ] 알림 수신 테스트
- [ ] 다크모드 완벽 지원

**성능 최적화**:
- [ ] 앱 크기 < 50MB
- [ ] 콜드 스타트 < 2초
- [ ] 메모리 사용 < 100MB
- [ ] 배터리 효율 최적화

**App Store 준비**:
- [ ] 1024x1024 앱 아이콘
- [ ] 모든 크기 스크린샷
- [ ] 설명 2개 언어 이상
- [ ] 개인정보 처리방침
- [ ] 지원 URL

**심사 대비**:
- [ ] 가이드라인 준수
- [ ] 권한 설명 명확
- [ ] 크래시 없음
- [ ] 콘텐츠 적절성

---

## 🚀 출시 후 관리

### 모니터링
```swift
// Analytics 설정
Analytics.logEvent("app_launched", parameters: [
    "version": AppVersion.current,
    "device": UIDevice.current.model
])

// 크래시 리포팅
Crashlytics.crashlytics().log("User opened weather detail")

// 성능 모니터링
let trace = Performance.startTrace(name: "weather_fetch")
// ... 작업 수행
trace?.stop()
```

### 사용자 피드백 대응
1. App Store 리뷰 모니터링
2. 크래시 리포트 분석
3. 사용자 문의 대응
4. 정기 업데이트 계획

---

**축하합니다! 🎉 이제 완벽한 프로덕션 레벨의 날씨 앱이 App Store에 출시되었습니다!**

**다음 단계**:
- A/B 테스팅 도입
- 사용자 분석 강화
- 새 기능 추가
- 글로벌 확장