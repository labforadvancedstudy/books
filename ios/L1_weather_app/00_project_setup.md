# L1-00: 프로덕션 수준 프로젝트 설정
## App Store 출시 가능한 앱의 시작점

---

> **"The difference between amateur and professional is in the details you can't see."**

L0에서 기본 앱을 만들었다면, 이제 실제 사용자가 다운받을 수 있는 프로덕션 수준의 앱을 만들어봅시다.

---

## 🎯 목표

**1시간 후 결과물**:
- 완전한 Xcode 프로젝트 구조
- App Store 출시 준비된 설정
- 프로덕션 수준 아키텍처

**배울 것**:
- 프로덕션 프로젝트 구조 설계
- Bundle ID와 팀 설정
- Info.plist 최적화
- 아키텍처 패턴 적용

---

## 📋 사전 준비

### 필수 도구 확인
```bash
# Xcode 버전 확인
xcodebuild -version
# 출력: Xcode 16.0 이상

# Swift 버전 확인  
swift --version
# 출력: Swift 6.0 이상

# iOS Simulator 확인
xcrun simctl list devices available
```

### Apple Developer 계정 (선택사항)
- **무료 계정**: 시뮬레이터 테스트 가능
- **유료 계정 ($99/년)**: 실제 기기 테스트 및 App Store 출시

---

## 🚀 실습: 프로덕션 프로젝트 생성

### 1단계: 새 프로젝트 생성

**Xcode → File → New → Project**

```
Platform: iOS
Template: App
Product Name: WeatherNow
Team: (본인 계정 선택)
Organization Identifier: com.yourname.weathernow
Bundle Identifier: com.yourname.weathernow.app
Language: Swift
Interface: SwiftUI
Use Core Data: ✅ (체크)
Include Tests: ✅ (체크)
```

⚠️ **중요**: 
- Bundle Identifier는 전 세계에서 유일해야 함
- 나중에 변경하기 어려우므로 신중히 선택
- 예: `com.yourname.weathernow` → `com.johnsmith.weathernow`

### 2단계: 프로젝트 구조 설계

생성된 프로젝트에 다음 폴더 구조를 만들어봅시다:

```
WeatherNow/
├── App/
│   ├── WeatherNowApp.swift
│   └── ContentView.swift
├── Features/
│   ├── Weather/
│   │   ├── Models/
│   │   ├── Views/
│   │   ├── ViewModels/
│   │   └── Services/
│   ├── Location/
│   └── Settings/
├── Core/
│   ├── Network/
│   ├── Storage/
│   ├── Extensions/
│   └── Utils/
├── Resources/
│   ├── Assets.xcassets
│   ├── Localizable.strings
│   └── Info.plist
└── Tests/
    ├── WeatherNowTests/
    └── WeatherNowUITests/
```

**폴더 생성하기**:
1. 프로젝트 네비게이터에서 `WeatherNow` 우클릭
2. **New Group** 선택
3. 위 구조대로 폴더 생성

### 3단계: Info.plist 최적화

`Info.plist`에 다음 설정 추가:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <!-- 기본 앱 정보 -->
    <key>CFBundleDisplayName</key>
    <string>날씨Now</string>
    <key>CFBundleName</key>
    <string>WeatherNow</string>
    <key>CFBundleShortVersionString</key>
    <string>1.0</string>
    <key>CFBundleVersion</key>
    <string>1</string>
    
    <!-- 최소 iOS 버전 -->
    <key>LSMinimumSystemVersion</key>
    <string>17.0</string>
    
    <!-- 권한 설명 -->
    <key>NSLocationWhenInUseUsageDescription</key>
    <string>현재 위치의 날씨 정보를 제공하기 위해 위치 권한이 필요합니다.</string>
    <key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
    <string>백그라운드에서도 날씨 업데이트를 받기 위해 위치 권한이 필요합니다.</string>
    
    <!-- 네트워크 보안 -->
    <key>NSAppTransportSecurity</key>
    <dict>
        <key>NSAllowsArbitraryLoads</key>
        <false/>
        <key>NSExceptionDomains</key>
        <dict>
            <key>api.openweathermap.org</key>
            <dict>
                <key>NSExceptionRequiresForwardSecrecy</key>
                <false/>
                <key>NSExceptionMinimumTLSVersion</key>
                <string>TLSv1.2</string>
            </dict>
        </dict>
    </dict>
    
    <!-- 백그라운드 모드 -->
    <key>UIBackgroundModes</key>
    <array>
        <string>background-fetch</string>
        <string>remote-notification</string>
    </array>
    
    <!-- 지원하는 인터페이스 방향 -->
    <key>UISupportedInterfaceOrientations</key>
    <array>
        <string>UIInterfaceOrientationPortrait</string>
        <string>UIInterfaceOrientationPortraitUpsideDown</string>
    </array>
    
    <!-- 상태바 스타일 -->
    <key>UIStatusBarStyle</key>
    <string>UIStatusBarStyleDefault</string>
    <key>UIViewControllerBasedStatusBarAppearance</key>
    <false/>
    
    <!-- 론치 스크린 -->
    <key>UILaunchScreen</key>
    <dict>
        <key>UIColorName</key>
        <string>AccentColor</string>
        <key>UIImageName</key>
        <string>LaunchIcon</string>
    </dict>
