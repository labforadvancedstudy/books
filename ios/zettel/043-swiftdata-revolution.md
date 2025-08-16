# SwiftData: 모델이 곧 스키마

## Core Insight
SwiftData는 Core Data의 복잡함을 Swift의 단순함으로 대체한다. @Model 매크로 하나로 영속성이 시작된다.

Core Data는 강력하지만 복잡했다. NSManagedObject, NSPersistentContainer, .xcdatamodeld - 보일러플레이트의 늪이었다. SwiftData는 이 모든 것을 매크로 뒤로 숨긴다.

```swift
@Model
class Item {
    var name: String
    var createdAt: Date
}
```

이게 전부다. 마이그레이션, 관계, 쿼리 - 모든 것이 Swift 코드로 표현된다. 더 이상 XML 스키마 파일을 편집할 필요 없다.

ModelContext는 작업 공간이다. insert, delete, save - 명확한 동사로 데이터를 조작한다. @Query는 SwiftUI와 자연스럽게 통합된다. 데이터가 바뀌면 View가 자동으로 업데이트된다.

CloudKit 동기화가 투명해진다. 단순히 CloudKit을 활성화하면 끝이다. 충돌 해결, 동기화 상태 - 프레임워크가 처리한다.

## Connections
→ [[044-model-macro-magic]]
→ [[045-swiftdata-migration]]
← [[030-swiftui-state-management]]

---
Level: L3
Date: 2025-08-15
Tags: #ios #swiftdata #persistence #model