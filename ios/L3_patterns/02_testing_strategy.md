# iOS 테스팅 전략 완벽 가이드

## 핵심: 테스트는 자신감이다

좋은 테스트는 **리팩토링의 자유**를 줍니다. 코드를 바꿔도 기능이 동작한다는 확신이 있을 때, 진짜 개선이 시작됩니다.

## 1. Swift Testing (iOS 18+)

### 새로운 테스팅 패러다임

```swift
import Testing
import SwiftUI

// MARK: - 기본 테스트 구조
@Suite("Order Management Tests")
struct OrderTests {
    // 테스트 전 설정
    init() async throws {
        // 초기화 코드
    }
    
    deinit {
        // 정리 코드
    }
    
    // MARK: - 기본 테스트
    @Test("새 주문 생성")
    func createNewOrder() async throws {
        // Given
        let customer = Customer(
            name: "김철수",
            email: "test@example.com",
            phone: "010-1234-5678"
        )
        
        // When
        let order = Order(
            orderNumber: "ORD-001",
            customer: customer
        )
        
        // Then
        #expect(order.status == .pending)
        #expect(order.items.isEmpty)
        #expect(order.customer?.name == "김철수")
    }
    
    // MARK: - 파라미터화 테스트
    @Test(
        "주문 상태 전환",
        arguments: [
            (from: OrderStatus.pending, to: OrderStatus.confirmed, valid: true),
            (from: OrderStatus.confirmed, to: OrderStatus.preparing, valid: true),
            (from: OrderStatus.delivered, to: OrderStatus.pending, valid: false),
            (from: OrderStatus.cancelled, to: OrderStatus.delivered, valid: false)
        ]
    )
    func orderStatusTransition(
        from: OrderStatus,
        to: OrderStatus,
        valid: Bool
    ) async throws {
        // Given
        var order = Order.sample
        order.status = from
        
        // When
        let result = order.transitionTo(to)
        
        // Then
        if valid {
            #expect(result == true)
            #expect(order.status == to)
        } else {
            #expect(result == false)
            #expect(order.status == from)
        }
    }
    
    // MARK: - 태그 기반 테스트
    @Test("결제 처리", .tags(.payment, .critical))
    func processPayment() async throws {
        // 중요한 결제 테스트
        let payment = Payment(amount: 50000, method: .card)
        let result = try await PaymentProcessor.process(payment)
        
        #expect(result.isSuccess)
        #expect(result.transactionID != nil)
    }
    
    // MARK: - 조건부 테스트
    @Test("푸시 알림", .enabled(if: PushService.isAvailable))
    func sendPushNotification() async throws {
        let notification = OrderNotification(
            title: "주문 확인",
            body: "주문이 확인되었습니다"
        )
        
        let sent = try await PushService.send(notification)
        #expect(sent == true)
    }
    
    // MARK: - 시간 제한 테스트
    @Test("API 응답 시간", .timeLimit(.seconds(3)))
    func apiResponseTime() async throws {
        let start = Date()
        _ = try await APIClient.fetchOrders()
        let elapsed = Date().timeIntervalSince(start)
        
        #expect(elapsed < 3.0)
    }
}

// MARK: - 커스텀 태그
extension Tag {
    @Tag static var payment: Self
    @Tag static var critical: Self
    @Tag static var slow: Self
    @Tag static var integration: Self
}

// MARK: - 복잡한 Expectation
@Suite("Advanced Expectations")
struct AdvancedTests {
    @Test("컬렉션 검증")
    func collectionValidation() async throws {
        let orders = [
            Order(orderNumber: "001", status: .pending),
            Order(orderNumber: "002", status: .confirmed),
            Order(orderNumber: "003", status: .delivered)
        ]
        
        // 다양한 검증
        #expect(orders.count == 3)
        #expect(orders.allSatisfy { !$0.orderNumber.isEmpty })
        #expect(orders.contains { $0.status == .delivered })
        #expect(orders.first?.status == .pending)
        
        // 에러 검증
        #expect(throws: ValidationError.self) {
            try validateOrder(Order.invalid)
        }
        
        // 비동기 검증
        await #expect(throws: NetworkError.self) {
            try await APIClient.fetchWithError()
        }
    }
    
    @Test("Custom Matcher")
    func customMatcher() {
        let order = Order.sample
        
        // 커스텀 매처
        #expect(order.isValid)
        #expect(order.totalAmount > 0)
        #expect(order.estimatedDeliveryTime != nil)
    }
}
```

