# App Extensions: 앱 경계의 확장

## Core Insight
App Extension은 앱의 기능을 다른 앱이나 시스템으로 확장하는 메커니즘이다. 앱의 영향력을 전체 시스템으로 넓힌다.

```swift
// Today Widget - 가장 간단한 Extension
import WidgetKit
import SwiftUI

struct SimpleWidget: Widget {
    let kind: String = "SimpleWidget"
    
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: Provider()) { entry in
            SimpleWidgetEntryView(entry: entry)
        }
        .configurationDisplayName("My Widget")
        .description("This is an example widget.")
    }
}
```

Extension의 종류와 용도:
1. **Today Extension**: 알림 센터에 위젯 표시
2. **Share Extension**: 다른 앱에서 콘텐츠 공유받기
3. **Action Extension**: 다른 앱의 콘텐츠에 액션 제공
4. **Photo Editing Extension**: 사진 앱에서 편집 기능 제공
5. **Custom Keyboard**: 시스템 키보드 대체
6. **Notification Service Extension**: 푸시 알림 처리

가장 중요한 제약사항: **메모리와 실행시간 제한**
- 메모리: 16-120MB (기기에 따라)
- 실행시간: 30초 (백그라운드에서)
- 네트워크: 제한된 백그라운드 액세스

Host App과의 데이터 공유:
```swift
// App Groups로 데이터 공유
let userDefaults = UserDefaults(suiteName: "group.com.company.myapp")
userDefaults?.set("shared data", forKey: "key")

// 공유 Core Data 스택
let container = NSPersistentCloudKitContainer(name: "DataModel")
container.persistentStoreDescriptions.first?.url = 
    FileManager.default.containerURL(forSecurityApplicationGroupIdentifier: "group.com.company.myapp")
```

Extension은 **사용자 경험의 확장**이다. 사용자가 메인 앱을 열지 않고도 핵심 기능을 사용할 수 있게 한다.

성공 사례: 1Password의 AutoFill Extension - 다른 앱에서도 비밀번호 자동 입력.

## Connections
→ [[251-share-extension-implementation]]
→ [[252-app-groups-data-sharing]]
→ [[253-extension-lifecycle]]
← [[040-app-clips-ephemeral]]

---
Level: L4
Date: 2025-08-16
Tags: #ios #extensions #widgets #system-integration #sharing