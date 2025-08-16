# iOS 테스팅 전략: 신뢰할 수 있는 코드

## Core Insight
효과적인 테스팅 전략은 테스트 피라미드를 따라 단위 테스트를 기반으로 하고, 통합 테스트와 UI 테스트를 적절히 조합한다.

테스팅은 코드의 품질을 보장하는 유일한 객관적 방법이다. iOS 개발에서 테스트는 단순히 버그를 찾는 것을 넘어서, 코드의 설계를 개선하고 리팩토링을 안전하게 만드는 도구다.

**테스트 피라미드:**
```
    /\
   /UI\     <- 소수의 E2E 테스트 (느림, 비쌈)
  /____\
 /Integration\ <- 중간 수준의 통합 테스트
/__________\
/Unit Tests \ <- 다수의 단위 테스트 (빠름, 저렴)
```

**1. 단위 테스트 (Unit Tests)**
```swift
import Testing
@testable import UserFeature

struct UserServiceTests {
    @Test("사용자 목록 로드 성공")
    func loadUsersSuccess() async throws {
        // Given
        let mockNetwork = MockNetworkManager()
        mockNetwork.stubUsers = [
            User(id: "1", name: "John"),
            User(id: "2", name: "Jane")
        ]
        let userService = UserService(networkManager: mockNetwork)
        
        // When
        let users = try await userService.loadUsers()
        
        // Then
        #expect(users.count == 2)
        #expect(users.first?.name == "John")
    }
    
    @Test("네트워크 에러 처리")
    func loadUsersNetworkError() async {
        // Given
        let mockNetwork = MockNetworkManager()
        mockNetwork.shouldFail = true
        let userService = UserService(networkManager: mockNetwork)
        
        // When & Then
        await #expect(throws: NetworkError.self) {
            try await userService.loadUsers()
        }
    }
}
```

**2. 통합 테스트 (Integration Tests)**
```swift
struct UserIntegrationTests {
    @Test("실제 API와 캐시 통합")
    func userServiceIntegration() async throws {
        // Given
        let config = URLSessionConfiguration.ephemeral
        let session = URLSession(configuration: config)
        let network = URLSessionNetworkManager(session: session)
        let cache = InMemoryCache()
        let userService = UserService(
            networkManager: network,
            cache: cache
        )
        
        // When
        let users = try await userService.loadUsers()
        
        // Then
        #expect(!users.isEmpty)
        
        // 캐시 확인
        let cachedUsers = await cache.getUsers()
        #expect(cachedUsers.count == users.count)
    }
}
```

**3. SwiftUI 뷰 테스트**
```swift
import SwiftUI
import Testing
import ViewInspector

struct UserListViewTests {
    @Test("사용자 목록 표시")
    func userListDisplay() throws {
        // Given
        let users = [
            User(id: "1", name: "John"),
            User(id: "2", name: "Jane")
        ]
        let viewModel = UserListViewModel()
        viewModel.users = users
        
        // When
        let view = UserListView(viewModel: viewModel)
        
        // Then
        let list = try view.inspect().list()
        #expect(try list.count() == 2)
        
        let firstCell = try list.forEach(0).view(UserRowView.self)
        #expect(try firstCell.text().string() == "John")
    }
}
```

**4. UI 테스트 (E2E Tests)**
```swift
import XCTest

final class UserFlowUITests: XCTestCase {
    var app: XCUIApplication!
    
    override func setUp() {
        super.setUp()
        app = XCUIApplication()
        app.launchArguments = ["--uitesting"]
        app.launch()
    }
    
    @MainActor
    func testUserLoginFlow() {
        // Given
        let loginButton = app.buttons["로그인"]
        let emailField = app.textFields["이메일"]
        let passwordField = app.secureTextFields["비밀번호"]
        
        // When
        emailField.tap()
        emailField.typeText("test@example.com")
        passwordField.tap()
        passwordField.typeText("password123")
        loginButton.tap()
        
        // Then
        let welcomeLabel = app.staticTexts["환영합니다"]
        XCTAssertTrue(welcomeLabel.waitForExistence(timeout: 5))
    }
}
```

**테스트 더블 패턴:**

1. **Mock (모의 객체)**
```swift
class MockUserService: UserServiceProtocol {
    var loadUsersCalled = false
    var stubUsers: [User] = []
    var shouldFail = false
    
    func loadUsers() async throws -> [User] {
        loadUsersCalled = true
        
        if shouldFail {
            throw NetworkError.connectionFailed
        }
        
        return stubUsers
    }
}
```

2. **Spy (감시 객체)**
```swift
class SpyAnalytics: AnalyticsProtocol {
    private(set) var trackedEvents: [String] = []
    
    func track(event: String, parameters: [String: Any]) {
        trackedEvents.append(event)
    }
}
```

3. **Fake (가짜 객체)**
```swift
class FakeDatabase: DatabaseProtocol {
    private var users: [User] = []
    
    func save(_ user: User) async {
        users.append(user)
    }
    
    func loadUsers() async -> [User] {
        return users
    }
}
```

**테스트 환경 설정:**
```swift
// 테스트용 앱 설정
@main
struct TestApp: App {
    var body: some Scene {
        WindowGroup {
            if ProcessInfo.processInfo.arguments.contains("--uitesting") {
                ContentView()
                    .environment(\.userService, MockUserService())
                    .environment(\.analytics, MockAnalytics())
            } else {
                ContentView()
            }
        }
    }
}
```

**스냅샷 테스트:**
```swift
import SnapshotTesting

struct UserRowViewTests {
    @Test("사용자 행 레이아웃")
    func userRowSnapshot() {
        let user = User(id: "1", name: "John Doe", email: "john@example.com")
        let view = UserRowView(user: user)
        
        assertSnapshot(matching: view, as: .image)
    }
}
```

**성능 테스트:**
```swift
struct PerformanceTests {
    @Test("대량 데이터 로딩 성능")
    func massiveDataLoading() async {
        let userService = UserService()
        
        await measureAsync {
            _ = try? await userService.loadUsers(limit: 10000)
        }
    }
}
```

**테스트 구성 원칙:**
- **AAA 패턴**: Arrange(준비), Act(실행), Assert(검증)
- **단일 책임**: 하나의 테스트는 하나의 기능만 검증
- **독립성**: 테스트 간 의존성 없음
- **반복 가능**: 언제든 같은 결과
- **명확한 이름**: 테스트 목적이 명확히 드러남

**CI/CD와 통합:**
```yaml
# GitHub Actions 예시
- name: Run Tests
  run: |
    xcodebuild test \
      -scheme MyApp \
      -destination 'platform=iOS Simulator,name=iPhone 15' \
      -enableCodeCoverage YES
```

테스팅은 코드를 작성하는 시간을 늘리지만, 디버깅과 버그 수정에 드는 시간을 극적으로 줄인다. 초기 투자 대비 장기적 수익이 확실한 영역이다.

## Connections
→ [[294-xcode-productivity-techniques]]
→ [[295-continuous-integration-xcode-cloud]]
← [[292-modular-architecture-spm]]
← [[061-swift-testing-revolution]]

---
Level: L4
Date: 2025-08-16
Tags: #testing #unit-test #integration-test #ui-test #tdd #quality-assurance