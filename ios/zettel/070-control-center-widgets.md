# Control Center Widgets: 시스템의 일부되기

## Core Insight
iOS 18의 Control Center Widget은 앱을 시스템 레벨로 승격시킨다. 앱이 아니라 기능이 된다.

ControlWidget은 즉각적인 행동이다. 토글, 버튼, 슬라이더 - 단 하나의 작업을 빠르게 수행한다. 앱을 열지 않고도 커피를 주문하고, 집 조명을 끄고, 운동을 시작한다.

```swift
struct CoffeeControlWidget: ControlWidget {
    var body: some ControlWidgetConfiguration {
        StaticControlConfiguration(
            kind: "com.app.coffee-order"
        ) {
            ControlWidgetButton(action: OrderCoffeeIntent()) {
                Label("Order Coffee", systemImage: "cup.and.saucer")
                Text("Your usual")
            }
        }
        .displayName("Quick Coffee")
        .description("Order your favorite coffee")
    }
}
```

Lock Screen에도 배치 가능하다. Face ID 없이도 접근 가능한 기능과 보안이 필요한 기능을 구분하라. isLuminanceReduced로 Always-On Display도 지원한다.

Action Button과의 연동이 자연스럽다. 물리 버튼이 앱의 기능을 직접 실행한다. 하드웨어와 소프트웨어의 경계가 흐려진다.

## Connections
→ [[071-ios18-customization]]
→ [[072-action-button-integration]]
← [[006-app-intents-philosophy]]

---
Level: L3
Date: 2025-08-15
Tags: #ios #ios18 #control-center #widgets