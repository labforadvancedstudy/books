# 애널리틱스 통합

## 개녕
VoIP 앱의 사용자 행동, 성능 메트릭, 비즈니스 KPI를 추적하여 데이터 기반 의사결정.

## 애널리틱스 플랫폼 통합
```swift
import Firebase
import Amplitude
import Mixpanel

protocol AnalyticsService {
    func track(event: String, properties: [String: Any]?)
    func setUserProperty(_ property: String, value: Any)
    func startSession()
    func endSession()
}

class UnifiedAnalytics: AnalyticsService {
    private let firebase = Analytics.self
    private let amplitude = Amplitude.instance()
    private let mixpanel = Mixpanel.mainInstance()
    
    func track(event: String, properties: [String: Any]?) {
        // Firebase
        firebase.logEvent(event, parameters: properties)
        
        // Amplitude
        amplitude.logEvent(event, withEventProperties: properties)
        
        // Mixpanel
        mixpanel.track(event: event, properties: properties as? Properties)
        
        // 로컬 로그 (디버그)
        #if DEBUG
        print("Analytics Event: \(event) - \(properties ?? [:])")
        #endif
    }
}
```

## VoIP 특화 이벤트
```swift
enum VoIPAnalyticsEvent: String {
    // 통화 이벤트
    case callInitiated = "call_initiated"
    case callConnected = "call_connected"
    case callEnded = "call_ended"
    case callFailed = "call_failed"
    case callMissed = "call_missed"
    
    // 품질 이벤트
    case poorQualityDetected = "poor_quality_detected"
    case networkSwitched = "network_switched"
    case codecChanged = "codec_changed"
    
    // 사용자 액션
    case muteToggled = "mute_toggled"
    case speakerToggled = "speaker_toggled"
    case holdToggled = "hold_toggled"
    
    var defaultProperties: [String: Any] {
        switch self {
        case .callInitiated, .callConnected:
            return [
                "network_type": NetworkMonitor.currentType,
                "codec": AudioManager.currentCodec,
                "region": LocationManager.currentRegion
            ]
        case .callEnded:
            return [
                "duration": CallManager.lastCallDuration,
                "end_reason": CallManager.endReason,
                "quality_score": CallManager.averageMOS
            ]
        default:
            return [:]
        }
    }
}
```

## 퍼널 분석
```swift
class FunnelAnalytics {
    enum OnboardingStep: String {
        case appOpened = "onboarding_started"
        case permissionsRequested = "permissions_requested"
        case permissionsGranted = "permissions_granted"
        case accountCreated = "account_created"
        case firstCallAttempted = "first_call_attempted"
        case firstCallCompleted = "first_call_completed"
    }
    
    static func trackOnboardingStep(_ step: OnboardingStep, success: Bool) {
        analytics.track(event: step.rawValue, properties: [
            "success": success,
            "time_spent": getTimeSpentInStep(),
            "attempt_number": getAttemptNumber(for: step)
        ])
        
        // 전환율 계산
        if step == .firstCallCompleted && success {
            let conversionRate = calculateConversionRate()
            analytics.track(event: "onboarding_completed", properties: [
                "total_time": getTotalOnboardingTime(),
                "conversion_rate": conversionRate
            ])
        }
    }
}
```

## 성능 메트릭
```swift
class PerformanceAnalytics {
    struct CallPerformanceMetrics {
        let connectionTime: TimeInterval  // Time to connect
        let setupTime: TimeInterval       // Time to first audio
        let averageLatency: Double
        let averageJitter: Double
        let packetLossRate: Float
        let averageMOS: Float
        let networkHandovers: Int
        let codecSwitches: Int
    }
    
    static func trackCallPerformance(_ metrics: CallPerformanceMetrics) {
        // 성능 임계값 확인
        if metrics.connectionTime > 5.0 {
            analytics.track(event: "slow_connection", properties: [
                "connection_time": metrics.connectionTime,
                "network_type": NetworkMonitor.currentType
            ])
        }
        
        if metrics.averageMOS < 3.5 {
            analytics.track(event: "poor_call_quality", properties: [
                "mos_score": metrics.averageMOS,
                "packet_loss": metrics.packetLossRate,
                "jitter": metrics.averageJitter
            ])
        }
        
        // 집계 데이터
        analytics.track(event: "call_performance", properties: metrics.toDictionary())
    }
}
```

## 사용자 세그먼트
```swift
class UserSegmentation {
    enum UserSegment: String {
        case newUser = "new_user"           // < 7 days
        case activeUser = "active_user"     // > 3 calls/week
        case powerUser = "power_user"       // > 10 calls/week
        case churningUser = "churning_user" // No calls in 7 days
        case dormantUser = "dormant_user"   // No calls in 30 days
    }
    
    static func identifyUserSegment(userId: String) -> UserSegment {
        let accountAge = getAccountAge(userId)
        let weeklyCallCount = getWeeklyCallCount(userId)
        let daysSinceLastCall = getDaysSinceLastCall(userId)
        
        if accountAge < 7 {
            return .newUser
        } else if weeklyCallCount > 10 {
            return .powerUser
        } else if weeklyCallCount > 3 {
            return .activeUser
        } else if daysSinceLastCall > 30 {
            return .dormantUser
        } else if daysSinceLastCall > 7 {
            return .churningUser
        } else {
            return .activeUser
        }
    }
    
    static func updateUserSegment(_ segment: UserSegment) {
        analytics.setUserProperty("user_segment", value: segment.rawValue)
        
        // 세그먼트별 타겟팅
        switch segment {
        case .churningUser:
            triggerReengagementCampaign()
        case .powerUser:
            offerPremiumFeatures()
        default:
            break
        }
    }
}
```

## 실시간 대시보드
```swift
class RealtimeDashboard {
    private var websocket: URLSessionWebSocketTask?
    
    func connectToAnalyticsDashboard() {
        let url = URL(string: "wss://analytics.voipapp.com/realtime")!
        websocket = URLSession.shared.webSocketTask(with: url)
        websocket?.resume()
        
        // 실시간 메트릭 전송
        Timer.scheduledTimer(withTimeInterval: 5, repeats: true) { _ in
            self.sendRealtimeMetrics()
        }
    }
    
    private func sendRealtimeMetrics() {
        let metrics = [
            "active_calls": CallManager.activeCallCount,
            "total_users_online": UserManager.onlineUserCount,
            "average_mos": QualityMonitor.currentAverageMOS,
            "server_load": ServerMonitor.currentLoad,
            "timestamp": Date().timeIntervalSince1970
        ]
        
        let message = URLSessionWebSocketTask.Message.string(
            try! JSONSerialization.data(withJSONObject: metrics).base64EncodedString()
        )
        
        websocket?.send(message) { _ in }
    }
}
```

## 연관 개념
- [[usage_tracking]]
- [[ab_testing]]
- [[crash_reporting]]

## 태그
#analytics #metrics #tracking #dashboard #insights