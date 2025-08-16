# Haptic Feedback: 디지털 물리학

## Core Insight
촉각 피드백은 가상 객체에 질량과 질감을 부여한다. 진동의 패턴이 곧 디지털 물질의 성질이다.

Taptic Engine은 단순한 진동 모터가 아니다. 이는 **Linear Resonant Actuator**로, 정밀한 파형을 생성할 수 있다. Apple은 세 가지 기본 패턴을 정의했다:

```swift
// 각각 다른 질감을 가진다
UIImpactFeedbackGenerator(style: .light)    // 종이 터치
UIImpactFeedbackGenerator(style: .medium)   // 나무 터치  
UIImpactFeedbackGenerator(style: .heavy)    // 금속 터치

UINotificationFeedbackGenerator().notificationOccurred(.success)  // 성공의 질감
UISelectionFeedbackGenerator().selectionChanged()  // 선택의 질감
```

심리학적으로 haptic은 **confirmation bias**를 강화한다. 버튼을 누르고 진동을 느끼면 뇌는 "정말 뭔가를 눌렀다"고 확신한다. 이는 물리적 버튼보다 더 확실한 피드백이 될 수 있다.

과용은 금물이다. 모든 터치에 피드백을 주면 사용자는 마비된다. haptic은 **의미 있는 순간**에만 사용해야 한다:
- 중요한 액션 완료
- 경계선 교차 (switch toggle)
- 에러나 성공 상태
- 선택의 변화

## Connections
→ [[103-touch-response-timing]]
→ [[105-haptic-design-patterns]]
← [[101-gesture-recognizer-system]]

---
Level: L3
Date: 2025-08-16
Tags: #ios #haptic #feedback #psychology #ux