## 2. TCA (The Composable Architecture) 테스팅

### TestStore 패턴

```swift
import ComposableArchitecture
import XCTest

@MainActor
final class OrderFeatureTests: XCTestCase {
    func testOrderFlow() async {
        // TestStore 생성
        let store = TestStore(
            initialState: OrderFeature.State(),
            reducer: { OrderFeature() }
        ) {
            // Dependencies 목업
            $0.apiClient = .mock
            $0.uuid = .incrementing
            $0.date = .constant(Date(timeIntervalSince1970: 1234567890))
        }
        
        // 주문 생성 테스트
        await store.send(.createOrderButtonTapped) {
            $0.isLoading = true
        }
        
        // API 응답 시뮬레이션
        await store.receive(
            .orderResponse(.success(Order.mock))
        ) {
            $0.isLoading = false
            $0.currentOrder = Order.mock
            $0.orderStatus = .confirmed
        }
        
        // 타이머 효과 테스트
        await store.send(.startTracking) {
            $0.isTracking = true
        }
        
        // 1초 후 업데이트
        await store.advance(by: .seconds(1))
        
        await store.receive(.trackingUpdate) {
            $0.driverLocation = Location.mock
        }
        
        // 완료
        await store.send(.completeOrder) {
            $0.orderStatus = .delivered
            $0.isTracking = false
        }
    }
    
    func testErrorHandling() async {
        let store = TestStore(
            initialState: OrderFeature.State(),
            reducer: { OrderFeature() }
        ) {
            $0.apiClient.fetchOrder = { _ in
                throw APIError.networkError
            }
        }
        
        await store.send(.loadOrder(id: "123")) {
            $0.isLoading = true
        }
        
        await store.receive(
            .orderResponse(.failure(APIError.networkError))
        ) {
            $0.isLoading = false
            $0.alert = AlertState {
                TextState("오류")
            } actions: {
                ButtonState(role: .cancel) {
                    TextState("확인")
                }
            } message: {
                TextState("네트워크 오류가 발생했습니다.")
            }
        }
    }
}

// MARK: - Dependency 목업
extension APIClient: TestDependencyKey {
    static let testValue = APIClient(
        fetchOrders: { [] },
        fetchOrder: { _ in .mock },
        createOrder: { _ in .mock },
        updateOrder: { _, _ in .mock },
        deleteOrder: { _ in }
    )
    
    static let mock = APIClient(
        fetchOrders: {
            [Order.mock, Order.mock2]
        },
        fetchOrder: { id in
            Order(id: id, orderNumber: "MOCK-001")
        },
        createOrder: { order in
            var newOrder = order
            newOrder.id = UUID()
            newOrder.status = .confirmed
            return newOrder
        },
        updateOrder: { id, updates in
            Order(id: id, status: updates.status ?? .pending)
        },
        deleteOrder: { _ in }
    )
}

// MARK: - 통합 테스트
@MainActor
final class AppIntegrationTests: XCTestCase {
    func testFullOrderFlow() async {
        let store = TestStore(
            initialState: AppFeature.State(),
            reducer: { AppFeature() }
        )
        
        // 1. 로그인
        await store.send(.login(.submitted(email: "test@example.com", password: "password"))) {
            $0.login.isLoading = true
        }
        
        await store.receive(.login(.loginResponse(.success(User.mock)))) {
            $0.login.isLoading = false
            $0.user = User.mock
            $0.isLoggedIn = true
        }
        
        // 2. 메뉴 탐색
        await store.send(.tab(.menu)) {
            $0.selectedTab = .menu
        }
        
        await store.send(.menu(.loadMenuItems)) {
            $0.menu.isLoading = true
        }
        
        await store.receive(.menu(.menuItemsLoaded([MenuItem.mock]))) {
            $0.menu.items = [MenuItem.mock]
            $0.menu.isLoading = false
        }
        
        // 3. 장바구니 추가
        await store.send(.menu(.addToCart(MenuItem.mock))) {
            $0.cart.items.append(CartItem(menuItem: MenuItem.mock, quantity: 1))
            $0.cart.badge = 1
        }
        
        // 4. 주문
        await store.send(.cart(.checkout)) {
            $0.cart.isProcessing = true
        }
        
        await store.receive(.cart(.orderCreated(Order.mock))) {
            $0.cart.isProcessing = false
            $0.cart.items = []
            $0.orders.current = Order.mock
            $0.selectedTab = .tracking
        }
    }
}
```

