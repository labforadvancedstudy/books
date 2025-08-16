# A/B 테스트

## 개념
VoIP 앱의 기능, UI, 성능 개선을 위한 사용자 그룹 분할 실험과 결과 분석.

## A/B 테스트 프레임워크
```swift
import Firebase

class ABTestManager {
    static let shared = ABTestManager()
    private let remoteConfig = RemoteConfig.remoteConfig()
    
    enum Experiment: String, CaseIterable {
        case callUIDesign = "call_ui_design"
        case codecPriority = "codec_priority"
        case onboardingFlow = "onboarding_flow"
        case connectionTimeout = "connection_timeout"
        case jitterBufferSize = "jitter_buffer_size"
        
        var defaultValue: Any {
            switch self {
            case .callUIDesign: return "classic"
            case .codecPriority: return "opus_first"
            case .onboardingFlow: return "standard"
            case .connectionTimeout: return 10.0
            case .jitterBufferSize: return 50
            }
        }
    }
    
    func setupExperiments() {
        // 기본값 설정
        let defaults = Experiment.allCases.reduce(into: [String: Any]()) {
            $0[$1.rawValue] = $1.defaultValue
        }
        remoteConfig.setDefaults(defaults)
        
        // Remote Config 가져오기
        remoteConfig.fetch(withExpirationDuration: 3600) { status, error in
            if status == .success {
                self.remoteConfig.activate { _, _ in
                    self.applyExperiments()
                }
            }
        }
    }
}
```

## UI A/B 테스트
```swift
class CallUIExperiment {
    enum Variant: String {
        case classic = "classic"
        case modern = "modern"
        case minimal = "minimal"
        
        func createCallViewController() -> UIViewController {
            switch self {
            case .classic:
                return ClassicCallViewController()
            case .modern:
                return ModernCallViewController()
            case .minimal:
                return MinimalCallViewController()
            }
        }
    }
    
    static func getVariant() -> Variant {
        let variantString = ABTestManager.shared.getValue(for: .callUIDesign)
        return Variant(rawValue: variantString) ?? .classic
    }
    
    static func trackInteraction(_ action: String) {
        Analytics.logEvent("call_ui_interaction", parameters: [
            "variant": getVariant().rawValue,
            "action": action,
            "timestamp": Date().timeIntervalSince1970
        ])
    }
}
```

## 성능 A/B 테스트
```swift
class PerformanceExperiment {
    struct CodecExperiment {
        enum Variant: String {
            case opusFirst = "opus_first"
            case g711First = "g711_first"
            case adaptive = "adaptive"
            
            var codecPriority: [AudioCodec] {
                switch self {
                case .opusFirst:
                    return [.opus, .g711, .g729]
                case .g711First:
                    return [.g711, .opus, .g729]
                case .adaptive:
                    // 네트워크 상태에 따라 동적 선택
                    return NetworkMonitor.isHighBandwidth ? 
                        [.opus, .g711] : [.g711, .opus]
                }
            }
        }
    }
    
    static func measureCodecPerformance() {
        let variant = getCurrentCodecVariant()
        let metrics = CallQualityMonitor.getCurrentMetrics()
        
        Analytics.logEvent("codec_performance", parameters: [
            "variant": variant.rawValue,
            "mos_score": metrics.mos,
            "packet_loss": metrics.packetLoss,
            "latency": metrics.latency,
            "bandwidth_used": metrics.bandwidthUsed
        ])
    }
}
```

## 온보딩 A/B 테스트
```swift
class OnboardingExperiment {
    enum Flow: String {
        case standard = "standard"      // 모든 단계
        case simplified = "simplified"  // 최소 단계
        case guided = "guided"          // 튜토리얼 포함
        
        var steps: [OnboardingStep] {
            switch self {
            case .standard:
                return [.welcome, .permissions, .account, .contacts, .testCall]
            case .simplified:
                return [.welcome, .permissions, .done]
            case .guided:
                return [.welcome, .tutorial, .permissions, .guidedTestCall, .done]
            }
        }
    }
    
    static func trackConversion(flow: Flow, completed: Bool) {
        let conversionRate = calculateConversionRate(for: flow)
        
        Analytics.logEvent("onboarding_conversion", parameters: [
            "flow": flow.rawValue,
            "completed": completed,
            "steps_completed": getCompletedStepsCount(),
            "time_spent": getTotalOnboardingTime(),
            "conversion_rate": conversionRate
        ])
    }
}
```

## 통계적 유의성 검증
```swift
class ABTestAnalyzer {
    struct ExperimentResult {
        let variant: String
        let sampleSize: Int
        let conversionRate: Double
        let averageValue: Double
        let standardDeviation: Double
        let confidence: Double
    }
    
    static func calculateSignificance(
        control: ExperimentResult,
        variant: ExperimentResult
    ) -> SignificanceResult {
        // Z-test for proportions
        let p1 = control.conversionRate
        let p2 = variant.conversionRate
        let n1 = Double(control.sampleSize)
        let n2 = Double(variant.sampleSize)
        
        let pooledProportion = (p1 * n1 + p2 * n2) / (n1 + n2)
        let standardError = sqrt(pooledProportion * (1 - pooledProportion) * (1/n1 + 1/n2))
        let zScore = (p2 - p1) / standardError
        
        // p-value 계산
        let pValue = 2 * (1 - normalCDF(abs(zScore)))
        
        return SignificanceResult(
            isSignificant: pValue < 0.05,
            pValue: pValue,
            confidenceInterval: calculateConfidenceInterval(p1, p2, standardError),
            recommendedAction: pValue < 0.05 ? .adoptVariant : .continueTest
        )
    }
}
```

## 실시간 실험 모니터링
```swift
class ExperimentMonitor {
    private var activeExperiments: [String: ExperimentStatus] = [:]
    
    func monitorExperiment(_ name: String) {
        Timer.scheduledTimer(withTimeInterval: 3600, repeats: true) { _ in
            self.updateExperimentMetrics(name)
        }
    }
    
    private func updateExperimentMetrics(_ name: String) {
        let metrics = gatherExperimentMetrics(name)
        
        // 조기 종료 조건 확인
        if shouldStopExperiment(metrics) {
            stopExperiment(name, reason: .statisticalSignificance)
        }
        
        // 대시보드 업데이트
        publishToDashboard(name, metrics: metrics)
    }
    
    private func shouldStopExperiment(_ metrics: ExperimentMetrics) -> Bool {
        // 통계적 유의성 달성
        if metrics.pValue < 0.01 && metrics.sampleSize > 1000 {
            return true
        }
        
        // 부정적 영향 감지
        if metrics.variantCrashRate > metrics.controlCrashRate * 1.5 {
            return true
        }
        
        return false
    }
}
```

## 연관 개녕
- [[analytics_integration]]
- [[testflight_deployment]]
- [[subscription_model]]

## 태그
#abtesting #experiments #analytics #optimization #statistics