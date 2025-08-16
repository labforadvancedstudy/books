# SwiftUI의 선언적 철학

## Core Insight
SwiftUI는 "어떻게"에서 "무엇"으로의 패러다임 전환이다. 상태가 곧 UI이고, UI가 곧 상태다. 이 동일성이 복잡성을 제거한다.

UIKit에서 우리는 명령했다. "레이블을 만들어라", "텍스트를 바꿔라", "화면을 다시 그려라". SwiftUI에서는 선언한다. "이 상태일 때 이렇게 보여라". 명령형에서 선언형으로의 전환은 단순한 문법 변화가 아니라 사고의 혁명이다.

@State, @Binding, @ObservedObject는 단순한 프로퍼티 래퍼가 아니다. 이들은 진실의 단일 원천(Single Source of Truth)을 코드로 표현한 것이다. 상태가 바뀌면 UI는 자동으로 따라온다. 동기화 코드는 사라진다.

View는 상태의 함수다. View = f(State). 이 단순한 공식이 수천 줄의 보일러플레이트를 제거한다. 더 이상 viewDidLoad나 layoutSubviews를 고민할 필요 없다. 그저 현재 상태에서 어떻게 보여야 하는지만 선언하면 된다.

## Connections
→ [[008-state-as-truth]]
→ [[009-view-as-function]]
← [[002-interface-invisibility]]

---
Level: L6
Date: 2025-08-15
Tags: #ios #swiftui #declarative #paradigm