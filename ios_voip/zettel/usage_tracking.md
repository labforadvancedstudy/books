# 사용량 추적

## 개념
VoIP 앱 사용자의 통화 패턴, 데이터 사용량, 기능 사용 빈도 등을 추적하여 분석 및 최적화.

## 사용량 메트릭
```swift
struct UsageMetrics {
    // 통화 메트릭
    struct CallMetrics {
        let totalCalls: Int
        let totalDuration: TimeInterval
        let averageDuration: TimeInterval
        let successRate: Float
        let failureReasons: [String: Int]
        let peakHours: [Int: Int]  // Hour: Count
        let topDestinations: [(String, Int)]  // Country: Count
    }
    
    // 데이터 메트릭
    struct DataMetrics {
        let totalDataUsed: Int64  // Bytes
        let audioDataUsed: Int64
        let videoDataUsed: Int64
        let signalingDataUsed: Int64
        let averageBandwidth: Double  // Kbps
        let peakBandwidth: Double
    }
    
    // 품질 메트릭
    struct QualityMetrics {
        let averageMOS: Float
        let averageLatency: Double
        let averageJitter: Double
        let averagePacketLoss: Float
        let networkTypeDistribution: [String: Int]  // WiFi/Cellular/etc
    }
}
```

## 사용량 추적 엔진
```swift
class UsageTrackingEngine {
    private let analytics: AnalyticsService
    private let database: Database
    
    func trackCallStart(_ call: CallInfo) {
        // 실시간 추적
        analytics.track("call_started", properties: [
            "destination": call.destination,
            "network_type": getCurrentNetworkType(),
            "codec": call.codec,
            "timestamp": Date().timeIntervalSince1970
        ])
        
        // 데이터베이스 저장
        database.save(CallSession(
            id: call.id,
            startTime: Date(),
            destination: call.destination
        ))
    }
    
    func trackCallEnd(_ call: CallInfo, duration: TimeInterval) {
        // 통화 종료 메트릭
        let metrics = gatherCallMetrics(call)
        
        analytics.track("call_ended", properties: [
            "call_id": call.id,
            "duration": duration,
            "mos_score": metrics.mosScore,
            "packet_loss": metrics.packetLoss,
            "data_used": metrics.dataUsed,
            "end_reason": call.endReason
        ])
        
        // 누적 통계 업데이트
        updateCumulativeStats(duration: duration, metrics: metrics)
    }
}
```

## 데이터 사용량 모니터링
```swift
class DataUsageMonitor {
    private var sessionStartBytes: Int64 = 0
    private var sessionEndBytes: Int64 = 0
    
    func startMonitoring() {
        sessionStartBytes = getCurrentDataUsage()
        
        // 주기적 업데이트
        Timer.scheduledTimer(withTimeInterval: 10, repeats: true) { _ in
            self.updateDataUsage()
        }
    }
    
    private func getCurrentDataUsage() -> Int64 {
        var info = ifaddrs()
        var totalBytes: Int64 = 0
        
        guard getifaddrs(&info) == 0 else { return 0 }
        defer { freeifaddrs(info) }
        
        var cursor = info
        while cursor != nil {
            if let data = cursor?.pointee.ifa_data {
                let stats = data.assumingMemoryBound(to: if_data.self).pointee
                totalBytes += Int64(stats.ifi_ibytes + stats.ifi_obytes)
            }
            cursor = cursor?.pointee.ifa_next
        }
        
        return totalBytes
    }
    
    func getSessionDataUsage() -> DataUsage {
        sessionEndBytes = getCurrentDataUsage()
        let totalUsed = sessionEndBytes - sessionStartBytes
        
        return DataUsage(
            total: totalUsed,
            cellular: getCellularDataUsage(),
            wifi: getWiFiDataUsage(),
            timestamp: Date()
        )
    }
}
```

## 사용 패턴 분석
```swift
class UsagePatternAnalyzer {
    func analyzeCallPatterns(for userId: String) async -> CallPattern {
        let calls = await database.fetchCalls(for: userId, days: 30)
        
        // 시간대별 분석
        let hourlyDistribution = calls.reduce(into: [Int: Int]()) { result, call in
            let hour = Calendar.current.component(.hour, from: call.startTime)
            result[hour, default: 0] += 1
        }
        
        // 요일별 분석
        let weekdayDistribution = calls.reduce(into: [Int: Int]()) { result, call in
            let weekday = Calendar.current.component(.weekday, from: call.startTime)
            result[weekday, default: 0] += 1
        }
        
        // 주요 연락처
        let topContacts = Dictionary(grouping: calls, by: { $0.destination })
            .mapValues { $0.count }
            .sorted { $0.value > $1.value }
            .prefix(10)
        
        return CallPattern(
            peakHour: hourlyDistribution.max { $0.value < $1.value }?.key ?? 0,
            averageCallsPerDay: Float(calls.count) / 30,
            preferredDays: weekdayDistribution,
            topContacts: Array(topContacts)
        )
    }
}
```

## 사용량 제한 및 경고
```swift
class UsageLimitManager {
    struct UsageLimit {
        let dailyMinutes: Int?
        let monthlyMinutes: Int?
        let dailyDataMB: Int?
        let monthlyDataMB: Int?
    }
    
    func checkUsageLimits(current: UsageMetrics, limits: UsageLimit) -> [Warning] {
        var warnings: [Warning] = []
        
        // 일일 통화 시간 확인
        if let dailyLimit = limits.dailyMinutes {
            let todayMinutes = getTodayCallMinutes()
            if todayMinutes > Int(Float(dailyLimit) * 0.8) {
                warnings.append(.approachingDailyLimit(todayMinutes, dailyLimit))
            }
        }
        
        // 월간 데이터 사용량 확인
        if let monthlyDataLimit = limits.monthlyDataMB {
            let monthlyDataMB = getMonthlyDataUsageMB()
            if monthlyDataMB > Int(Float(monthlyDataLimit) * 0.9) {
                warnings.append(.approachingDataLimit(monthlyDataMB, monthlyDataLimit))
            }
        }
        
        return warnings
    }
    
    func enforceLimit() {
        // 제한 초과 시 통화 차단
        if hasExceededLimits() {
            blockOutgoingCalls()
            showLimitExceededAlert()
        }
    }
}
```

## 연관 개념
- [[call_billing]]
- [[analytics_integration]]
- [[call_history_storage]]

## 태그
#usage #tracking #analytics #metrics #monitoring