## 3. UI Testing 실전

### XCTest UI 자동화

```swift
import XCTest

final class DeliveryAppUITests: XCTestCase {
    var app: XCUIApplication!
    
    override func setUp() {
        continueAfterFailure = false
        app = XCUIApplication()
        app.launchArguments = ["--uitesting"]
        app.launchEnvironment = [
            "MOCK_API": "true",
            "SKIP_ANIMATIONS": "true"
        ]
        app.launch()
    }
    
    // MARK: - 주문 플로우 테스트
    func testCompleteOrderFlow() {
        // 1. 온보딩 스킵
        if app.buttons["Skip"].exists {
            app.buttons["Skip"].tap()
        }
        
        // 2. 로그인
        let emailField = app.textFields["Email"]
        XCTAssertTrue(emailField.waitForExistence(timeout: 5))
        emailField.tap()
        emailField.typeText("test@example.com")
        
        let passwordField = app.secureTextFields["Password"]
        passwordField.tap()
        passwordField.typeText("password123")
        
        app.buttons["Login"].tap()
        
        // 3. 메뉴 선택
        XCTAssertTrue(app.navigationBars["Menu"].waitForExistence(timeout: 5))
        
        // 첫 번째 메뉴 아이템 선택
        let firstMenuItem = app.cells.element(boundBy: 0)
        XCTAssertTrue(firstMenuItem.waitForExistence(timeout: 3))
        firstMenuItem.tap()
        
        // 4. 장바구니 추가
        app.buttons["Add to Cart"].tap()
        
        // 수량 증가
        app.steppers["Quantity"].buttons["Increment"].tap()
        
        // 5. 장바구니로 이동
        app.tabBars.buttons["Cart"].tap()
        
        // 6. 주문하기
        XCTAssertTrue(app.buttons["Checkout"].exists)
        app.buttons["Checkout"].tap()
        
        // 7. 주소 입력
        let addressField = app.textFields["Delivery Address"]
        addressField.tap()
        addressField.typeText("서울시 강남구 테헤란로 123")
        
        // 8. 결제 정보
        let cardNumberField = app.textFields["Card Number"]
        cardNumberField.tap()
        cardNumberField.typeText("4242424242424242")
        
        // 9. 주문 확인
        app.buttons["Place Order"].tap()
        
        // 10. 추적 화면 확인
        XCTAssertTrue(app.staticTexts["Order Confirmed"].waitForExistence(timeout: 5))
        XCTAssertTrue(app.maps.element.exists)
    }
    
    // MARK: - 접근성 테스트
    func testAccessibility() {
        // VoiceOver 시뮬레이션
        let menuButton = app.tabBars.buttons["Menu"]
        XCTAssertTrue(menuButton.isHittable)
        XCTAssertEqual(menuButton.label, "Menu")
        XCTAssertEqual(menuButton.value as? String, "Tab 1 of 4")
        
        // Dynamic Type 테스트
        app.buttons["Settings"].tap()
        app.cells["Text Size"].tap()
        app.sliders["Text Size"].adjust(toNormalizedSliderPosition: 0.8)
        
        // 텍스트 크기 확인
        let sampleText = app.staticTexts.element(boundBy: 0)
        let originalFrame = sampleText.frame
        
        app.sliders["Text Size"].adjust(toNormalizedSliderPosition: 1.0)
        let enlargedFrame = sampleText.frame
        
        XCTAssertGreaterThan(enlargedFrame.height, originalFrame.height)
    }
    
    // MARK: - 성능 테스트
    func testScrollPerformance() {
        measure(metrics: [XCTOSSignpostMetric.scrollDecelerationMetric]) {
            app.tables.element.swipeUp(velocity: .fast)
            app.tables.element.swipeDown(velocity: .fast)
        }
    }
}

// MARK: - Page Object Pattern
class LoginPage {
    let app: XCUIApplication
    
    init(app: XCUIApplication) {
        self.app = app
    }
    
    var emailField: XCUIElement {
        app.textFields["Email"]
    }
    
    var passwordField: XCUIElement {
        app.secureTextFields["Password"]
    }
    
    var loginButton: XCUIElement {
        app.buttons["Login"]
    }
    
    var errorMessage: XCUIElement {
        app.staticTexts["ErrorMessage"]
    }
    
    func login(email: String, password: String) {
        emailField.tap()
        emailField.typeText(email)
        
        passwordField.tap()
        passwordField.typeText(password)
        
        loginButton.tap()
    }
    
    func waitForError(timeout: TimeInterval = 5) -> Bool {
        errorMessage.waitForExistence(timeout: timeout)
    }
}

// Page Object 활용
final class LoginUITests: XCTestCase {
    var app: XCUIApplication!
    var loginPage: LoginPage!
    
    override func setUp() {
        app = XCUIApplication()
        app.launch()
        loginPage = LoginPage(app: app)
    }
    
    func testInvalidLogin() {
        loginPage.login(email: "invalid", password: "wrong")
        XCTAssertTrue(loginPage.waitForError())
        XCTAssertEqual(loginPage.errorMessage.label, "Invalid credentials")
    }
}
```

