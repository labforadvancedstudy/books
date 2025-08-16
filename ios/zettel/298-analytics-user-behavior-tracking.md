# 사용자 행동 분석: 데이터로 읽는 사용자 마음

## Core Insight
사용자 분석은 추측을 사실로 바꾸는 과정이며, 올바른 메트릭 선택과 해석이 제품 개선의 핵심이다.

데이터는 거짓말하지 않는다. 사용자들이 실제로 어떻게 앱을 사용하는지, 어디서 이탈하는지, 무엇을 가장 좋아하는지를 객관적으로 보여준다. 하지만 데이터를 올바르게 수집하고 해석하는 것은 별개의 기술이다.

**핵심 메트릭 체계:**

**AARRR 프레임워크 (해적 메트릭):**
```swift
enum PirateMetrics: String, CaseIterable {
    case acquisition = "획득"      // 사용자가 어떻게 앱을 발견했는가?
    case activation = "활성화"     // 첫 경험이 얼마나 좋았는가?
    case retention = "유지"        // 사용자가 계속 돌아오는가?
    case revenue = "수익"          // 돈을 지불할 의향이 있는가?
    case referral = "추천"         // 다른 사람에게 추천하는가?
}

struct MetricTracker {
    // 획득 메트릭
    func trackUserAcquisition(source: String, medium: String, campaign: String) {
        Analytics.track("user_acquired", parameters: [
            "source": source,           // App Store, Google, Social Media
            "medium": medium,           // organic, paid, referral
            "campaign": campaign,       // specific campaign name
            "install_date": Date()
        ])
    }
    
    // 활성화 메트릭
    func trackActivation(event: ActivationEvent) {
        Analytics.track("user_activated", parameters: [
            "event": event.rawValue,
            "time_to_activation": calculateTimeToActivation(),
            "steps_completed": getOnboardingStepsCompleted()
        ])
    }
}
```

**코호트 분석 구현:**
```swift
struct CohortAnalysis {
    struct Cohort {
        let installDate: Date
        let userCount: Int
        let retentionRates: [Int: Double] // Day -> Retention Rate
    }
    
    func calculateRetention(cohort: Cohort, day: Int) -> Double {
        // Day 0 = 100% (설치일)
        // Day 1 = 다음날 돌아온 사용자 비율
        // Day 7 = 일주일 후 돌아온 사용자 비율
        // Day 30 = 한달 후 돌아온 사용자 비율
        
        let activeUsersOnDay = getUsersActiveOnDay(cohort: cohort, day: day)
        return Double(activeUsersOnDay) / Double(cohort.userCount)
    }
    
    private func getUsersActiveOnDay(cohort: Cohort, day: Int) -> Int {
        // 실제 데이터베이스 쿼리 또는 분석 서비스 호출
        return 0 // placeholder
    }
}
```

**사용자 세그멘테이션:**
```swift
enum UserSegment: String {
    case newUser = "new_user"           // 0-7일
    case activeUser = "active_user"     // 8-30일, 주간 활성
    case powerUser = "power_user"       // 고빈도 사용자
    case dormantUser = "dormant_user"   // 30일+ 비활성
    case churnedUser = "churned_user"   // 90일+ 비활성
}

class UserSegmentation {
    func classifyUser(_ userID: String) -> UserSegment {
        let lastActiveDate = getLastActiveDate(userID)
        let sessionCount = getSessionCount(userID, days: 30)
        let daysSinceLastActive = Calendar.current.dateComponents([.day], 
                                                                from: lastActiveDate, 
                                                                to: Date()).day ?? 0
        
        switch (daysSinceLastActive, sessionCount) {
        case (0...7, _):
            return .newUser
        case (8...30, 20...):
            return .powerUser
        case (8...30, 1...19):
            return .activeUser
        case (31...90, _):
            return .dormantUser
        default:
            return .churnedUser
        }
    }
}
```

