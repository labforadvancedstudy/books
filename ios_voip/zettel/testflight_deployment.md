# TestFlight 배포 전략

## 개념
iOS VoIP 앱의 베타 테스트를 위한 TestFlight 배포 프로세스와 모범 사례.

## 빌드 구성
```swift
// Build Configuration
enum BuildConfiguration {
    case debug
    case testflight
    case appStore
    
    var serverURL: String {
        switch self {
        case .debug:
            return "wss://dev.voip.example.com"
        case .testflight:
            return "wss://staging.voip.example.com"
        case .appStore:
            return "wss://voip.example.com"
        }
    }
    
    var isLoggingEnabled: Bool {
        switch self {
        case .debug, .testflight:
            return true
        case .appStore:
            return false
        }
    }
    
    static var current: BuildConfiguration {
        #if DEBUG
        return .debug
        #else
        if Bundle.main.appStoreReceiptURL?.lastPathComponent == "sandboxReceipt" {
            return .testflight
        } else {
            return .appStore
        }
        #endif
    }
}
```

## 버전 관리
```swift
struct AppVersion {
    static var version: String {
        Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String ?? "Unknown"
    }
    
    static var build: String {
        Bundle.main.infoDictionary?["CFBundleVersion"] as? String ?? "Unknown"
    }
    
    static var fullVersion: String {
        "\(version) (\(build))"
    }
    
    // TestFlight 빌드 정보
    static func incrementBuildNumber() {
        // fastlane에서 자동 증가
        // agvtool next-version -all
    }
}
```

## TestFlight 특화 기능
```swift
class TestFlightFeatures {
    static var isTestFlightBuild: Bool {
        BuildConfiguration.current == .testflight
    }
    
    static func setupTestFlightFeatures() {
        if isTestFlightBuild {
            // 디버그 메뉴 활성화
            enableDebugMenu()
            
            // 상세 로그 활성화
            enableVerboseLogging()
            
            // 테스트 서버 연결
            connectToTestServer()
            
            // 크래시 리포팅 활성화
            enableCrashReporting()
        }
    }
    
    private static func enableDebugMenu() {
        // Shake gesture로 디버그 메뉴 표시
        UIApplication.shared.windows.first?.rootViewController?
            .becomeFirstResponder()
    }
}
```

## 피드백 수집
```swift
class TestFlightFeedback {
    static func collectFeedback() {
        let feedbackVC = FeedbackViewController()
        
        feedbackVC.addField("Call Quality", type: .rating(1...5))
        feedbackVC.addField("Audio Issues", type: .multipleChoice([
            "Echo",
            "Noise",
            "Delay",
            "Choppy",
            "None"
        ]))
        feedbackVC.addField("Comments", type: .text)
        
        feedbackVC.onSubmit = { feedback in
            submitToAnalytics(feedback)
            submitToCrashlytics(feedback)
        }
    }
    
    private static func submitToAnalytics(_ feedback: [String: Any]) {
        Analytics.logEvent("testflight_feedback", parameters: feedback)
    }
}
```

## A/B 테스트 구성
```swift
class TestFlightABTesting {
    enum TestGroup: String, CaseIterable {
        case control = "A"
        case variant = "B"
        
        var codecPriority: [AudioCodec] {
            switch self {
            case .control:
                return [.opus, .g711]
            case .variant:
                return [.g711, .opus]
            }
        }
    }
    
    static func assignTestGroup() -> TestGroup {
        // 사용자 ID 기반 랜덤 할당
        let userId = getUserId()
        let hash = userId.hashValue
        
        if hash % 2 == 0 {
            return .control
        } else {
            return .variant
        }
    }
    
    static func logTestMetrics(group: TestGroup, mos: Float) {
        Analytics.logEvent("ab_test_result", parameters: [
            "group": group.rawValue,
            "mos_score": mos,
            "version": AppVersion.fullVersion
        ])
    }
}
```

## 자동 배포 (Fastlane)
```ruby
# Fastfile
lane :beta do
  # 버전 번호 증가
  increment_build_number(
    build_number: latest_testflight_build_number + 1
  )
  
  # 빌드
  build_app(
    scheme: "VoIPApp",
    configuration: "Release",
    export_method: "app-store",
    include_bitcode: true
  )
  
  # TestFlight 업로드
  upload_to_testflight(
    skip_waiting_for_build_processing: true,
    changelog: generate_changelog,
    groups: ["Internal", "Beta Testers"]
  )
  
  # Slack 알림
  slack(
    message: "New TestFlight build available: #{get_version_number} (#{get_build_number})",
    channel: "#ios-releases"
  )
end

def generate_changelog
  changelog = changelog_from_git_commits(
    between: [last_git_tag, "HEAD"],
    pretty: "- %s",
    merge_commit_filtering: "exclude_merges"
  )
  
  "What's New:\n#{changelog}"
end
```

## 테스터 관리
```swift
struct TestFlightTester {
    let email: String
    let group: String
    let permissions: Set<Feature>
    
    enum Feature {
        case videoCall
        case groupCall
        case recording
        case advancedSettings
    }
}

class TesterManager {
    static func inviteTesters(_ testers: [TestFlightTester]) {
        // App Store Connect API 사용
        for tester in testers {
            addTesterToGroup(tester.email, group: tester.group)
            configurePermissions(tester.email, permissions: tester.permissions)
        }
    }
    
    static func exportTestResults() -> Data {
        // 테스트 결과 CSV 엑스포트
        var csv = "Email,Version,Crashes,Sessions,MOS Score\n"
        
        for tester in getActiveTesters() {
            let stats = getTestStatistics(for: tester)
            csv += "\(tester.email),\(stats.version),\(stats.crashes),\(stats.sessions),\(stats.averageMOS)\n"
        }
        
        return csv.data(using: .utf8)!
    }
}
```

## 연관 개녕
- [[crash_reporting]]
- [[analytics_integration]]
- [[ab_testing]]

## 태그
#deployment #testflight #beta #testing #distribution