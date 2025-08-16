# Widget 구현 예제

## Core Insight
실제 Widget 코드는 단순하다. 복잡함은 TimelineProvider에 있고, View는 순수하다.

```swift
struct SimpleEntry: TimelineEntry {
    let date: Date
    let emoji: String
}

struct Provider: TimelineProvider {
    func placeholder(in context: Context) -> SimpleEntry {
        SimpleEntry(date: Date(), emoji: "🎯")
    }
    
    func getSnapshot(in context: Context, 
                    completion: @escaping (SimpleEntry) -> ()) {
        completion(SimpleEntry(date: Date(), emoji: "🎯"))
    }
    
    func getTimeline(in context: Context,
                    completion: @escaping (Timeline<SimpleEntry>) -> ()) {
        let entries = [SimpleEntry(date: Date(), emoji: "🎯")]
        let timeline = Timeline(entries: entries, 
                               policy: .after(Date().addingTimeInterval(3600)))
        completion(timeline)
    }
}
```

View는 Entry를 받아서 표시만 한다. 상태 관리 없음, 애니메이션 없음, 그저 현재를 표현한다.

## Connections
→ [[047-widget-deep-link-code]]
→ [[048-widget-intent-configuration]]
← [[011-timeline-provider-pattern]]

---
Level: L0
Date: 2025-08-15
Tags: #ios #widget #code #implementation