**이벤트 추적 아키텍처:**
```swift
protocol AnalyticsEvent {
    var name: String { get }
    var parameters: [String: Any] { get }
    var userProperties: [String: Any] { get }
}

struct UserAction: AnalyticsEvent {
    let name: String
    let parameters: [String: Any]
    let userProperties: [String: Any]
    
    init(action: String, screen: String, element: String? = nil) {
        self.name = "user_action"
        self.parameters = [
            "action": action,
            "screen": screen,
            "element": element ?? "",
            "timestamp": Date().timeIntervalSince1970
        ]
        self.userProperties = [:]
    }
}

// 사용 예시
extension UIButton {
    @objc func trackButtonTap() {
        let event = UserAction(
            action: "button_tap",
            screen: getCurrentScreenName(),
            element: self.accessibilityIdentifier
        )
        Analytics.track(event)
    }
}
```

**퍼널 분석:**
```swift
struct ConversionFunnel {
    enum OnboardingStep: String, CaseIterable {
        case appOpen = "app_open"
        case welcomeScreen = "welcome_screen"
        case signUp = "sign_up"
        case profileSetup = "profile_setup"
        case firstAction = "first_action"
        case activation = "activation"
    }
    
    func calculateConversionRates() -> [OnboardingStep: Double] {
        var conversions: [OnboardingStep: Double] = [:]
        let allSteps = OnboardingStep.allCases
        
        for (index, step) in allSteps.enumerated() {
            if index == 0 {
                conversions[step] = 1.0 // 100% start at first step
            } else {
                let previousStep = allSteps[index - 1]
                let currentStepUsers = getUsersCompletingStep(step)
                let previousStepUsers = getUsersCompletingStep(previousStep)
                
                conversions[step] = Double(currentStepUsers) / Double(previousStepUsers)
            }
        }
        
        return conversions
    }
    
    func identifyDropOffPoints() -> [OnboardingStep] {
        let conversions = calculateConversionRates()
        return conversions.compactMap { step, rate in
            rate < 0.5 ? step : nil // 50% 미만 전환율을 문제점으로 식별
        }
    }
}
```

**A/B 테스트 프레임워크:**
```swift
class ABTestManager {
    enum TestVariant: String {
        case control = "control"
        case variantA = "variant_a"
        case variantB = "variant_b"
    }
    
    struct ABTest {
        let name: String
        let variants: [TestVariant]
        let trafficAllocation: [TestVariant: Double]
        let startDate: Date
        let endDate: Date
    }
    
    func getVariantForUser(_ userID: String, test: ABTest) -> TestVariant {
        // 일관된 사용자 할당을 위한 해시 기반 분배
        let hash = userID.hashValue
        let normalizedHash = abs(hash) % 100
        
        var cumulativePercentage = 0.0
        for (variant, percentage) in test.trafficAllocation {
            cumulativePercentage += percentage * 100
            if Double(normalizedHash) < cumulativePercentage {
                return variant
            }
        }
        
        return .control
    }
    
    func trackTestExposure(userID: String, test: ABTest, variant: TestVariant) {
        Analytics.track("ab_test_exposure", parameters: [
            "test_name": test.name,
            "variant": variant.rawValue,
            "user_id": userID
        ])
    }
}
```

**실시간 대시보드 구현:**
```swift
class RealTimeDashboard: ObservableObject {
    @Published var activeUsers: Int = 0
    @Published var sessionDuration: TimeInterval = 0
    @Published var conversionRate: Double = 0.0
    @Published var topEvents: [(String, Int)] = []
    
    private let timer = Timer.publish(every: 60, on: .main, in: .common)
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        timer
            .autoconnect()
            .sink { [weak self] _ in
                self?.updateMetrics()
            }
            .store(in: &cancellables)
    }
    
    private func updateMetrics() {
        // 실시간 메트릭 업데이트
        Task {
            let metrics = await fetchRealTimeMetrics()
            await MainActor.run {
                self.activeUsers = metrics.activeUsers
                self.sessionDuration = metrics.avgSessionDuration
                self.conversionRate = metrics.conversionRate
                self.topEvents = metrics.topEvents
            }
        }
    }
}
```

