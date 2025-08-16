# 모듈러 아키텍처와 SPM: 확장 가능한 코드베이스

## Core Insight
모듈러 아키텍처는 거대한 모놀리식 앱을 독립적인 기능 모듈로 분해하여 개발 속도와 코드 품질을 동시에 향상시킨다.

Swift Package Manager(SPM)의 등장으로 iOS 앱의 모듈화가 혁명적으로 단순해졌다. 예전에는 복잡한 워크스페이스 설정과 프레임워크 관리가 필요했지만, 이제는 Package.swift 파일 하나로 모든 것이 해결된다.

**모듈러 아키텍처의 핵심 원칙:**

1. **기능별 분리 (Feature Modules)**
```
MyApp/
├── Packages/
│   ├── UserFeature/
│   │   ├── Sources/
│   │   │   ├── UserFeature/
│   │   │   │   ├── UserView.swift
│   │   │   │   ├── UserViewModel.swift
│   │   │   │   └── UserService.swift
│   │   │   └── UserFeatureInterface/
│   │   │       └── UserServiceProtocol.swift
│   │   └── Package.swift
│   ├── SettingsFeature/
│   └── NetworkModule/
```

2. **계층별 분리 (Layered Architecture)**
```swift
// Package.swift 예시
let package = Package(
    name: "MyAppCore",
    products: [
        .library(name: "NetworkLayer", targets: ["NetworkLayer"]),
        .library(name: "DataLayer", targets: ["DataLayer"]),
        .library(name: "DomainLayer", targets: ["DomainLayer"]),
        .library(name: "PresentationLayer", targets: ["PresentationLayer"])
    ],
    dependencies: [
        .package(url: "https://github.com/Alamofire/Alamofire.git", from: "5.0.0")
    ],
    targets: [
        .target(
            name: "NetworkLayer",
            dependencies: ["Alamofire"]
        ),
        .target(
            name: "DataLayer",
            dependencies: ["NetworkLayer"]
        ),
        .target(
            name: "DomainLayer",
            dependencies: ["DataLayer"]
        ),
        .target(
            name: "PresentationLayer",
            dependencies: ["DomainLayer"]
        )
    ]
)
```

**인터페이스와 구현 분리:**
```swift
// UserFeatureInterface (프로토콜만)
public protocol UserServiceProtocol {
    func loadUsers() async throws -> [User]
}

// UserFeature (구현)
import UserFeatureInterface

public struct UserService: UserServiceProtocol {
    public func loadUsers() async throws -> [User] {
        // 실제 구현
    }
}
```

**Feature Flag를 통한 모듈 활성화:**
```swift
// AppConfiguration
struct FeatureFlags {
    static let newUserProfileEnabled = true
    static let betaSettingsEnabled = false
}

// App.swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            if FeatureFlags.newUserProfileEnabled {
                NewUserProfileView()
            } else {
                LegacyUserProfileView()
            }
        }
    }
}
```

**모듈 간 통신 패턴:**

1. **이벤트 버스 (Event Bus)**
```swift
public protocol EventBus {
    func publish<T>(_ event: T)
    func subscribe<T>(_ type: T.Type, handler: @escaping (T) -> Void)
}

// 사용법
eventBus.publish(UserLoggedInEvent(userId: "123"))
```

2. **델리게이트 패턴**
```swift
public protocol UserFeatureDelegate: AnyObject {
    func userDidLogin(_ user: User)
    func userDidLogout()
}
```

3. **Coordinator 패턴**
```swift
public protocol Coordinator {
    func start()
    func coordinate(to destination: Destination)
}

public class UserCoordinator: Coordinator {
    public func coordinate(to destination: Destination) {
        switch destination {
        case .settings:
            settingsCoordinator.start()
        case .profile:
            profileCoordinator.start()
        }
    }
}
```

**모듈 테스트 전략:**
```swift
// UserFeatureTests Package.swift
.testTarget(
    name: "UserFeatureTests",
    dependencies: [
        "UserFeature",
        "UserFeatureInterface",
        "TestHelpers"  // 공통 테스트 유틸리티
    ]
)
```

**빌드 성능 최적화:**
SPM은 모듈을 병렬로 컴파일하므로 빌드 시간이 크게 단축된다. 또한 변경된 모듈만 재컴파일하므로 증분 빌드가 매우 빨라진다.

**의존성 그래프 관리:**
```swift
// DependencyGraph.swift
public struct AppDependencies {
    public let userService: UserServiceProtocol
    public let settingsService: SettingsServiceProtocol
    
    public init() {
        self.userService = UserService()
        self.settingsService = SettingsService()
    }
}
```

**모듈화의 이점:**
- 컴파일 시간 단축 (병렬 빌드)
- 코드 재사용성 증가
- 팀 간 작업 분리 가능
- 테스트 분리로 더 빠른 피드백
- 기능별 릴리스 가능

**주의사항:**
- 과도한 모듈화는 복잡성 증가
- 인터페이스 설계가 중요
- 순환 의존성 방지 필수
- 버전 관리 전략 필요

모듈러 아키텍처는 작은 프로젝트에서는 오버엔지니어링일 수 있지만, 팀이 커지고 기능이 복잡해질수록 그 가치가 빛을 발한다. SPM의 등장으로 이 패턴을 적용하는 비용이 크게 줄어들었다.

## Connections
→ [[293-testing-strategies-ios]]
→ [[294-xcode-productivity-techniques]]
← [[291-dependency-injection-strategies]]
← [[240-swift-package-manager-ecosystem]]

---
Level: L5
Date: 2025-08-16
Tags: #modular-architecture #spm #swift-package-manager #scalability #team-development