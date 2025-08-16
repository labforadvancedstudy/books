# Swift Testing: 테스트의 재발명

## Core Insight
Swift Testing은 XCTest를 대체하는 것이 아니라 테스트를 다시 생각하게 만든다. 매크로와 표현력이 만나면 테스트가 문서가 된다.

```swift
@Test("User can order coffee")
func orderCoffee() async throws {
    let service = CoffeeService()
    let order = try await service.order(.latte, size: .large)
    
    #expect(order.status == .confirmed)
    #expect(order.estimatedTime > 0)
}

@Test("Orders with different sizes", 
      arguments: [CoffeeSize.small, .medium, .large])
func orderWithSize(_ size: CoffeeSize) async throws {
    let order = try await CoffeeService().order(.latte, size: size)
    #expect(order.size == size)
}

@Suite("Payment Processing")
struct PaymentTests {
    @Test func successfulPayment() { }
    @Test func failedPayment() { }
    @Test func refund() { }
}
```

#expect는 XCTAssert보다 표현력이 좋다. 실패하면 실제 값과 예상 값을 명확히 보여준다. 디버깅이 쉬워진다.

Parameterized tests는 테스트 폭발을 방지한다. 하나의 테스트가 여러 입력으로 실행된다. 테이블 기반 테스트가 자연스러워진다.

@Suite는 테스트를 구조화한다. 관련 테스트를 그룹화하고, 공통 설정을 공유한다. 테스트도 아키텍처가 필요하다.

## Connections
→ [[062-test-driven-swiftui]]
→ [[063-snapshot-testing]]
← [[026-swift6-concurrency-revolution]]

---
Level: L3
Date: 2025-08-15
Tags: #ios #testing #swift-testing #quality