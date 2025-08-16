# Sendable: 동시성의 계약

## Core Insight
Sendable은 "이 타입은 다른 스레드로 안전하게 보낼 수 있다"는 컴파일러와의 약속이다. 메모리 안전성의 새로운 차원이다.

```swift
// 이 프로토콜은 마커일 뿐이지만 강력하다
struct UserProfile: Sendable {
    let name: String
    let age: Int
    // 모든 프로퍼티가 Sendable이어야 한다
}

// 위험: 가변 상태는 Sendable이 될 수 없다
class MutableCounter {  // 컴파일 에러가 날 것
    var count = 0
}
```

Sendable의 규칙은 엄격하다:
1. **Value types**: struct, enum은 모든 프로퍼티가 Sendable이면 자동으로 Sendable
2. **Reference types**: class는 final이고 모든 프로퍼티가 immutable이어야 함
3. **Actor types**: 모든 actor는 자동으로 Sendable
4. **Functions**: `@Sendable` 클로저만 다른 isolation domain으로 전달 가능

가장 중요한 것은 **data race 방지**다. 두 스레드가 동시에 같은 메모리를 변경하려 할 때 발생하는 버그를 컴파일 타임에 잡는다:

```swift
actor DataManager {
    private var cache: [String: Data] = [:]
    
    func store(_ data: Data, key: String) {
        cache[key] = data  // 안전: actor 내부
    }
}
```

## Connections
→ [[121-actor-isolation-model]]
→ [[122-mainactor-annotation]]
→ [[123-nonisolated-keyword]]
← [[026-swift6-concurrency-revolution]]

---
Level: L4
Date: 2025-08-16
Tags: #swift6 #sendable #concurrency #memory-safety #protocol