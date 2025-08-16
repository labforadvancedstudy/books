# 시간대 처리

## 개념
글로벌 VoIP 앱에서 다양한 시간대의 사용자 간 통화 기록, 예약, 표시를 정확하게 처리.

## 시간대 변환
```swift
class TimezoneManager {
    static let shared = TimezoneManager()
    
    // 현재 사용자 시간대
    var userTimezone: TimeZone {
        return TimeZone.current
    }
    
    // 서버 시간대 (UTC)
    let serverTimezone = TimeZone(identifier: "UTC")!
    
    // 로컬 시간을 UTC로 변환
    func convertToUTC(_ date: Date) -> Date {
        let seconds = TimeInterval(userTimezone.secondsFromGMT(for: date))
        return date.addingTimeInterval(-seconds)
    }
    
    // UTC를 로컬 시간으로 변환
    func convertToLocal(_ utcDate: Date) -> Date {
        let seconds = TimeInterval(userTimezone.secondsFromGMT(for: utcDate))
        return utcDate.addingTimeInterval(seconds)
    }
    
    // 특정 시간대로 변환
    func convert(_ date: Date, to timezone: TimeZone) -> Date {
        let sourceOffset = TimeInterval(userTimezone.secondsFromGMT(for: date))
        let targetOffset = TimeInterval(timezone.secondsFromGMT(for: date))
        return date.addingTimeInterval(targetOffset - sourceOffset)
    }
}
```

## 통화 기록 시간 표시
```swift
class CallHistoryTimeDisplay {
    static func formatCallTime(_ call: CallRecord) -> String {
        let formatter = DateFormatter()
        formatter.timeZone = TimeZone.current
        
        // 사용자 지역에 맞는 형식
        if Calendar.current.isDateInToday(call.startTime) {
            formatter.dateFormat = "HH:mm"  // 오늘: 14:30
            return formatter.string(from: call.startTime)
        } else if Calendar.current.isDateInYesterday(call.startTime) {
            return "Yesterday \(formatter.string(from: call.startTime))"
        } else {
            formatter.dateStyle = .medium
            formatter.timeStyle = .short
            return formatter.string(from: call.startTime)
        }
    }
    
    static func formatCallWithTimezone(_ call: CallRecord, showTimezone: Bool = false) -> String {
        let formatter = DateFormatter()
        formatter.dateStyle = .long
        formatter.timeStyle = .medium
        
        if showTimezone {
            formatter.timeZone = TimeZone(identifier: call.remoteTimezone ?? "UTC")
            formatter.dateFormat = "MMM d, yyyy 'at' HH:mm zzz"
        }
        
        return formatter.string(from: call.startTime)
    }
}
```

## 국제 통화 예약
```swift
class InternationalCallScheduler {
    struct ScheduledCall {
        let localTime: Date
        let remoteTime: Date
        let remoteTimezone: TimeZone
        let participantNumber: String
    }
    
    static func scheduleCall(
        at localTime: Date,
        with remoteNumber: String,
        remoteTimezone: TimeZone
    ) -> ScheduledCall {
        // 상대방 시간 계산
        let timezoneManager = TimezoneManager.shared
        let remoteTime = timezoneManager.convert(localTime, to: remoteTimezone)
        
        // 적절한 시간대 확인 (업무 시간)
        if !isBusinessHours(remoteTime, in: remoteTimezone) {
            showTimezoneWarning(remoteTime: remoteTime, timezone: remoteTimezone)
        }
        
        return ScheduledCall(
            localTime: localTime,
            remoteTime: remoteTime,
            remoteTimezone: remoteTimezone,
            participantNumber: remoteNumber
        )
    }
    
    static func isBusinessHours(_ date: Date, in timezone: TimeZone) -> Bool {
        var calendar = Calendar.current
        calendar.timeZone = timezone
        
        let hour = calendar.component(.hour, from: date)
        let weekday = calendar.component(.weekday, from: date)
        
        // 주말 체크 (1 = 일요일, 7 = 토요일)
        if weekday == 1 || weekday == 7 {
            return false
        }
        
        // 업무 시간 (9 AM - 6 PM)
        return hour >= 9 && hour < 18
    }
}
```

## 서머타임 처리
```swift
class DaylightSavingHandler {
    static func handleDSTTransition() {
        // DST 전환 감지
        NotificationCenter.default.addObserver(
            forName: .NSSystemTimeZoneDidChange,
            object: nil,
            queue: .main
        ) { _ in
            self.updateScheduledCalls()
        }
    }
    
    static func updateScheduledCalls() {
        // 모든 예약된 통화 시간 재계산
        let scheduledCalls = fetchScheduledCalls()
        
        for call in scheduledCalls {
            let originalTime = call.scheduledTime
            let adjustedTime = adjustForDST(originalTime)
            
            if originalTime != adjustedTime {
                // 사용자에게 알림
                notifyDSTChange(
                    from: originalTime,
                    to: adjustedTime,
                    callId: call.id
                )
                
                // 데이터베이스 업데이트
                updateScheduledCall(call.id, newTime: adjustedTime)
            }
        }
    }
    
    static func adjustForDST(_ date: Date) -> Date {
        let timezone = TimeZone.current
        let dstOffset = timezone.daylightSavingTimeOffset(for: date)
        return date.addingTimeInterval(dstOffset)
    }
}
```

## 세계 시각 표시
```swift
class WorldClockView: UIView {
    struct CityTime {
        let city: String
        let timezone: TimeZone
        let offset: String
        
        var currentTime: String {
            let formatter = DateFormatter()
            formatter.timeZone = timezone
            formatter.dateFormat = "HH:mm"
            return formatter.string(from: Date())
        }
    }
    
    let cities = [
        CityTime(city: "New York", timezone: TimeZone(identifier: "America/New_York")!, offset: "EST"),
        CityTime(city: "London", timezone: TimeZone(identifier: "Europe/London")!, offset: "GMT"),
        CityTime(city: "Tokyo", timezone: TimeZone(identifier: "Asia/Tokyo")!, offset: "JST"),
        CityTime(city: "Sydney", timezone: TimeZone(identifier: "Australia/Sydney")!, offset: "AEDT")
    ]
    
    func updateClocks() {
        for (index, city) in cities.enumerated() {
            let label = viewWithTag(100 + index) as? UILabel
            label?.text = "\(city.city): \(city.currentTime) \(city.offset)"
            
            // 업무 시간 표시
            let isBusinessHours = InternationalCallScheduler.isBusinessHours(
                Date(),
                in: city.timezone
            )
            label?.textColor = isBusinessHours ? .systemGreen : .systemGray
        }
    }
}
```

## 통화 기록 시간대 저장
```swift
struct CallRecordWithTimezone: Codable {
    let id: UUID
    let startTimeUTC: Date      // 항상 UTC로 저장
    let endTimeUTC: Date
    let localTimezone: String    // 사용자 시간대
    let remoteTimezone: String?  // 상대방 시간대
    
    // 표시용 계산 프로퍼티
    var localStartTime: Date {
        let timezone = TimeZone(identifier: localTimezone) ?? .current
        return TimezoneManager.shared.convert(startTimeUTC, to: timezone)
    }
    
    var duration: TimeInterval {
        return endTimeUTC.timeIntervalSince(startTimeUTC)
    }
}
```

## 연관 개녕
- [[localization_i18n]]
- [[country_regulations]]
- [[call_history_storage]]

## 태그
#timezone #datetime #dst #internationalization #scheduling