# Live Activity 구현 코드

## Core Insight
Live Activity는 ActivityAttributes로 정의하고, Activity.request로 시작한다. 푸시로 업데이트하거나 앱에서 직접 업데이트한다.

```swift
struct DeliveryAttributes: ActivityAttributes {
    struct ContentState: Codable, Hashable {
        var driverName: String
        var deliveryTime: Date
        var currentLocation: String
    }
    
    var orderNumber: String
}

// 시작
let attributes = DeliveryAttributes(orderNumber: "12345")
let initialState = ContentState(
    driverName: "김기사",
    deliveryTime: Date().addingTimeInterval(1800),
    currentLocation: "음식점에서 픽업 중"
)

let activity = try Activity.request(
    attributes: attributes,
    content: .init(state: initialState, staleDate: nil)
)

// 업데이트
await activity.update(using: newState)

// 종료
await activity.end(using: finalState, dismissalPolicy: .after(Date().addingTimeInterval(60)))
```

Dynamic Island와 Lock Screen UI는 별도로 정의한다. 각각 다른 공간 제약을 가진다.

## Connections
→ [[050-push-to-live-activity]]
→ [[051-dynamic-island-layouts]]
← [[013-live-activities-presence]]

---
Level: L0
Date: 2025-08-15
Tags: #ios #live-activity #activitykit #code