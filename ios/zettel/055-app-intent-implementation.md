# App Intent 구현 예제

## Core Insight
AppIntent는 사용자의 의도를 코드로 표현한다. Siri, Shortcuts, Spotlight가 이해할 수 있는 언어다.

```swift
struct OrderCoffeeIntent: AppIntent {
    static var title: LocalizedStringResource = "Order Coffee"
    static var description = IntentDescription("Order your favorite coffee")
    
    @Parameter(title: "Coffee Type")
    var coffeeType: CoffeeType
    
    @Parameter(title: "Size", default: .medium)
    var size: CoffeeSize
    
    static var parameterSummary: some ParameterSummary {
        Summary("Order \(\.$coffeeType) in \(\.$size) size")
    }
    
    func perform() async throws -> some IntentResult & ReturnsValue<String> {
        let order = try await CoffeeService.order(
            type: coffeeType,
            size: size
        )
        
        return .result(
            value: "Ordered \(coffeeType.name)! Ready in \(order.waitTime) minutes",
            dialog: "Your \(size.name) \(coffeeType.name) has been ordered."
        )
    }
}

// Entity for Siri understanding
struct CoffeeType: AppEntity {
    let id: String
    let name: String
    
    static var typeDisplayRepresentation = TypeDisplayRepresentation(name: "Coffee")
    
    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(title: "\(name)")
    }
    
    static var defaultQuery = CoffeeQuery()
}
```

Focus Filter, Control Center, Action Button - 모든 곳에서 작동한다.

## Connections
→ [[056-entity-query-pattern]]
→ [[057-shortcuts-donation]]
← [[006-app-intents-philosophy]]

---
Level: L0
Date: 2025-08-15
Tags: #ios #app-intents #siri #code