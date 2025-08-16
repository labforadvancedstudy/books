# 테스팅의 본질: 확신을 코드화하기

## Core Insight
테스트는 버그를 찾는 것이 아니라 의도를 문서화하는 것이다. 최고의 테스트는 "이 코드가 왜 존재하는가?"에 답한다.

## The Fundamental Question
왜 테스트를 작성하는가? 

First principles:
1. **엔트로피**: 코드는 자연적으로 부패한다
2. **인지 한계**: 인간은 7±2개 이상을 동시에 기억할 수 없다
3. **변화의 불가피성**: 요구사항은 반드시 변한다
4. **신뢰의 필요성**: 배포 버튼을 누를 수 있는 용기

## Testing Pyramid: 재해석

전통적 피라미드는 틀렸다. 진짜는 이렇다:

```swift
// 가치 피라미드
struct TestingValue {
    // 최상단: 사용자 여정 테스트 (1%)
    let userJourney = "실제 사용자가 목표를 달성하는가?"
    
    // 중간: 통합 테스트 (20%)
    let integration = "컴포넌트들이 함께 작동하는가?"
    
    // 하단: 단위 테스트 (79%)
    let unit = "각 함수가 약속을 지키는가?"
    
    // 숨겨진 기초: 타입 시스템
    let types = "컴파일러가 보장하는 것"
}
```

## Unit Testing: 약속의 코드화

```swift
// 테스트는 계약서다
final class PaymentTests: XCTestCase {
    func test_payment_succeeds_with_valid_card() async throws {
        // Given: 명확한 전제조건
        let card = ValidCard.visa
        let amount = Money(100, .usd)
        
        // When: 단 하나의 행동
        let result = await processor.charge(card, amount)
        
        // Then: 검증 가능한 약속
        #expect(result.isSuccess)
        #expect(result.charged == amount)
        
        // 이 테스트는 말한다:
        // "유효한 카드로 결제하면 반드시 성공한다"
    }
}
```

## UI Testing: 사용자 관점의 진실

```swift
// UI 테스트는 사용자 스토리다
final class OnboardingUITests: XCTestCase {
    func test_new_user_can_complete_onboarding() throws {
        // 사용자의 여정을 코드로
        let app = XCUIApplication()
        app.launch()
        
        // "처음 앱을 열었을 때"
        #expect(app.staticTexts["Welcome"].exists)
        
        // "시작하기를 탭하면"
        app.buttons["Get Started"].tap()
        
        // "이름을 입력할 수 있고"
        let nameField = app.textFields["Your Name"]
        nameField.tap()
        nameField.typeText("Steve")
        
        // "계속하기를 탭하면"
        app.buttons["Continue"].tap()
        
        // "홈 화면에 도달한다"
        #expect(app.navigationBars["Home"].exists)
        
        // 이것은 기능 명세가 아니라 경험 명세다
    }
}
```

## Integration Testing: 현실의 복잡성

```swift
// 통합 테스트는 현실이다
final class SyncIntegrationTests: XCTestCase {
    func test_offline_changes_sync_when_online() async throws {
        // 실제 시나리오
        let sync = SyncManager()
        
        // 오프라인 상태에서 변경
        NetworkSimulator.goOffline()
        let localChange = await createLocalChange()
        
        // 온라인 복귀
        NetworkSimulator.goOnline()
        await Task.sleep(for: .seconds(2))
        
        // 동기화 확인
        let remoteData = await fetchFromServer()
        #expect(remoteData.contains(localChange))
        
        // 이것은 실제 사용자가 겪는 상황이다
    }
}
```

## The Testing Mindset

```swift
// 테스트 가능한 설계
protocol DateProviding {
    var now: Date { get }
}

// 나쁜 설계: 테스트 불가능
class BadSubscription {
    func isExpired() -> Bool {
        return expiryDate < Date() // 테스트할 때마다 다른 결과
    }
}

// 좋은 설계: 의존성 주입
class GoodSubscription {
    let dateProvider: DateProviding
    
    func isExpired() -> Bool {
        return expiryDate < dateProvider.now // 제어 가능
    }
}

// 테스트에서
struct MockDateProvider: DateProviding {
    let now = Date(timeIntervalSince1970: 1234567890)
}
```

## Snapshot Testing: 시각적 계약

```swift
// 스냅샷은 시각적 약속이다
final class ViewSnapshotTests: XCTestCase {
    func test_button_states() {
        let button = CustomButton()
        
        // 모든 상태를 문서화
        assertSnapshot(of: button.configured(as: .normal))
        assertSnapshot(of: button.configured(as: .highlighted))
        assertSnapshot(of: button.configured(as: .disabled))
        
        // 픽셀 하나가 바뀌면 의도적인지 묻는다
    }
}
```

