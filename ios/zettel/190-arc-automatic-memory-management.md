# ARC: 자동 메모리 관리의 혁명

## Core Insight
ARC(Automatic Reference Counting)는 메모리 관리를 컴파일러에게 맡긴 혁명이다. 개발자는 비즈니스 로직에만 집중할 수 있다.

```swift
class DataManager {
    var cache: [String: Data] = [:]  // 자동으로 메모리 관리됨
    
    deinit {
        print("DataManager 해제됨")  // 참조가 0이 되면 자동 호출
    }
}

var manager: DataManager? = DataManager()  // retain count: 1
manager = nil  // retain count: 0, 자동으로 해제
```

ARC의 동작 원리:
1. **Reference counting**: 객체에 대한 참조 개수 추적
2. **Compile-time insertion**: 컴파일러가 retain/release 코드 자동 삽입
3. **Zero-cost abstraction**: 런타임 오버헤드 거의 없음

하지만 함정이 있다: **Strong reference cycles**

```swift
class Parent {
    var children: [Child] = []
}

class Child {
    weak var parent: Parent?  // weak로 순환 참조 방지
}
```

메모리 문제의 90%는 다음 중 하나:
1. **Strong reference cycles**: 서로를 참조하는 객체들
2. **Delegate patterns**: delegate를 strong으로 참조
3. **Closure capture**: self를 강하게 캡처하는 클로저
4. **Observer patterns**: 옵저버 해제 누락

ARC가 해결하지 못하는 것:
- C 메모리 (malloc/free)
- Core Foundation 객체
- 순환 참조

Swift의 해결책:
```swift
// weak와 unowned로 순환 참조 방지
[weak self] in
[unowned self] in
```

## Connections
→ [[191-weak-unowned-references]]
→ [[192-memory-leak-detection]]
→ [[193-closure-capture-semantics]]
← [[064-performance-as-feature]]

---
Level: L3
Date: 2025-08-16
Tags: #ios #arc #memory #references #swift #compiler