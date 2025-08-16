# @Observable 매크로: 단순함의 복귀

## Core Insight
@Observable은 ObservableObject의 보일러플레이트를 제거한다. 매크로가 복잡함을 숨기고 본질만 남긴다.

iOS 17의 @Observable은 패러다임 시프트다. 더 이상 @Published를 붙일 필요 없다. 모든 저장 프로퍼티가 자동으로 관찰 가능하다. objectWillChange.send()도 사라진다. 

```swift
@Observable class ViewModel {
    var count = 0  // 자동으로 관찰됨
    var name = ""  // 이것도 자동으로
}
```

Observation 프레임워크는 세밀한 업데이트를 가능하게 한다. ObservableObject는 객체 전체가 바뀌었다고 알리지만, @Observable은 정확히 어떤 프로퍼티가 바뀌었는지 안다. 불필요한 뷰 업데이트가 사라진다.

withObservationTracking은 수동 관찰을 가능하게 한다. 특정 프로퍼티의 변경을 코드로 감지한다. Combine 없이도 반응형 프로그래밍이 가능하다.

## Connections
→ [[033-macro-system-architecture]]
→ [[034-observation-performance]]
← [[030-swiftui-state-management]]

---
Level: L2
Date: 2025-08-15
Tags: #ios #swift #observable #macros #ios17