# 통화 기록 저장 (Core Data vs SQLite)

## 개념
VoIP 앱에서 통화 내역, 연락처, 녹음 파일 경로 등을 지속적으로 저장하기 위한 데이터베이스 선택.

## Core Data 구현
```swift
// CallRecord.xcdatamodeld
@objc(CallRecord)
class CallRecord: NSManagedObject {
    @NSManaged var callId: UUID
    @NSManaged var remoteNumber: String
    @NSManaged var startTime: Date
    @NSManaged var duration: Int32
    @NSManaged var direction: String // incoming/outgoing
    @NSManaged var quality: Float // MOS score
}

// iCloud 동기화 지원
container.persistentStoreDescriptions.forEach {
    $0.setOption(true, forKey: NSPersistentHistoryTrackingKey)
    $0.cloudKitContainerOptions = NSPersistentCloudKitContainerOptions()
}
```

## SQLite 직접 구현
```swift
import SQLite3

class CallDatabase {
    private var db: OpaquePointer?
    
    func createTable() {
        let sql = """
            CREATE TABLE IF NOT EXISTS calls (
                id TEXT PRIMARY KEY,
                remote_number TEXT,
                start_time REAL,
                duration INTEGER,
                codec TEXT,
                packet_loss REAL
            )
        """
        sqlite3_exec(db, sql, nil, nil, nil)
    }
}
```

## 비교 분석
| 요소 | Core Data | SQLite |
|------|-----------|--------|
| 성능 | 보통 | 빠름 |
| 메모리 | 많음 | 적음 |
| iCloud | 지원 | 수동 구현 |
| 복잡도 | 높음 | 낮음 |

## VoIP 특화 스키마
```swift
// 필수 필드
struct CallSchema {
    let callId: UUID
    let startTime: Date
    let endTime: Date?
    let remoteParty: String
    let direction: CallDirection
    let connectionType: String // WiFi/Cellular/VPN
    let codec: String
    let averageMOS: Float
    let packetLoss: Float
    let jitter: Float
}
```

## 연관 개념
- [[local_caching_strategy]]
- [[mos_score]]
- [[analytics_integration]]

## 태그
#storage #database #coredata #sqlite #persistence