</dict>
</plist>
```

### 4단계: 앱 아이콘 및 리소스 설정

**Assets.xcassets 최적화**:

1. **AppIcon 추가**:
   - `Assets.xcassets` → `AppIcon` 클릭
   - 1024x1024 PNG 이미지 드래그 앤 드롭
   - Xcode가 자동으로 모든 크기 생성

2. **Color Set 추가**:
```swift
// Assets.xcassets에서 New Color Set 생성
Primary Color: #007AFF (iOS Blue)
Secondary Color: #8E8E93 (iOS Gray)
Accent Color: #FF9500 (iOS Orange)
Background Color: #F2F2F7 (iOS Light Gray)
```

3. **Launch Icon 추가**:
   - 심플한 날씨 아이콘 (512x512)
   - 브랜드 컬러 사용

### 5단계: Core 아키텍처 구현

**Core/Network/NetworkManager.swift** 생성:

```swift
import Foundation
import Network

// MARK: - Network Manager
@MainActor
class NetworkManager: ObservableObject {
    static let shared = NetworkManager()
    
    @Published var isConnected = true
    @Published var connectionType: NWInterface.InterfaceType?
    
    private let monitor = NWPathMonitor()
    private let queue = DispatchQueue(label: "NetworkMonitor")
    
    private init() {
        startMonitoring()
    }
    
    private func startMonitoring() {
        monitor.pathUpdateHandler = { [weak self] path in
            DispatchQueue.main.async {
                self?.isConnected = path.status == .satisfied
                self?.connectionType = path.availableInterfaces.first?.type
            }
        }
        monitor.start(queue: queue)
    }
    
    deinit {
        monitor.cancel()
    }
}

// MARK: - API Configuration
struct APIConfig {
    static let baseURL = "https://api.openweathermap.org/data/2.5"
    static let apiKey = Bundle.main.object(forInfoDictionaryKey: "OPENWEATHER_API_KEY") as? String ?? ""
    static let units = "metric"
    static let language = "kr"
    
    // API 엔드포인트
    enum Endpoint {
        case currentWeather(city: String)
        case weatherByCoordinates(lat: Double, lon: Double)
        case forecast(city: String)
        
        var url: URL? {
            var components = URLComponents(string: APIConfig.baseURL)
            
            switch self {
            case .currentWeather(let city):
                components?.path += "/weather"
                components?.queryItems = [
                    URLQueryItem(name: "q", value: city),
                    URLQueryItem(name: "appid", value: APIConfig.apiKey),
                    URLQueryItem(name: "units", value: APIConfig.units),
                    URLQueryItem(name: "lang", value: APIConfig.language)
                ]
            case .weatherByCoordinates(let lat, let lon):
                components?.path += "/weather"
                components?.queryItems = [
                    URLQueryItem(name: "lat", value: String(lat)),
                    URLQueryItem(name: "lon", value: String(lon)),
                    URLQueryItem(name: "appid", value: APIConfig.apiKey),
                    URLQueryItem(name: "units", value: APIConfig.units),
                    URLQueryItem(name: "lang", value: APIConfig.language)
                ]
            case .forecast(let city):
                components?.path += "/forecast"
                components?.queryItems = [
                    URLQueryItem(name: "q", value: city),
                    URLQueryItem(name: "appid", value: APIConfig.apiKey),
                    URLQueryItem(name: "units", value: APIConfig.units),
                    URLQueryItem(name: "lang", value: APIConfig.language)
                ]
            }
            
            return components?.url
        }
    }
}

// MARK: - HTTP Client
class HTTPClient {
    static let shared = HTTPClient()
    
    private let session: URLSession
    
