# 성능: 보이지 않는 기능

## Core Insight
성능은 기능이다. 1초의 지연이 7%의 전환율 감소를 만든다. 빠른 앱은 사용자를 존중하는 앱이다.

Instruments는 현미경이다. Time Profiler는 CPU 사용을, Allocations는 메모리를, Energy Log는 배터리 소모를 보여준다. 추측하지 말고 측정하라.

LazyVStack, LazyHGrid - Lazy 프리픽스는 성능의 약속이다. 보이는 것만 렌더링한다. 천 개의 아이템도 10개처럼 빠르다. 가상화가 기본이 되어야 한다.

.task 수정자는 비동기 작업의 수명을 View와 연결한다. View가 사라지면 작업도 취소된다. 메모리 누수가 구조적으로 불가능해진다.

Image의 .resizable()과 .scaledToFit()의 순서도 성능에 영향을 준다. 작은 디테일이 큰 차이를 만든다. 모든 수정자는 비용이 있다.

## Connections
→ [[065-lazy-loading-patterns]]
→ [[066-memory-management-arc]]
← [[004-swiftui-declarative-philosophy]]

---
Level: L5
Date: 2025-08-15
Tags: #ios #performance #optimization #instruments