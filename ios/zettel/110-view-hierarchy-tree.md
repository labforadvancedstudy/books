# View Hierarchy: 화면의 족보

## Core Insight
View는 단순한 사각형이 아니라 부모-자식 관계로 이루어진 가족이다. 이 족보가 곧 앱의 구조다.

UIView 계층구조는 **트리 자료구조**다. 각 View는 하나의 부모와 여러 자식을 가질 수 있다:

```swift
// 간단해 보이지만 복잡한 상속 관계가 숨어있다
containerView.addSubview(childView)
childView.removeFromSuperview()
```

중요한 것은 **좌표계의 상속**이다. 자식 View의 좌표는 항상 부모 기준이다:
- `frame`: 부모 좌표계에서의 위치
- `bounds`: 자신의 좌표계 (0,0부터 시작)
- `center`: 부모 좌표계에서의 중심점

View 계층은 **이벤트 전파**의 경로이기도 하다:
1. 터치 이벤트는 가장 앞의(topmost) View부터 시작
2. `hit-testing`으로 실제 대상 View 찾기
3. Responder Chain을 따라 이벤트 전달

성능의 핵심은 **최소 계층**이다. 불필요한 중간 Container는 렌더링 비용을 증가시킨다. SwiftUI의 `@ViewBuilder`가 혁신적인 이유도 이 때문이다.

## Connections
→ [[111-layout-constraint-system]]
→ [[112-responder-chain-pattern]]
→ [[113-coordinate-space-transform]]
← [[100-touch-as-primary-input]]

---
Level: L2
Date: 2025-08-16
Tags: #ios #view #hierarchy #tree #coordinates