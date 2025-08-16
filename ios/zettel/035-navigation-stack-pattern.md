# NavigationStack: 선언적 네비게이션

## Core Insight
NavigationStack은 네비게이션을 상태로 만든다. 화면 이동이 명령이 아니라 데이터 구조가 된다.

NavigationPath는 네비게이션 히스토리를 배열로 표현한다. append하면 push되고, removeLast하면 pop된다. 네비게이션이 투명해진다. 어디서 왔고 어디로 갈 수 있는지 코드로 볼 수 있다.

```swift
@State private var path = NavigationPath()
// path.append(destination) - push
// path.removeLast() - pop  
// path = .init() - root로
```

navigationDestination은 타입 기반 라우팅이다. Destination의 타입이 곧 화면이다. String이면 DetailView, User면 ProfileView. 타입 시스템이 네비게이션을 안전하게 만든다.

Deep linking이 자연스러워진다. URL을 파싱해서 NavigationPath를 구성하면 끝이다. 복잡한 라우팅 로직이 단순한 데이터 변환이 된다.

## Connections
→ [[036-type-safe-navigation]]
→ [[037-deep-linking-architecture]]
← [[003-user-intention-architecture]]

---
Level: L3
Date: 2025-08-15
Tags: #ios #swiftui #navigation #routing