## 4. 스냅샷 테스팅

### swift-snapshot-testing 활용

```swift
import SnapshotTesting
import XCTest
import SwiftUI

final class ViewSnapshotTests: XCTestCase {
    override func setUp() {
        super.setUp()
        // 스냅샷 기록 모드 (최초 실행 시)
        // isRecording = true
    }
    
    // MARK: - View 스냅샷
    func testOrderCardView() {
        let order = Order(
            orderNumber: "ORD-001",
            status: .confirmed,
            items: [
                OrderItem(name: "버거", price: 15000, quantity: 2),
                OrderItem(name: "감자튀김", price: 5000, quantity: 1)
            ]
        )
        
        let view = OrderCardView(order: order)
            .frame(width: 375, height: 200)
        
        // 라이트 모드
        assertSnapshot(
            matching: view,
            as: .image(
                layout: .fixed(width: 375, height: 200),
                traits: .init(userInterfaceStyle: .light)
            ),
            named: "light"
        )
        
        // 다크 모드
        assertSnapshot(
            matching: view,
            as: .image(
                layout: .fixed(width: 375, height: 200),
                traits: .init(userInterfaceStyle: .dark)
            ),
            named: "dark"
        )
        
        // 접근성 (큰 텍스트)
        assertSnapshot(
            matching: view,
            as: .image(
                layout: .fixed(width: 375, height: 250),
                traits: .init(preferredContentSizeCategory: .accessibilityLarge)
            ),
            named: "accessibility"
        )
    }
    
    // MARK: - ViewController 스냅샷
    func testOrderViewController() {
        let vc = OrderViewController()
        vc.configure(with: Order.mock)
        
        // iPhone 15 Pro
        assertSnapshot(
            matching: vc,
            as: .image(on: .iPhone15Pro),
            named: "iPhone15Pro"
        )
        
        // iPad
        assertSnapshot(
            matching: vc,
            as: .image(on: .iPadPro12_9),
            named: "iPadPro"
        )
        
        // 가로 모드
        assertSnapshot(
            matching: vc,
            as: .image(on: .iPhone15Pro(.landscape)),
            named: "landscape"
        )
    }
    
    // MARK: - 전체 화면 스냅샷
    func testFullScreenFlow() {
        let app = XCUIApplication()
        app.launch()
        
        // 각 화면 캡처
        let screens = [
            "Login",
            "Menu",
            "Cart",
            "Checkout",
            "Tracking"
        ]
        
        for screen in screens {
            navigateToScreen(screen)
            
            let screenshot = app.screenshot()
            let attachment = XCTAttachment(screenshot: screenshot)
            attachment.name = "Screen_\(screen)"
            attachment.lifetime = .keepAlways
            add(attachment)
            
            // 스냅샷 비교
            assertSnapshot(
                matching: UIImage(data: screenshot.pngRepresentation)!,
                as: .image,
                named: screen
            )
        }
    }
}

// MARK: - 커스텀 스냅샷 전략
extension Snapshotting where Value == UIViewController, Format == UIImage {
    static func imageWithAccessibility(
        on config: ViewImageConfig,
        perceptualPrecision: Float = 0.98
    ) -> Snapshotting {
        Snapshotting.image(
            on: config,
            perceptualPrecision: perceptualPrecision,
            layout: .device(config: config)
        ).pullback { vc in
            // 접근성 검증 추가
            vc.view.accessibilityElements?.forEach { element in
                guard let element = element as? UIAccessibilityElement else { return }
                assert(!element.accessibilityLabel.isEmpty, "Missing accessibility label")
            }
            return vc
        }
    }
}
```

