# SwiftUI State Management 패턴

## Core Insight
State 관리는 선택이 아니라 설계다. @State, @StateObject, @ObservedObject, @EnvironmentObject - 각각은 다른 수명과 범위를 가진다.

@State는 View의 사적 진실이다. Toggle의 on/off, TextField의 텍스트. View가 소유하고 View와 함께 사라진다. struct View의 불변성 속에서 유일한 가변 공간이다.

@StateObject는 참조 타입의 소유권이다. View가 ObservableObject를 생성하고 소유한다. View가 재생성되어도 살아남는다. ViewModel의 자연스러운 집이다.

@ObservedObject는 참조 타입의 관찰이다. 다른 곳에서 생성된 객체를 지켜본다. 소유하지 않고 반응한다. 부모에서 자식으로 전달되는 의존성이다.

@EnvironmentObject는 암묵적 의존성이다. 명시적 전달 없이 트리 전체에서 접근한다. 남용하면 숨겨진 결합이 되지만, 적절히 사용하면 보일러플레이트를 제거한다.

## Connections
→ [[031-observable-macro-pattern]]
→ [[032-environment-values]]
← [[004-swiftui-declarative-philosophy]]

---
Level: L3
Date: 2025-08-15
Tags: #ios #swiftui #state-management #observable