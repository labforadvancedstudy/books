# Auto Layout: 관계의 수학

## Core Insight
Auto Layout은 절대 좌표가 아닌 관계를 통해 레이아웃을 정의한다. 수학적으로는 연립방정식 해결 시스템이다.

```swift
// 이 한 줄은 실제로는 수학 방정식이다
button.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
// 번역: button.centerX = view.centerX + 0 * 1.0
```

Auto Layout의 핵심은 **Cassowary 알고리즘**이다. 제약조건(constraint)들을 받아서 가능한 해를 찾는 수학 엔진이다. 각 제약조건은 다음 형태다:

`item1.attribute1 = multiplier × item2.attribute2 + constant`

우선순위 시스템이 충돌을 해결한다:
- **Required (1000)**: 반드시 만족
- **High (750)**: 가능하면 만족
- **Low (250)**: 여유가 있으면 만족

성능의 함정: 제약조건이 n개일 때 해결 시간은 O(n³)이다. 복잡한 레이아웃에서는 Stack View를 사용해 제약조건 수를 줄여야 한다.

가장 어려운 개념은 **Intrinsic Content Size**다. Label과 Button은 텍스트에 따라 자동으로 크기가 결정된다. 이를 무시하고 강제로 크기를 지정하면 텍스트가 잘릴 수 있다.

## Connections
→ [[112-stack-view-pattern]]
→ [[113-constraint-priority-system]]
→ [[114-intrinsic-content-size]]
← [[110-view-hierarchy-tree]]

---
Level: L3
Date: 2025-08-16
Tags: #ios #autolayout #constraints #cassowary #layout