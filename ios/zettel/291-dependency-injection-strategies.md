# 의존성 주입 전략: 테스트 가능한 코드의 기초

## Core Insight
의존성 주입은 객체가 자신의 의존성을 생성하지 않고 외부에서 받는 패턴으로, 테스트 가능성과 모듈성의 핵심이다.

의존성 주입(Dependency Injection)은 SOLID 원칙 중 의존성 역전 원칙의 구현체다. 하드코딩된 의존성을 외부에서 주입받는 방식으로 변경하여, 코드의 결합도를 낮추고 테스트 가능성을 높인다.

**iOS에서의 의존성 주입 방식들:**

1. **생성자 주입 (Constructor Injection)**
```swift
class UserService {
    private let networkManager: NetworkManaging
    private let cache: CacheManaging
    
    init(networkManager: NetworkManaging, cache: CacheManaging) {
        self.networkManager = networkManager
        self.cache = cache
    }
}
```

2. **프로퍼티 주입 (Property Injection)**
```swift
class UserViewController: UIViewController {
    var userService: UserServiceProtocol!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        loadUsers()
    }
}
```

3. **메서드 주입 (Method Injection)**
```swift
protocol UserServiceConsumer {
    func configure(with userService: UserServiceProtocol)
}
```

**DI 컨테이너 없이 간단하게:**
```swift
// Environment 패턴 (SwiftUI에서 많이 사용)
struct Dependencies {
    let userService: UserServiceProtocol
    let settingsService: SettingsServiceProtocol
    let analyticsService: AnalyticsServiceProtocol
}

extension Dependencies {
    static let live = Dependencies(
        userService: UserService(),
        settingsService: SettingsService(),
        analyticsService: AnalyticsService()
    )
    
    static let preview = Dependencies(
        userService: MockUserService(),
        settingsService: MockSettingsService(),
        analyticsService: MockAnalyticsService()
    )
}
```

**Factory 패턴과 결합:**
```swift
protocol ServiceFactory {
    func makeUserService() -> UserServiceProtocol
    func makeNetworkManager() -> NetworkManaging
}

class AppServiceFactory: ServiceFactory {
    func makeUserService() -> UserServiceProtocol {
        UserService(
            networkManager: makeNetworkManager(),
            cache: CoreDataCache()
        )
    }
    
    func makeNetworkManager() -> NetworkManaging {
        URLSessionNetworkManager()
    }
}
```

**SwiftUI Environment를 활용한 DI:**
```swift
struct UserServiceKey: EnvironmentKey {
    static let defaultValue: UserServiceProtocol = UserService()
}

extension EnvironmentValues {
    var userService: UserServiceProtocol {
        get { self[UserServiceKey.self] }
        set { self[UserServiceKey.self] = newValue }
    }
}

// 사용법
struct ContentView: View {
    @Environment(\.userService) var userService
    
    var body: some View {
        // ...
    }
}

// 주입
ContentView()
    .environment(\.userService, MockUserService())
```

**테스트에서의 활용:**
```swift
class UserServiceTests: XCTestCase {
    func testUserLoading() async {
        // Given
        let mockNetwork = MockNetworkManager()
        let mockCache = MockCache()
        let userService = UserService(
            networkManager: mockNetwork,
            cache: mockCache
        )
        
        // When
        let users = await userService.loadUsers()
        
        // Then
        XCTAssertEqual(users.count, 3)
        XCTAssertTrue(mockNetwork.fetchUsersCalled)
    }
}
```

**DI의 철학:**
의존성 주입은 "묻지 말고 말해라(Tell, Don't Ask)" 원칙의 구현이다. 객체는 자신이 필요한 것을 요구하지 않고, 필요한 것을 제공받는다. 이는 더 선언적이고 예측 가능한 코드로 이어진다.

DI는 처음에는 복잡해 보이지만, 코드베이스가 커질수록 그 가치가 드러난다. 특히 CI/CD 환경에서 빠른 테스트 실행이 필요할 때, 네트워크나 데이터베이스 의존성을 모킹할 수 있는 능력은 필수적이다.

## Connections
→ [[292-modular-architecture-spm]]
→ [[293-testing-strategies-ios]]
← [[290-mvvm-vs-tca-architecture]]
← [[061-swift-testing-revolution]]

---
Level: L5
Date: 2025-08-16
Tags: #dependency-injection #testing #architecture #solid-principles #modular-design