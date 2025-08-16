# Push Notifications: 주의력 경제의 도구

## Core Insight
Push notification은 사용자의 주의력을 구매하는 화폐다. 올바르게 사용하면 가치를 더하고, 남용하면 신뢰를 파괴한다.

```swift
// 권한 요청은 신중해야 한다
UNUserNotificationCenter.current().requestAuthorization(
    options: [.alert, .badge, .sound]
) { granted, error in
    if granted {
        DispatchQueue.main.async {
            UIApplication.shared.registerForRemoteNotifications()
        }
    }
}
```

Push notification의 복잡한 여정:
1. **앱 → APNs**: 토큰 등록
2. **서버 → APNs**: 알림 발송
3. **APNs → 기기**: 전달
4. **시스템 → 사용자**: 표시

가장 중요한 것은 **사용자 경험의 맥락**이다:
- **Timely**: 적절한 시간에
- **Relevant**: 관련성 있는 내용을
- **Actionable**: 행동할 수 있는 것을

Rich notifications의 힘:
```swift
// 단순한 텍스트를 넘어선 풍부한 경험
{
    "aps": {
        "alert": {
            "title": "새 메시지",
            "body": "김철수님이 사진을 보냈습니다"
        },
        "mutable-content": 1
    },
    "attachment-url": "https://example.com/image.jpg"
}
```

Notification Service Extension으로 이미지, 비디오까지 표시 가능하다.

사용자는 알림을 **신뢰의 척도**로 본다:
- 유용한 알림 = 앱을 계속 사용
- 스팸성 알림 = 앱을 삭제

Analytics가 중요한 이유: 열람률, 액션률, 해제률을 모니터링해야 한다.

## Connections
→ [[221-notification-service-extension]]
→ [[222-silent-notifications]]
→ [[223-notification-analytics]]
← [[130-app-lifecycle-states]]

---
Level: L3
Date: 2025-08-16
Tags: #ios #push-notifications #apns #user-attention #engagement