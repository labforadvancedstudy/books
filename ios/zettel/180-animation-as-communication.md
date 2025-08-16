# Animation: 움직임의 언어

## Core Insight
애니메이션은 장식이 아니라 의사소통의 수단이다. 움직임으로 사용자에게 무엇이 일어나고 있는지 설명한다.

```swift
// 단순해 보이지만 복잡한 심리학이 숨어있다
withAnimation(.easeInOut(duration: 0.3)) {
    isExpanded.toggle()
}
```

애니메이션의 세 가지 목적:
1. **Orientation**: 어디서 어디로 이동했는지 알려주기
2. **Feedback**: 액션이 성공했는지 실패했는지 전달
3. **Continuity**: 화면 전환 시 연결감 유지

Disney의 12가지 애니메이션 원칙이 iOS에도 적용된다:
- **Ease In/Out**: 현실의 물리법칙을 따라 가속/감속
- **Anticipation**: 큰 움직임 전에 작은 예고 동작
- **Squash and Stretch**: 버튼이 눌릴 때 살짝 변형
- **Follow Through**: 메인 액션 후 잔여 움직임

iOS의 표준 timing:
- **Quick interactions**: 0.1-0.2초 (버튼 피드백)
- **Screen transitions**: 0.3-0.5초 (네비게이션)
- **Complex animations**: 0.8-1.2초 (복잡한 상태 변화)

가장 중요한 원칙: **Purposeful motion**. 모든 애니메이션은 명확한 목적이 있어야 한다. 화려함을 위한 애니메이션은 사용자를 혼란스럽게 만든다.

SwiftUI는 이를 혁신적으로 단순화했다:
```swift
Text(isVisible ? "Hello" : "")
    .opacity(isVisible ? 1 : 0)
    .animation(.easeInOut, value: isVisible)
```

상태의 변화가 곧 애니메이션이다.

## Connections
→ [[181-swiftui-animation-system]]
→ [[182-uikit-animation-legacy]]
→ [[183-spring-animation-physics]]
← [[002-interface-invisibility]]

---
Level: L4
Date: 2025-08-16
Tags: #ios #animation #communication #ux #swiftui #motion