    private init() {
        let config = URLSessionConfiguration.default
        config.timeoutIntervalForRequest = 30
        config.timeoutIntervalForResource = 60
        config.waitsForConnectivity = true
        self.session = URLSession(configuration: config)
    }
    
    func fetch<T: Codable>(_ type: T.Type, from url: URL) async throws -> T {
        do {
            let (data, response) = try await session.data(from: url)
            
            guard let httpResponse = response as? HTTPURLResponse else {
                throw NetworkError.invalidResponse
            }
            
            switch httpResponse.statusCode {
            case 200...299:
                break
            case 401:
                throw NetworkError.unauthorized
            case 404:
                throw NetworkError.notFound
            case 429:
                throw NetworkError.tooManyRequests
            case 500...599:
                throw NetworkError.serverError
            default:
                throw NetworkError.requestFailed(httpResponse.statusCode)
            }
            
            let decoder = JSONDecoder()
            decoder.dateDecodingStrategy = .secondsSince1970
            
            return try decoder.decode(type, from: data)
            
        } catch let error as NetworkError {
            throw error
        } catch is DecodingError {
            throw NetworkError.decodingFailed
        } catch {
            throw NetworkError.networkError(error)
        }
    }
}

// MARK: - Network Errors
enum NetworkError: LocalizedError, Equatable {
    case invalidURL
    case invalidResponse
    case unauthorized
    case notFound
    case tooManyRequests
    case serverError
    case requestFailed(Int)
    case decodingFailed
    case networkError(Error)
    
    var errorDescription: String? {
        switch self {
        case .invalidURL:
            return "잘못된 URL입니다."
        case .invalidResponse:
            return "서버 응답이 올바르지 않습니다."
        case .unauthorized:
            return "API 키를 확인해주세요."
        case .notFound:
            return "요청한 정보를 찾을 수 없습니다."
        case .tooManyRequests:
            return "너무 많은 요청입니다. 잠시 후 다시 시도해주세요."
        case .serverError:
            return "서버에 문제가 발생했습니다."
        case .requestFailed(let code):
            return "요청 실패 (코드: \(code))"
        case .decodingFailed:
            return "데이터 처리 중 오류가 발생했습니다."
        case .networkError(let error):
            return "네트워크 오류: \(error.localizedDescription)"
        }
    }
    
    static func == (lhs: NetworkError, rhs: NetworkError) -> Bool {
        switch (lhs, rhs) {
        case (.invalidURL, .invalidURL),
             (.invalidResponse, .invalidResponse),
             (.unauthorized, .unauthorized),
             (.notFound, .notFound),
             (.tooManyRequests, .tooManyRequests),
             (.serverError, .serverError),
             (.decodingFailed, .decodingFailed):
            return true
        case let (.requestFailed(lhsCode), .requestFailed(rhsCode)):
            return lhsCode == rhsCode
        case let (.networkError(lhsError), .networkError(rhsError)):
            return lhsError.localizedDescription == rhsError.localizedDescription
        default:
            return false
        }
    }
}
```

### 6단계: App 진입점 최적화

**App/WeatherNowApp.swift** 수정:

```swift
import SwiftUI
import CoreData

@main
struct WeatherNowApp: App {
    // Core Data 스택
    let persistenceController = PersistenceController.shared
    
    // 네트워크 모니터링
    @StateObject private var networkManager = NetworkManager.shared
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(\.managedObjectContext, persistenceController.container.viewContext)
                .environmentObject(networkManager)
                .onAppear {
                    setupApp()
                }
        }
    }
    
    private func setupApp() {
        // 앱 설정 초기화
        configureAppearance()
        configureNotifications()
    }
    
    private func configureAppearance() {
        // 네비게이션 바 스타일
        let navBarAppearance = UINavigationBarAppearance()
        navBarAppearance.configureWithOpaqueBackground()
        navBarAppearance.backgroundColor = UIColor.systemBackground
        navBarAppearance.shadowColor = UIColor.clear
        
        UINavigationBar.appearance().standardAppearance = navBarAppearance
        UINavigationBar.appearance().scrollEdgeAppearance = navBarAppearance
        
        // 탭 바 스타일
        let tabBarAppearance = UITabBarAppearance()
        tabBarAppearance.configureWithOpaqueBackground()
        UITabBar.appearance().standardAppearance = tabBarAppearance
        UITabBar.appearance().scrollEdgeAppearance = tabBarAppearance
    }
    
    private func configureNotifications() {
        // 노티피케이션 권한 요청
        UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound, .badge]) { granted, error in
            if granted {
                print("노티피케이션 권한 허용됨")
            } else if let error = error {
                print("노티피케이션 권한 오류: \(error)")
            }
        }
    }
}

