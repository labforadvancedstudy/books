# 크래시 리포팅 (Firebase Crashlytics)

## 개녕
VoIP 앱의 실시간 크래시 및 오류를 추적하고 분석. 통화 중 크래시는 사용자 경험에 치명적.

## Firebase Crashlytics 통합
```swift
import Firebase
import FirebaseCrashlytics

class CrashReporter {
    static func configure() {
        FirebaseApp.configure()
        
        // 디버그 빌드에서는 비활성화
        #if DEBUG
        Crashlytics.crashlytics().setCrashlyticsCollectionEnabled(false)
        #else
        Crashlytics.crashlytics().setCrashlyticsCollectionEnabled(true)
        #endif
    }
    
    // 사용자 정보 설정
    static func setUser(_ userId: String) {
        Crashlytics.crashlytics().setUserID(userId)
    }
    
    // 커스텀 키 설정
    static func setCustomValue(_ value: Any, forKey key: String) {
        Crashlytics.crashlytics().setCustomValue(value, forKey: key)
    }
}
```

## VoIP 특화 크래시 추적
```swift
extension CrashReporter {
    // 통화 상태 기록
    static func logCallState(_ state: CallState) {
        Crashlytics.crashlytics().setCustomValue(
            state.rawValue,
            forKey: "call_state"
        )
        
        Crashlytics.crashlytics().log("Call state changed to: \(state)")
    }
    
    // 네트워크 품질 기록
    static func logNetworkQuality(mos: Float, packetLoss: Float) {
        Crashlytics.crashlytics().setCustomValue(mos, forKey: "mos_score")
        Crashlytics.crashlytics().setCustomValue(packetLoss, forKey: "packet_loss")
    }
    
    // WebRTC 상태 기록
    static func logWebRTCState(
        iceState: RTCIceConnectionState,
        signalingState: RTCSignalingState
    ) {
        let iceStateString = iceStateToString(iceState)
        let signalingStateString = signalingStateToString(signalingState)
        
        Crashlytics.crashlytics().setCustomValue(
            iceStateString,
            forKey: "ice_connection_state"
        )
        Crashlytics.crashlytics().setCustomValue(
            signalingStateString,
            forKey: "signaling_state"
        )
    }
}
```

## Non-Fatal 오류 기록
```swift
class ErrorReporter {
    enum VoIPError: Error {
        case callKitActivationFailed
        case audioSessionConfigurationFailed
        case webRTCConnectionFailed
        case codecNegotiationFailed
        case stunServerUnreachable
        case turnAuthenticationFailed
        
        var userInfo: [String: Any] {
            switch self {
            case .callKitActivationFailed:
                return ["error_type": "callkit", "severity": "high"]
            case .audioSessionConfigurationFailed:
                return ["error_type": "audio", "severity": "critical"]
            case .webRTCConnectionFailed:
                return ["error_type": "webrtc", "severity": "high"]
            case .codecNegotiationFailed:
                return ["error_type": "codec", "severity": "medium"]
            case .stunServerUnreachable:
                return ["error_type": "stun", "severity": "low"]
            case .turnAuthenticationFailed:
                return ["error_type": "turn", "severity": "medium"]
            }
        }
    }
    
    static func recordError(_ error: VoIPError, userInfo: [String: Any]? = nil) {
        var info = error.userInfo
        if let userInfo = userInfo {
            info.merge(userInfo) { _, new in new }
        }
        
        // Non-fatal로 기록
        Crashlytics.crashlytics().record(
            error: error,
            userInfo: info
        )
    }
}
```

## 크래시 비드렅 및 보고서
```swift
class CrashBreadcrumbs {
    private static var breadcrumbs: [String] = []
    private static let maxBreadcrumbs = 100
    
    static func addBreadcrumb(_ message: String) {
        let timestamp = DateFormatter.localizedString(
            from: Date(),
            dateStyle: .none,
            timeStyle: .medium
        )
        
        let breadcrumb = "[\(timestamp)] \(message)"
        breadcrumbs.append(breadcrumb)
        
        // 최대 개수 유지
        if breadcrumbs.count > maxBreadcrumbs {
            breadcrumbs.removeFirst()
        }
        
        // Crashlytics에 기록
        Crashlytics.crashlytics().log(breadcrumb)
    }
    
    static func logCallFlow() {
        addBreadcrumb("Call initiated")
        addBreadcrumb("ICE gathering started")
        addBreadcrumb("STUN binding request sent")
        addBreadcrumb("ICE candidates exchanged")
        addBreadcrumb("DTLS handshake started")
        addBreadcrumb("Media streams established")
    }
}
```

## 크래시 예방
```swift
class CrashPrevention {
    // 안전한 API 호출
    static func safeCallKitAction(_ action: () throws -> Void) {
        do {
            try action()
        } catch {
            // 크래시 대신 오류 기록
            ErrorReporter.recordError(
                .callKitActivationFailed,
                userInfo: ["error": error.localizedDescription]
            )
            
            // Fallback 처리
            handleCallKitFailure()
        }
    }
    
    // 메모리 경고 처리
    static func handleMemoryWarning() {
        CrashBreadcrumbs.addBreadcrumb("Memory warning received")
        
        // 비필수 리소스 해제
        releaseNonEssentialResources()
        
        // 상태 기록
        Crashlytics.crashlytics().setCustomValue(
            true,
            forKey: "memory_warning_active"
        )
    }
}
```

## 테스트 크래시
```swift
#if DEBUG
extension CrashReporter {
    static func testCrash() {
        // 테스트 크래시 발생
        Crashlytics.crashlytics().log("Test crash triggered")
        fatalError("Test crash for Crashlytics")
    }
    
    static func testNonFatal() {
        // Non-fatal 테스트
        let error = NSError(
            domain: "com.voipapp.test",
            code: -1,
            userInfo: [
                NSLocalizedDescriptionKey: "Test non-fatal error"
            ]
        )
        
        Crashlytics.crashlytics().record(error: error)
    }
}
#endif
```

## 연관 개념
- [[memory_leak_detection]]
- [[analytics_integration]]
- [[testflight_deployment]]

## 태그
#crash #reporting #firebase #crashlytics #debugging