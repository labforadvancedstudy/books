# Gesture Recognizer: 의도 해석 엔진

## Core Insight
Gesture Recognizer는 원시적 터치 데이터를 인간의 의도로 번역하는 해석기다. 패턴 인식의 예술이다.

```swift
// 단순해 보이지만 수십 개의 변수가 숨어있다
let tapGesture = UITapGestureRecognizer(target: self, action: #selector(handleTap))
tapGesture.numberOfTapsRequired = 2
tapGesture.numberOfTouchesRequired = 1
```

시스템이 해결해야 하는 모호성:
- **Tap vs Pan**: 손가락이 살짝 움직였을 때는?
- **Pan vs Swipe**: 속도로 구분해야 하는가?
- **Pinch vs Rotation**: 두 손가락의 복합 동작
- **Long Press vs Tap**: 시간 임계값은 얼마?

iOS의 천재성은 이런 모호성을 **상태 기계**로 해결한 것이다:
`Possible → Began → Changed → Ended/Cancelled`

가장 어려운 문제는 **제스처 간 우선순위**다. ScrollView 안의 버튼을 터치하면 무엇이 우선인가? 스크롤인가 버튼 탭인가? iOS는 `delayContentTouches`와 `cancelsTouchesInView`로 이를 제어한다.

## Connections
→ [[102-haptic-feedback-psychology]]
→ [[104-gesture-conflict-resolution]]
← [[100-touch-as-primary-input]]

---
Level: L2
Date: 2025-08-16
Tags: #ios #gesture #recognizer #state-machine