**사용자 여정 맵핑:**
```swift
struct UserJourney {
    struct TouchPoint {
        let screen: String
        let action: String
        let timestamp: Date
        let sessionID: String
    }
    
    func analyzeUserPath(userID: String, days: Int = 30) -> [TouchPoint] {
        // 사용자의 앱 내 경로 추적
        return getUserEvents(userID: userID, days: days)
            .map { event in
                TouchPoint(
                    screen: event.screen,
                    action: event.action,
                    timestamp: event.timestamp,
                    sessionID: event.sessionID
                )
            }
            .sorted { $0.timestamp < $1.timestamp }
    }
    
    func findCommonPaths() -> [String: Int] {
        // 가장 일반적인 사용자 경로 식별
        var pathCounts: [String: Int] = [:]
        
        // 모든 사용자의 경로를 분석하여 패턴 식별
        // 예: "HomeScreen -> SearchScreen -> ProductDetail -> Purchase"
        
        return pathCounts
    }
}
```

**개인정보 보호 친화적 분석:**
```swift
class PrivacyFriendlyAnalytics {
    // 개인 식별 정보 없이 패턴 분석
    func trackAnonymousEvent(_ event: String, properties: [String: Any]) {
        // 사용자 ID 해싱
        let hashedUserID = hashUserID(getCurrentUserID())
        
        // 민감한 데이터 제거
        let sanitizedProperties = sanitizeProperties(properties)
        
        // 로컬에서 일정 기간 집계 후 전송
        bufferEvent(event: event, 
                   hashedUserID: hashedUserID,
                   properties: sanitizedProperties)
    }
    
    private func hashUserID(_ userID: String) -> String {
        // 복구 불가능한 해시 생성
        return userID.sha256
    }
    
    private func sanitizeProperties(_ properties: [String: Any]) -> [String: Any] {
        // PII 제거 및 카테고리화
        return properties.compactMapValues { value in
            if let stringValue = value as? String {
                return stringValue.contains("@") ? "email_provided" : stringValue
            }
            return value
        }
    }
}
```

**성과 측정 KPI:**
```swift
struct KPIDashboard {
    // 제품 KPI
    let dailyActiveUsers: Int
    let monthlyActiveUsers: Int
    let averageSessionDuration: TimeInterval
    let screenViews: Int
    
    // 비즈니스 KPI
    let conversionRate: Double
    let averageRevenuePerUser: Double
    let customerLifetimeValue: Double
    let churnRate: Double
    
    // 기술 KPI
    let crashRate: Double
    let appLaunchTime: TimeInterval
    let apiResponseTime: TimeInterval
    let errorRate: Double
    
    func calculateHealthScore() -> Double {
        // 종합 앱 건강도 점수 (0-100)
        let productScore = min(100, Double(dailyActiveUsers) / Double(monthlyActiveUsers) * 100)
        let businessScore = conversionRate * 100
        let technicalScore = max(0, 100 - (crashRate * 100))
        
        return (productScore + businessScore + technicalScore) / 3
    }
}
```

분석의 핵심은 액션 가능한 인사이트를 얻는 것이다. 데이터는 수집하기 위해 존재하는 것이 아니라, 더 나은 제품을 만들기 위해 존재한다. 사용자의 행동을 이해하고, 가설을 세우고, 실험하고, 개선하는 순환 구조가 성공적인 앱의 핵심이다.

## Connections
→ [[299-monetization-strategies-advanced]]
→ [[300-ab-testing-frameworks]]
← [[297-app-store-optimization-aso]]
← [[295-continuous-integration-xcode-cloud]]

---
Level: L4
Date: 2025-08-16
Tags: #analytics #user-behavior #metrics #cohort-analysis #ab-testing #kpi #data-driven