## Performance Testing: 속도의 약속

```swift
final class PerformanceTests: XCTestCase {
    func test_search_performance() {
        let data = Array(repeating: "item", count: 10_000)
        
        measure {
            // 이 작업은 반드시 X초 이내에 완료되어야 한다
            _ = search(in: data, for: "specific")
        }
        
        // 성능 저하는 즉시 발견된다
    }
}
```

## Test Driven Development: 의도 먼저

```swift
// TDD는 설계 도구다
final class TDDExample: XCTestCase {
    func test_what_i_want_to_build() {
        // 1. 원하는 API를 먼저 작성
        let calculator = TaxCalculator()
        let tax = calculator.calculate(income: 100_000)
        
        #expect(tax == 25_000) // 25% 세율
        
        // 2. 컴파일 에러가 설계를 가이드한다
        // 3. 최소한의 코드로 통과시킨다
        // 4. 리팩토링으로 개선한다
    }
}
```

## Mocking: 경계의 정의

```swift
// Mock은 외부 세계와의 계약
protocol NetworkClient {
    func fetch<T>(_ endpoint: Endpoint) async throws -> T
}

class MockNetworkClient: NetworkClient {
    var responses: [Endpoint: Any] = [:]
    var errors: [Endpoint: Error] = [:]
    var callCount = 0
    
    func fetch<T>(_ endpoint: Endpoint) async throws -> T {
        callCount += 1
        
        if let error = errors[endpoint] {
            throw error // 실패 시나리오 테스트
        }
        
        guard let response = responses[endpoint] as? T else {
            throw TestError.noMockData
        }
        
        return response // 성공 시나리오 테스트
    }
}
```

## The Coverage Fallacy

100% 커버리지 ≠ 버그 없음

```swift
// 의미 없는 100% 커버리지
func divide(_ a: Int, by b: Int) -> Int {
    return a / b
}

func test_divide() {
    #expect(divide(10, by: 2) == 5) // 100% 커버리지!
    // 하지만 divide(10, by: 0)은? 💥
}

// 의미 있는 테스트
func test_divide_handles_zero() {
    #expect(throws: DivisionError.self) {
        try safeDivide(10, by: 0)
    }
}
```

## Continuous Integration: 자동화된 확신

```swift
// CI는 팀의 심장박동이다
struct CIPipeline {
    let stages = [
        "Compile",           // 30초: 문법적 정확성
        "Unit Tests",        // 2분: 로직 검증
        "Integration Tests", // 5분: 컴포넌트 협업
        "UI Tests",         // 15분: 사용자 경험
        "Performance Tests", // 3분: 속도 약속
        "Deploy to TestFlight" // 자동 배포
    ]
    
    // 모든 커밋은 이 파이프라인을 통과해야 한다
}
```

## Testing Philosophy

테스트의 진정한 가치:
1. **두려움 제거**: "이 변경이 뭔가 망가뜨릴까?"
2. **문서화**: 테스트는 살아있는 문서
3. **설계 피드백**: 테스트하기 어렵다 = 설계가 나쁘다
4. **팀 커뮤니케이션**: 의도를 코드로 전달

## The Test Pyramid Reality

```
        🧑 사용자 여정 (E2E)
         "앱이 가치를 전달하는가?"
              (느리지만 중요)
                   
       🔗 통합 테스트
      "부품들이 함께 작동하는가?"
          (균형점)
           
    🧩 단위 테스트
   "각 부품이 제 역할을 하는가?"
       (빠르고 많이)
        
 🏗️ 컴파일러 / 타입 시스템
"코드가 말이 되는가?"
   (무료 테스트)
```

## Future: AI-Assisted Testing

```swift
// AI가 테스트를 생성하는 미래
class AITestGenerator {
    func generateTests(for code: String) -> [TestCase] {
        // 1. 코드 의도 파악
        let intent = analyzeIntent(code)
        
        // 2. Edge case 자동 발견
        let edgeCases = findEdgeCases(code)
        
        // 3. 테스트 생성
        return edgeCases.map { edge in
            TestCase(
                given: edge.precondition,
                when: edge.action,
                then: edge.expectedResult
            )
        }
    }
}
```

## The Ultimate Test

진짜 테스트: 새로운 팀원이 테스트만 읽고 코드의 의도를 이해할 수 있는가?

테스트는 미래의 자신에게 보내는 편지다.

## Connections
→ [[091-xctest-patterns]]
→ [[092-ui-testing-strategies]]
→ [[061-swift-testing-revolution]]
→ [[093-continuous-integration]]
← [[001-ios-app-essence]]

---
Level: L8
Date: 2025-08-15
Tags: #ios #testing #quality #xctest #tdd #ci-cd