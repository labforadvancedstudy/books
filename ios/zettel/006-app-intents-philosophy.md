# App Intents: 의도의 프로토콜

## Core Insight
App Intents는 앱의 기능을 시스템이 이해할 수 있는 의도로 변환한다. 코드가 아니라 사용자의 언어로 앱을 정의하는 것이다.

"함수를 호출한다"와 "의도를 실행한다"의 차이는 무엇인가? 함수는 orderCoffee(size: .large, type: .latte)지만, 의도는 "큰 라떼 주문하기"다. 파라미터와 타입이 아니라 자연어로 표현 가능한 행동이다.

Siri, Shortcuts, Spotlight, Focus Mode - 이 모든 시스템 기능이 앱의 의도를 이해한다. AppIntent 프로토콜을 구현하면 앱이 OS의 일부가 된다. 더 이상 고립된 앱이 아니라 시스템과 대화하는 시민이 된다.

@Parameter는 단순한 입력값이 아니라 사용자가 제공할 수 있는 맥락이다. IntentParameter는 동적으로 옵션을 제공하고, 사용자의 선택을 학습한다. 시스템이 사용 패턴을 이해하고 제안을 개선한다.

## Connections
→ [[017-intent-donation-pattern]]
→ [[018-shortcuts-integration]]
← [[003-user-intention-architecture]]

---
Level: L6
Date: 2025-08-15
Tags: #ios #app-intents #siri #shortcuts