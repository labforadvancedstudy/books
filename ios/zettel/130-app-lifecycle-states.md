# App Lifecycle: 생명의 주기

## Core Insight
앱의 생명주기는 사용자의 주의력 주기와 일치한다. 시스템은 리소스 관리자이자 경험의 연출자다.

```swift
// SwiftUI에서 생명주기 관찰
struct ContentView: View {
    var body: some View {
        Text("Hello")
            .onReceive(NotificationCenter.default.publisher(for: UIApplication.didEnterBackgroundNotification)) { _ in
                // 백그라운드 진입 - 무거운 작업 중단
            }
            .onReceive(NotificationCenter.default.publisher(for: UIApplication.willEnterForegroundNotification)) { _ in
                // 포그라운드 복귀 - 데이터 새로고침
            }
    }
}
```

앱의 5가지 상태:
1. **Not Running**: 시작되지 않음 또는 종료됨
2. **Inactive**: 실행 중이지만 이벤트 받지 않음 (전화 수신 시)
3. **Active**: 정상 실행 중
4. **Background**: 백그라운드에서 실행 중 (제한된 시간)
5. **Suspended**: 메모리에 있지만 실행되지 않음

가장 중요한 전환점들:
- **Active → Background**: 사용자가 홈 버튼을 누르거나 다른 앱으로 전환
- **Background → Suspended**: 시스템이 앱을 일시정지 (보통 3-10초 후)
- **Suspended → Not Running**: 메모리 부족 시 시스템이 강제 종료

백그라운드에서 할 수 있는 일은 극히 제한적이다:
- 음악 재생
- 위치 업데이트
- VoIP 호출
- 뉴스스탠드 다운로드
- 외부 액세서리 통신

현실: 대부분의 앱은 백그라운드에서 아무것도 할 수 없다.

## Connections
→ [[131-background-processing-limits]]
→ [[132-state-restoration-pattern]]
→ [[133-scene-lifecycle-ios13]]
← [[001-ios-app-essence]]

---
Level: L2
Date: 2025-08-16
Tags: #ios #lifecycle #background #system #resource-management