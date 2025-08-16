# Widget: 앱의 창문

## Core Insight
Widget은 미니 앱이 아니라 앱으로의 창문이다. 한 눈에 파악 가능한 정보와 한 번의 탭으로 이어지는 행동이 전부다.

Widget의 본질은 제약이다. 상호작용 없음, 애니메이션 없음, 실시간 업데이트 제한. 이 제약이 오히려 본질에 집중하게 만든다. 사용자가 지금 당장 알아야 할 것은 무엇인가? 

TimelineProvider는 시간의 흐름을 미리 계산한다. 미래를 예측해서 시스템에 전달하면, 시스템이 적절한 때에 보여준다. 이는 배터리와 성능을 위한 타협이 아니라 올바른 설계다. Widget은 살아있는 것이 아니라 스냅샷의 연속이다.

WidgetFamily(.small, .medium, .large)는 단순한 크기 옵션이 아니다. 각각은 다른 수준의 정보 밀도를 요구한다. Small은 숫자 하나, Medium은 한 줄 설명, Large는 목록. 같은 정보를 다른 깊이로 보여주는 예술이다.

## Connections
→ [[011-timeline-provider-pattern]]
→ [[012-widget-deep-linking]]
← [[015-home-screen-real-estate]]

---
Level: L5
Date: 2025-08-15
Tags: #ios #widget #widgetkit #glanceable