## 5. 네트워크 목업

### URLProtocol 활용

```swift
// MARK: - Mock URL Protocol
class MockURLProtocol: URLProtocol {
    static var mockResponses: [String: (data: Data?, response: URLResponse?, error: Error?)] = [:]
    
    override class func canInit(with request: URLRequest) -> Bool {
        return true
    }
    
    override class func canonicalRequest(for request: URLRequest) -> URLRequest {
        return request
    }
    
    override func startLoading() {
        guard let url = request.url?.absoluteString,
              let mock = MockURLProtocol.mockResponses[url] else {
            client?.urlProtocol(self, didFailWithError: URLError(.fileDoesNotExist))
            return
        }
        
        // 지연 시뮬레이션
        DispatchQueue.global().asyncAfter(deadline: .now() + 0.1) {
            if let data = mock.data {
                self.client?.urlProtocol(self, didLoad: data)
            }
            
            if let response = mock.response {
                self.client?.urlProtocol(
                    self,
                    didReceive: response,
                    cacheStoragePolicy: .notAllowed
                )
            }
            
            if let error = mock.error {
                self.client?.urlProtocol(self, didFailWithError: error)
            } else {
                self.client?.urlProtocolDidFinishLoading(self)
            }
        }
    }
    
    override func stopLoading() {
        // 정리 작업
    }
}

// MARK: - 테스트에서 사용
class NetworkTests: XCTestCase {
    var session: URLSession!
    
    override func setUp() {
        let config = URLSessionConfiguration.ephemeral
        config.protocolClasses = [MockURLProtocol.self]
        session = URLSession(configuration: config)
        
        // Mock 응답 설정
        MockURLProtocol.mockResponses = [
            "https://api.example.com/orders": (
                data: try? JSONEncoder().encode([Order.mock]),
                response: HTTPURLResponse(
                    url: URL(string: "https://api.example.com/orders")!,
                    statusCode: 200,
                    httpVersion: nil,
                    headerFields: nil
                ),
                error: nil
            )
        ]
    }
    
    func testFetchOrders() async throws {
        let orders = try await session.data(
            from: URL(string: "https://api.example.com/orders")!
        )
        
        let decoded = try JSONDecoder().decode([Order].self, from: orders.0)
        XCTAssertEqual(decoded.count, 1)
        XCTAssertEqual(decoded.first?.orderNumber, "MOCK-001")
    }
}
```

## 6. 성능 테스팅

### XCTest 성능 측정

```swift
final class PerformanceTests: XCTestCase {
    // MARK: - 앱 시작 성능
    func testAppLaunchPerformance() {
        measure(metrics: [XCTApplicationLaunchMetric()]) {
            XCUIApplication().launch()
        }
    }
    
    // MARK: - 메모리 사용량
    func testMemoryUsage() {
        let app = XCUIApplication()
        app.launch()
        
        measure(metrics: [XCTMemoryMetric()]) {
            // 대량 데이터 로드
            app.tables.cells.element(boundBy: 0).tap()
            app.navigationBars.buttons.element(boundBy: 0).tap()
            
            // 스크롤
            for _ in 0..<10 {
                app.tables.element.swipeUp()
            }
        }
    }
    
    // MARK: - CPU 사용량
    func testCPUUsage() {
        measure(metrics: [XCTCPUMetric()]) {
            // CPU 집약적 작업
            let processor = ImageProcessor()
            for _ in 0..<100 {
                _ = processor.processImage(UIImage.mock)
            }
        }
    }
    
    // MARK: - 디스크 I/O
    func testDiskIO() {
        measure(metrics: [XCTStorageMetric()]) {
            let cache = DiskCache()
            
            // 쓰기
            for i in 0..<100 {
                cache.save(data: Data.random(count: 1024), key: "key_\(i)")
            }
            
            // 읽기
            for i in 0..<100 {
                _ = cache.load(key: "key_\(i)")
            }
            
            // 삭제
            cache.clear()
        }
    }
    
    // MARK: - 커스텀 메트릭
    func testCustomMetrics() {
        let metric = XCTMetric(name: "API Response Time")
        
        measure(metrics: [metric]) {
            let expectation = expectation(description: "API call")
            
            APIClient.fetchOrders { _ in
                expectation.fulfill()
            }
            
            wait(for: [expectation], timeout: 10)
        }
    }
}
```