// MARK: - Core Data Stack
class PersistenceController {
    static let shared = PersistenceController()
    
    let container: NSPersistentContainer
    
    init(inMemory: Bool = false) {
        container = NSPersistentContainer(name: "WeatherNow")
        
        if inMemory {
            container.persistentStoreDescriptions.first?.url = URL(fileURLWithPath: "/dev/null")
        }
        
        container.persistentStoreDescriptions.first?.setOption(true as NSNumber, forKey: NSPersistentHistoryTrackingKey)
        container.persistentStoreDescriptions.first?.setOption(true as NSNumber, forKey: NSPersistentStoreRemoteChangeNotificationPostOptionKey)
        
        container.loadPersistentStores { _, error in
            if let error = error as NSError? {
                fatalError("Core Data 로드 실패: \(error), \(error.userInfo)")
            }
        }
        
        container.viewContext.automaticallyMergesChangesFromParent = true
        container.viewContext.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
    }
    
    func save() {
        let context = container.viewContext
        
        if context.hasChanges {
            do {
                try context.save()
            } catch {
                let nsError = error as NSError
                fatalError("Core Data 저장 실패: \(nsError), \(nsError.userInfo)")
            }
        }
    }
}
```

### 7단계: Build Settings 최적화

**Project → WeatherNow → Build Settings**에서:

```
// Release 최적화
Swift Compiler - Code Generation
  Optimization Level: Optimize for Speed [-O]
  
// 디버그 정보
Apple Clang - Code Generation  
  Debug Information Format: DWARF with dSYM File

// 보안 강화
Other Swift Flags:
  Debug: -DDEBUG
  Release: -DRELEASE

// 최소 배포 타겟
iOS Deployment Target: 17.0

// 아키텍처 설정
Architectures: Standard Architectures (arm64)
Valid Architectures: arm64

// 비트코드 (현재 Xcode에서는 자동 처리)
Enable Bitcode: Yes
```

### 8단계: Scheme 설정

**Product → Scheme → Edit Scheme**:

```
// Run (Debug)
Build Configuration: Debug
Arguments Passed On Launch:
  -com.apple.CoreData.SQLDebug 1
  -com.apple.CoreData.ConcurrencyDebug 1

Environment Variables:
  API_ENV: development
  LOG_LEVEL: debug

// Archive (Release)  
Build Configuration: Release
```

---

## 🎯 여기서 배운 것

### 1. **프로덕션 프로젝트 구조**
```
Features/ - 기능별 모듈화
Core/ - 공통 유틸리티
Resources/ - 리소스 관리
Tests/ - 테스트 코드
```

### 2. **Info.plist 최적화**
- 권한 설명 메시지
- 네트워크 보안 설정  
- 백그라운드 모드
- 론치 스크린 구성

### 3. **아키텍처 패턴**
- MVVM with Services
- Repository Pattern
- Dependency Injection
- Error Handling

### 4. **Core Data 통합**
- Persistent Container
- Context 관리
- 자동 저장 및 병합

---

## 🔍 프로덕션 체크리스트

### 필수 설정 ✅
- [ ] Bundle ID 고유성 확인
- [ ] Info.plist 권한 설명 추가
- [ ] 앱 아이콘 (모든 크기)
- [ ] 론치 스크린
- [ ] 최소 iOS 버전 설정

### 보안 설정 ✅
- [ ] ATS (App Transport Security) 구성
- [ ] API 키 환경 변수 처리
- [ ] 민감한 정보 하드코딩 방지
- [ ] HTTPS 강제 사용

### 성능 최적화 ✅
- [ ] Release 빌드 최적화
- [ ] 이미지 압축
- [ ] 네트워크 캐싱
- [ ] 메모리 관리

### 사용자 경험 ✅
- [ ] 다크 모드 지원
- [ ] 접근성 라벨
- [ ] 로딩 상태 표시
- [ ] 에러 메시지 현지화

---

## 🚨 자주하는 실수와 해결법

### 1. **Bundle ID 충돌**
```bash
# ❌ 이미 사용 중인 Bundle ID
com.apple.weather

# ✅ 고유한 Bundle ID
com.yourname.weathernow.app
```

### 2. **권한 설명 누락**
```xml
<!-- ❌ 설명 없음 - 심사 거부 -->
<key>NSLocationWhenInUseUsageDescription</key>
<string></string>

