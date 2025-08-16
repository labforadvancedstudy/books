# 사용자 의도 중심 아키텍처

## Core Insight
앱 아키텍처는 기술 스택이 아니라 사용자의 의도 흐름을 반영해야 한다. MVVM이나 VIPER가 중요한 게 아니라 사용자가 무엇을 원하는지가 중요하다.

전통적인 아키텍처는 데이터 흐름에 집착한다. Model에서 View로, View에서 Controller로. 하지만 사용자는 MVC를 모른다. 사용자는 "사진을 찍고 싶다", "친구와 공유하고 싶다"는 의도만 있을 뿐이다.

App Intents는 이 패러다임 전환을 시스템 레벨에서 구현한다. Siri, Shortcuts, Spotlight가 이해하는 것은 함수 호출이 아니라 사용자의 의도다. "커피 주문하기", "운동 시작하기" - 이것이 진짜 API다.

각 View는 하나의 명확한 의도를 구현해야 한다. NavigationStack은 의도의 연쇄를 표현한다. 사진 선택 → 필터 적용 → 캡션 작성 → 공유. 각 단계는 독립적이면서도 자연스럽게 연결된다.

## Connections
→ [[006-app-intents-philosophy]]
→ [[007-navigation-as-intention-flow]]
← [[001-ios-app-essence]]

---
Level: L7
Date: 2025-08-15
Tags: #ios #architecture #app-intents #user-intent