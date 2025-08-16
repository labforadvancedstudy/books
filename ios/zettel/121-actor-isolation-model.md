# Actor Isolation: 안전한 섬

## Core Insight
Actor는 자신만의 격리된 실행 컨텍스트를 가진다. 동시성 프로그래밍에서 "안전한 섬"이다.

```swift
actor BankAccount {
    private var balance: Int = 0
    
    func deposit(_ amount: Int) {
        balance += amount  // 안전: 오직 이 actor만 접근 가능
    }
    
    func withdraw(_ amount: Int) -> Bool {
        guard balance >= amount else { return false }
        balance -= amount
        return true
    }
}
```

Actor의 핵심 원리:
1. **Serial execution**: 모든 메소드는 순차적으로 실행
2. **Isolation boundary**: 외부에서 mutable state 직접 접근 불가
3. **Cross-actor calls**: 다른 actor 호출은 항상 async

가장 혁신적인 부분은 **automatic reentrancy**다. Actor 메소드가 await를 만나면 다른 메소드가 실행될 수 있다:

```swift
actor DatabaseManager {
    func processLargeDataset() async {
        for item in largeDataset {
            await processItem(item)  // 여기서 다른 메소드가 끼어들 수 있다
            // 돌아왔을 때 상태가 변했을 수도 있다!
        }
    }
}
```

이는 **assumption invalidation**의 위험을 만든다. await 전후로 불변성을 가정하면 안 된다.

## Connections
→ [[122-mainactor-annotation]]
→ [[124-actor-reentrancy-problem]]
→ [[125-global-actor-pattern]]
← [[120-swift6-sendable-protocol]]

---
Level: L4
Date: 2025-08-16
Tags: #swift6 #actor #isolation #concurrency #reentrancy