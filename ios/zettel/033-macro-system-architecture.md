# Swift Macros: 컴파일 타임 마법

## Core Insight
매크로는 보일러플레이트를 컴파일 타임에 제거한다. 반복되는 코드를 한 번 정의하고 컴파일러가 생성하게 한다.

@Observable, @Model, @Test - 이들은 단순한 어노테이션이 아니다. 컴파일 타임에 코드를 변환하고 확장한다. 수백 줄의 보일러플레이트가 한 줄로 압축된다.

```swift
// 매크로 사용
@Observable
class ViewModel {
    var count = 0
}

// 컴파일러가 생성하는 코드
class ViewModel {
    @ObservationTracked var count = 0
    
    internal let _$observationRegistrar = ObservationRegistrar()
    
    internal nonisolated func access<Member>(
        keyPath: KeyPath<ViewModel, Member>
    ) {
        _$observationRegistrar.access(self, keyPath: keyPath)
    }
    
    internal nonisolated func withMutation<Member, T>(
        keyPath: KeyPath<ViewModel, Member>,
        _ mutation: () throws -> T
    ) rethrows -> T {
        try _$observationRegistrar.withMutation(self, keyPath: keyPath, mutation)
    }
}
```

Expression, Declaration, Accessor, Extension 매크로 - 각각 다른 수준에서 코드를 변환한다. 적절한 타입을 선택하는 것이 중요하다.

매크로는 타입 안전하다. 잘못된 사용은 컴파일 에러를 낸다. 런타임 실패가 아니라 컴파일 타임 검증이다.

## Connections
→ [[073-custom-macro-creation]]
→ [[074-macro-debugging]]
← [[031-observable-macro-pattern]]

---
Level: L4
Date: 2025-08-15
Tags: #ios #swift #macros #metaprogramming