<!-- ✅ 명확한 설명 -->
<key>NSLocationWhenInUseUsageDescription</key>
<string>현재 위치의 날씨 정보를 제공하기 위해 위치 권한이 필요합니다.</string>
```

### 3. **API 키 보안**
```swift
// ❌ 하드코딩
let apiKey = "abcd1234efgh5678"

// ✅ Info.plist 환경 변수
let apiKey = Bundle.main.object(forInfoDictionaryKey: "OPENWEATHER_API_KEY") as? String ?? ""
```

### 4. **메모리 누수**
```swift
// ❌ 강한 참조 순환
class ViewController {
    let service = WeatherService()
    
    func setup() {
        service.onUpdate = {
            self.updateUI() // 강한 참조
        }
    }
}

// ✅ 약한 참조
service.onUpdate = { [weak self] in
    self?.updateUI()
}
```

---

## 🎯 실습 과제

### 과제 1: 환경별 설정
```swift
// Configuration.swift 생성
enum Environment {
    case development
    case staging  
    case production
    
    static var current: Environment {
        #if DEBUG
        return .development
        #else
        return .production
        #endif
    }
    
    var apiBaseURL: String {
        switch self {
        case .development: return "https://api-dev.openweathermap.org"
        case .staging: return "https://api-staging.openweathermap.org" 
        case .production: return "https://api.openweathermap.org"
        }
    }
}
```

### 과제 2: 로깅 시스템
```swift
// Logger.swift 생성
import os.log

class Logger {
    static let shared = Logger()
    private let osLog = OSLog(subsystem: Bundle.main.bundleIdentifier!, category: "WeatherNow")
    
    func debug(_ message: String) {
        os_log(.debug, log: osLog, "%@", message)
    }
    
    func error(_ message: String) {
        os_log(.error, log: osLog, "%@", message)
    }
}
```

### 과제 3: 앱 버전 관리
```swift
// AppVersion.swift 생성
struct AppVersion {
    static var current: String {
        Bundle.main.object(forInfoDictionaryKey: "CFBundleShortVersionString") as? String ?? "1.0"
    }
    
    static var build: String {
        Bundle.main.object(forInfoDictionaryKey: "CFBundleVersion") as? String ?? "1"
    }
    
    static var fullVersion: String {
        "\(current) (\(build))"
    }
}
```

---

## 🎉 성공 확인

**기본 설정 체크**:
- [ ] 프로젝트가 에러 없이 빌드됨
- [ ] 시뮬레이터에서 실행됨
- [ ] 모든 폴더 구조가 생성됨
- [ ] Core Data 스택이 초기화됨

**프로덕션 준비 체크**:
- [ ] Bundle ID가 고유함
- [ ] 앱 아이콘이 설정됨
- [ ] 권한 설명이 추가됨
- [ ] 네트워크 보안이 구성됨

**아키텍처 체크**:
- [ ] 네트워크 매니저가 작동함
- [ ] API 설정이 올바름
- [ ] 에러 처리가 구현됨
- [ ] 로깅 시스템이 동작함

---

## 💡 프로덕션 팁

### 1. **버전 관리 전략**
```
Major.Minor.Patch
1.0.0 - 초기 릴리스
1.1.0 - 새 기능 추가
1.1.1 - 버그 수정
```

### 2. **빌드 번호 자동화**
```bash
# Run Script Phase 추가
BUILD_NUMBER=$(git rev-list HEAD --count)
/usr/libexec/PlistBuddy -c "Set :CFBundleVersion $BUILD_NUMBER" "$INFOPLIST_FILE"
```

### 3. **코드 사이닝 설정**
```
Automatically manage signing: ✅
Team: (본인 Apple Developer 계정)
Bundle Identifier: 고유 ID 설정
```

---

## 🚀 다음 단계

훌륭합니다! 이제 프로덕션 수준의 프로젝트 기반이 완성되었습니다.

**지금까지 구축한 것**:
- ✅ 확장 가능한 프로젝트 구조
- ✅ 프로덕션 수준 설정
- ✅ 네트워크 및 데이터 아키텍처  
- ✅ 에러 처리 및 로깅

**다음에 할 것**:
→ **[01. 날씨 API 연동](01_weather_api.md)** - 실제 날씨 데이터 통합

---

**축하합니다! 이제 App Store 출시 가능한 프로젝트 기반을 갖추었습니다.**

→ **[다음: 날씨 API 완벽 연동](01_weather_api.md)**

---

*"Quality is not an act, it is a habit." - Aristotle*