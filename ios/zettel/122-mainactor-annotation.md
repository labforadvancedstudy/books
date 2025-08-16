# @MainActor: UI의 지배자

## Core Insight
@MainActor는 모든 UI 작업이 메인 스레드에서 실행되도록 보장하는 글로벌 액터다. SwiftUI의 안전장치다.

```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var items: [Item] = []
    
    func loadItems() async {
        // 자동으로 메인 스레드에서 실행
        let newItems = await networkService.fetchItems()
        items = newItems  // UI 업데이트 안전
    }
}
```

UIKit 시대에는 실수하기 쉬웠다:
```swift
// 위험한 코드 - iOS 15 이전
DispatchQueue.global().async {
    let data = heavyComputation()
    // 여기서 UI 업데이트하면 크래시!
    self.label.text = data  
}
```

@MainActor는 **컴파일 타임**에 이런 실수를 방지한다. 메인 스레드가 아닌 곳에서 UI를 건드리려 하면 컴파일 에러가 난다.

흥미로운 점은 **inheritance propagation**이다:
```swift
@MainActor
protocol ViewUpdatable {
    func updateView()
}

// 이 클래스의 모든 메소드는 자동으로 @MainActor
class MyViewController: UIViewController, ViewUpdatable {
    func updateView() {
        // 자동으로 메인 스레드 보장
    }
}
```

성능 최적화: nonisolated를 사용해 필요시 메인 스레드를 벗어날 수 있다:
```swift
@MainActor
class DataProcessor {
    nonisolated func heavyComputation() -> Data {
        // 백그라운드에서 실행 가능
    }
}
```

## Connections
→ [[123-nonisolated-keyword]]
→ [[126-ui-thread-safety]]
→ [[127-background-processing]]
← [[121-actor-isolation-model]]

---
Level: L3
Date: 2025-08-16
Tags: #swift6 #mainactor #ui #thread-safety #compiler