## 7. CI/CD 통합

### GitHub Actions 설정

```yaml
# .github/workflows/test.yml
name: iOS Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: macos-14
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Xcode
      uses: maxim-lobanov/setup-xcode@v1
      with:
        xcode-version: '15.2'
    
    - name: Cache Dependencies
      uses: actions/cache@v3
      with:
        path: |
          ~/Library/Caches/org.swift.swiftpm
          ~/Library/Developer/Xcode/DerivedData
        key: ${{ runner.os }}-spm-${{ hashFiles('Package.resolved') }}
    
    - name: Run Unit Tests
      run: |
        xcodebuild test \
          -scheme DeliveryApp \
          -destination 'platform=iOS Simulator,name=iPhone 15 Pro' \
          -resultBundlePath TestResults \
          -enableCodeCoverage YES
    
    - name: Run UI Tests
      run: |
        xcodebuild test \
          -scheme DeliveryAppUITests \
          -destination 'platform=iOS Simulator,name=iPhone 15 Pro' \
          -resultBundlePath UITestResults
    
    - name: Generate Coverage Report
      run: |
        xcrun xccov view --report TestResults.xcresult > coverage.txt
        
    - name: Upload Coverage
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage.txt
        
    - name: Upload Test Results
      if: always()
      uses: actions/upload-artifact@v3
      with:
        name: test-results
        path: |
          TestResults.xcresult
          UITestResults.xcresult
```

### Fastlane 자동화

```ruby
# fastlane/Fastfile
platform :ios do
  desc "Run all tests"
  lane :test do
    run_tests(
      scheme: "DeliveryApp",
      devices: ["iPhone 15 Pro"],
      code_coverage: true,
      output_types: "junit,html",
      output_directory: "./test_output"
    )
  end
  
  desc "Run UI tests"
  lane :ui_test do
    run_tests(
      scheme: "DeliveryAppUITests",
      devices: ["iPhone 15 Pro", "iPad Pro (12.9-inch)"],
      parallel_testing: true,
      concurrent_workers: 2
    )
  end
  
  desc "Generate coverage report"
  lane :coverage do
    xcov(
      scheme: "DeliveryApp",
      output_directory: "./coverage",
      minimum_coverage_percentage: 80.0
    )
  end
  
  desc "Snapshot tests"
  lane :snapshot do
    snapshot(
      devices: [
        "iPhone 15 Pro",
        "iPhone SE (3rd generation)",
        "iPad Pro (12.9-inch)"
      ],
      languages: ["ko", "en"],
      clear_previous_screenshots: true
    )
  end
end
```

## 8. 테스트 전략 체크리스트

### 테스트 피라미드

```swift
struct TestStrategy {
    let pyramid = """
    단위 테스트 (70%)
    - 빠르고 격리된 테스트
    - 모든 비즈니스 로직
    - Edge cases
    
    통합 테스트 (20%)
    - API 통합
    - 데이터베이스 연동
    - 의존성 상호작용
    
    UI 테스트 (10%)
    - 핵심 사용자 플로우
    - 중요한 UI 상호작용
    - 스냅샷 테스트
    """
    
    let coverage = """
    목표: 80% 이상
    - 비즈니스 로직: 90%+
    - UI 코드: 60%+
    - 유틸리티: 100%
    """
    
    let bestPractices = """
    1. AAA 패턴 (Arrange, Act, Assert)
    2. 테스트당 하나의 개념만
    3. 명확한 테스트 이름
    4. 테스트 격리
    5. Mock 최소화
    6. 실패 시 명확한 메시지
    """
}
```

## 결론

좋은 테스트는 **문서이자 안전망**입니다. Swift Testing의 표현력, TCA의 예측 가능성, UI 테스트의 자동화를 조합하면 견고한 앱을 만들 수 있습니다.

핵심은:
1. **빠른 피드백 루프 구축**
2. **테스트 가능한 아키텍처 설계**
3. **CI/CD로 자동화**
4. **측정 가능한 품질 지표**

"테스트 없는 코드는 레거시 코드다" - 자신감 있게 배포하려면 테스트가 필수입니다.