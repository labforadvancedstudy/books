# 터치: 인간의 가장 원시적 인터페이스

## Core Insight
터치는 인간이 세상을 탐색하는 가장 기본적인 방법이다. iOS는 이 원시성을 디지털 공간으로 옮겨온 최초의 플랫폼이다.

마우스는 간접적이다. 손가락은 직접적이다. 화면을 터치하는 순간, 우리는 디지털 객체를 "만지고" 있다는 착각에 빠진다. 이 착각이 바로 iOS의 마법이다.

터치 이벤트는 단순해 보이지만 실제로는 복잡한 물리학이다:
- **압력**: Force Touch로 의도의 강도를 전달
- **속도**: 제스처의 momentum을 계산
- **면적**: 손가락 끝과 손가락 배의 차이
- **동시성**: 멀티터치로 복잡한 조작 가능

가장 중요한 것은 **피드백**이다. 터치에 대한 즉각적인 시각/청각/촉각 반응이 없으면 인터페이스는 죽은 것이나 같다. 버튼을 누르는 순간 0.1초 이내에 반응이 와야 한다.

## Connections
→ [[101-gesture-recognizer-system]]
→ [[102-haptic-feedback-psychology]]
→ [[103-touch-response-timing]]
← [[001-ios-app-essence]]

---
Level: L3
Date: 2025-08-16
Tags: #ios #touch #interface #human-factors