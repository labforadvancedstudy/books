# State as Single Source of Truth

## Core Insight
상태는 분산되면 안 된다. 진실은 하나여야 한다. 동기화는 버그의 온상이다.

전통적인 MVC에서 Model과 View가 각자 상태를 가졌다. 그래서 동기화 코드가 필요했다. Model이 바뀌면 View를 업데이트하고, View가 바뀌면 Model을 업데이트한다. 이 양방향 바인딩이 복잡성의 근원이다.

SwiftUI는 단방향 데이터 흐름을 강제한다. State → View. View는 상태를 표현만 하고 소유하지 않는다. 상태가 바뀌면 View는 자동으로 다시 그려진다.

@State는 View의 private 진실이다. @Binding은 진실의 참조다. @StateObject는 참조 타입의 진실이다. 각각은 명확한 소유권과 범위를 가진다.

Environment는 암묵적 진실의 전파다. ColorScheme, Locale, TimeZone - 시스템 상태가 트리 전체에 스며든다. 명시적 전달 없이 모든 View가 일관된 상태를 공유한다.

## Connections
→ [[030-swiftui-state-management]]
→ [[032-environment-values]]
← [[004-swiftui-declarative-philosophy]]

---
Level: L5
Date: 2025-08-15
Tags: #ios #swiftui #state #architecture