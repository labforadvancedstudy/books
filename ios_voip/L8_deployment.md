# L8: 배포와 운영

## Core Insight
앱 개발의 90%는 배포 후에 시작된다. 실제 사용자, 실제 네트워크, 실제 기기에서 일어나는 일들은 개발 환경과 완전히 다르다.

---

## 앱스토어 심사 준비

Apple은 VoIP 앱을 특별히 엄격하게 심사한다:

### 심사 체크리스트

```swift
struct AppStoreReviewChecklist {
    let requirements = [
        RequirementItem(
            title: "VoIP 백그라운드 모드 정당성",
            description: "왜 백그라운드에서 실행되어야 하는가?",
            evidence: """
            우리 앱은 실시간 음성 통화를 제공합니다.
            사용자가 다른 앱을 사용하거나 화면이 꺼진 상태에서도
            통화를 받을 수 있어야 합니다.
            """
        ),
        
        RequirementItem(
            title: "CallKit 통합",
            description: "iOS 13+ 필수",
            evidence: "CallKitManager.swift 참조"
        ),
        
        RequirementItem(
            title: "긴급 전화 고지",
            description: "911 등 긴급 전화 불가능 명시",
            evidence: """
            앱 설명과 첫 실행 시 팝업으로 고지
            "이 앱은 긴급 전화(911 등)를 지원하지 않습니다"
            """
        ),
        
        RequirementItem(
            title: "개인정보 처리방침",
            description: "음성 데이터 처리 방법 명시",
            evidence: "https://example.com/privacy"
        ),
        
        RequirementItem(
            title: "테스트 계정",
            description: "심사팀용 테스트 계정 2개",
            evidence: """
            ID: apple_review_1@test.com / PW: Test1234!
            ID: apple_review_2@test.com / PW: Test1234!
            """
        )
    ]
}
```

### App Store Connect 설정

```swift
// Info.plist 필수 설정
let infoPlistRequirements = """
<key>UIBackgroundModes</key>
<array>
    <string>voip</string>
    <string>audio</string>
</array>

<key>NSMicrophoneUsageDescription</key>
<string>통화를 위해 마이크 접근이 필요합니다</string>

<key>NSCameraUsageDescription</key>
<string>영상 통화를 위해 카메라 접근이 필요합니다</string>

<key>NSContactsUsageDescription</key>
<string>연락처에서 통화할 사람을 선택할 수 있습니다</string>
"""
```

### 리젝 대응 전략

```swift
class RejectionHandler {
    enum CommonRejections {
        case guideline_2_5_4  // 백그라운드 VoIP 부적절 사용
        case guideline_4_2_1  // 최소 기능 미달
        case guideline_5_1_1  // 데이터 수집 고지 부족
        
        var response: String {
            switch self {
            case .guideline_2_5_4:
                return """
                우리 앱은 실제 VoIP 통화를 제공합니다:
                1. WebRTC를 사용한 실시간 음성/영상 통화
                2. CallKit 완전 통합
                3. 백그라운드에서 수신 전화 처리 필수
                
                첨부: 통화 플로우 다이어그램, 코드 스니펫
                """
                
            case .guideline_4_2_1:
                return """
                우리 앱의 고유 기능:
                1. End-to-End 암호화
                2. HD 음질 (Opus 코덱)
                3. 그룹 통화 지원
                4. 화면 공유 기능
                """
                
            case .guideline_5_1_1:
                return """
                개인정보 처리 상세:
                - 음성 데이터: E2EE 암호화, 서버 저장 안함
                - 연락처: 해시 처리 후 매칭만 수행
                - 위치 정보: 수집하지 않음
                
                Privacy Policy 업데이트 완료
                """
            }
        }
    }
}
```

## 서버 인프라 구축

### 시그널링 서버

```javascript
// Node.js + Socket.io 시그널링 서버
const express = require('express');
const https = require('https');
const socketIO = require('socket.io');

const app = express();
const server = https.createServer({
    cert: fs.readFileSync('cert.pem'),
    key: fs.readFileSync('key.pem')
}, app);

const io = socketIO(server, {
    cors: { origin: '*' }
});

// 룸 관리
const rooms = new Map();

io.on('connection', (socket) => {
    console.log(`Client connected: ${socket.id}`);
    
    // 룸 참여
    socket.on('join', (roomId) => {
        socket.join(roomId);
        
        if (!rooms.has(roomId)) {
            rooms.set(roomId, new Set());
        }
        rooms.get(roomId).add(socket.id);
        
        // 기존 참여자들에게 알림
        socket.to(roomId).emit('peer-joined', {
            peerId: socket.id
        });
    });
    
    // WebRTC 시그널링
    socket.on('offer', (data) => {
        socket.to(data.roomId).emit('offer', {
            sdp: data.sdp,
            from: socket.id
        });
    });
    
    socket.on('answer', (data) => {
        socket.to(data.to).emit('answer', {
            sdp: data.sdp,
            from: socket.id
        });
    });
    
    socket.on('ice-candidate', (data) => {
        socket.to(data.roomId).emit('ice-candidate', {
            candidate: data.candidate,
            from: socket.id
        });
    });
    
    // 연결 종료
    socket.on('disconnect', () => {
        rooms.forEach((participants, roomId) => {
            if (participants.has(socket.id)) {
                participants.delete(socket.id);
                socket.to(roomId).emit('peer-left', {
                    peerId: socket.id
                });
            }
        });
    });
});

server.listen(3000, () => {
    console.log('Signaling server running on port 3000');
});
```

### TURN 서버 설정

```bash
# coturn 설치 및 설정
sudo apt-get install coturn

# /etc/turnserver.conf
listening-port=3478
fingerprint
lt-cred-mech
user=username:password
realm=yourdomain.com
cert=/etc/ssl/certs/cert.pem
pkey=/etc/ssl/private/key.pem
```

### 푸시 알림 서버

```swift
// Vapor (Swift 서버)로 APNs 푸시 전송
import Vapor
import APNS

func configurePush(_ app: Application) throws {
    // APNs 설정
    let apnsConfig = APNSConfiguration(
        authenticationMethod: .jwt(
            key: .private(pem: Data(apnsPrivateKey.utf8)),
            keyIdentifier: "KEY_ID",
            teamIdentifier: "TEAM_ID"
        ),
        topic: "com.app.voip.voip",  // .voip 접미사 중요!
        environment: .production
    )
    
    app.apns.configuration = apnsConfig
    
    // VoIP 푸시 전송 엔드포인트
    app.post("send-voip-push") { req -> EventLoopFuture<HTTPStatus> in
        let payload = try req.content.decode(VoIPPushPayload.self)
        
        // VoIP 푸시 페이로드
        let notification = APNSPayload(
            alert: nil,  // VoIP 푸시는 alert 없음
            badge: nil,
            sound: nil,
            customKeys: [
                "callId": .string(payload.callId),
                "callerName": .string(payload.callerName),
                "handle": .string(payload.handle),
                "hasVideo": .bool(payload.hasVideo)
            ]
        )
        
        return req.apns.send(notification, 
                           to: payload.deviceToken,
                           topic: "com.app.voip.voip",
                           priority: .immediately,
                           collapseIdentifier: nil,
                           apnsPushType: .voip)  // 중요!
            .map { _ in .ok }
    }
}
```

## 모니터링과 분석

### 실시간 품질 모니터링

```swift
class QualityMonitoringService {
    private let analytics = AnalyticsService()
    
    func reportCallQuality(_ metrics: CallQualityMetrics) {
        // Firebase Analytics
        Analytics.logEvent("call_quality", parameters: [
            "mos_score": metrics.mos,
            "packet_loss": metrics.packetLoss,
            "jitter": metrics.jitter,
            "rtt": metrics.rtt,
            "codec": metrics.codec,
            "network_type": metrics.networkType
        ])
        
        // 품질이 낮으면 상세 로그
        if metrics.mos < 3.0 {
            reportPoorQuality(metrics)
        }
    }
    
    private func reportPoorQuality(_ metrics: CallQualityMetrics) {
        // Sentry로 상세 정보 전송
        SentrySDK.capture(message: "Poor call quality") { scope in
            scope.setContext("call_metrics", value: [
                "mos": metrics.mos,
                "packet_loss": metrics.packetLoss,
                "device_model": UIDevice.current.model,
                "ios_version": UIDevice.current.systemVersion,
                "network_type": metrics.networkType,
                "carrier": metrics.carrier ?? "Unknown"
            ])
        }
    }
}
```

### 크래시 리포팅

```swift
class CrashReporter {
    static func setup() {
        // Crashlytics 설정
        FirebaseApp.configure()
        
        // 추가 메타데이터
        Crashlytics.crashlytics().setCustomValue(
            UserDefaults.standard.string(forKey: "userId") ?? "anonymous",
            forKey: "user_id"
        )
        
        // 중요 이벤트 로깅
        Crashlytics.crashlytics().log("App launched")
    }
    
    static func logCallEvent(_ event: String, metadata: [String: Any]) {
        Crashlytics.crashlytics().log(event)
        
        for (key, value) in metadata {
            Crashlytics.crashlytics().setCustomValue(value, forKey: key)
        }
    }
}
```

## CI/CD 파이프라인

### Fastlane 설정

```ruby
# Fastfile
default_platform(:ios)

platform :ios do
  desc "Run tests"
  lane :test do
    run_tests(
      scheme: "VoIPApp",
      devices: ["iPhone 14", "iPhone SE (3rd generation)"],
      code_coverage: true
    )
  end
  
  desc "Beta deployment"
  lane :beta do
    # 버전 증가
    increment_build_number
    
    # 빌드
    build_app(
      scheme: "VoIPApp",
      configuration: "Release",
      export_method: "app-store"
    )
    
    # TestFlight 업로드
    upload_to_testflight(
      skip_waiting_for_build_processing: true,
      changelog: "Bug fixes and performance improvements"
    )
    
    # Slack 알림
    slack(
      message: "New beta build uploaded to TestFlight! 🚀",
      channel: "#ios-releases"
    )
  end
  
  desc "Production release"
  lane :release do
    # 스크린샷 생성
    capture_screenshots
    
    # 메타데이터 준비
    prepare_metadata
    
    # 앱스토어 업로드
    upload_to_app_store(
      submit_for_review: true,
      automatic_release: false,
      force: true,
      metadata_path: "./metadata",
      screenshots_path: "./screenshots"
    )
  end
end
```

### GitHub Actions

```yaml
# .github/workflows/ios.yml
name: iOS CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: macos-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup Xcode
      uses: maxim-lobanov/setup-xcode@v1
      with:
        xcode-version: latest-stable
    
    - name: Install dependencies
      run: |
        pod install
        
    - name: Run tests
      run: |
        xcodebuild test \
          -workspace VoIPApp.xcworkspace \
          -scheme VoIPApp \
          -destination 'platform=iOS Simulator,name=iPhone 14'
    
    - name: Build
      run: |
        xcodebuild build \
          -workspace VoIPApp.xcworkspace \
          -scheme VoIPApp \
          -configuration Release
  
  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: macos-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup Fastlane
      run: |
        bundle install
    
    - name: Deploy to TestFlight
      env:
        MATCH_PASSWORD: ${{ secrets.MATCH_PASSWORD }}
        FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD: ${{ secrets.APP_SPECIFIC_PASSWORD }}
      run: |
        bundle exec fastlane beta
```

## 운영 대시보드

```swift
// 관리자용 대시보드 데이터
struct OperationalMetrics {
    let activeUsers: Int
    let ongoingCalls: Int
    let avgCallDuration: TimeInterval
    let serverLoad: Double
    let errorRate: Double
    
    func generateDashboard() -> DashboardData {
        return DashboardData(
            kpis: [
                KPI(name: "DAU", value: activeUsers, trend: .up),
                KPI(name: "동시 통화", value: ongoingCalls, trend: .stable),
                KPI(name: "평균 통화 시간", value: avgCallDuration, trend: .up),
                KPI(name: "에러율", value: errorRate, trend: .down)
            ],
            alerts: checkAlerts(),
            graphs: generateGraphs()
        )
    }
    
    private func checkAlerts() -> [Alert] {
        var alerts: [Alert] = []
        
        if serverLoad > 0.8 {
            alerts.append(Alert(level: .warning, 
                              message: "서버 부하 높음: \(serverLoad * 100)%"))
        }
        
        if errorRate > 0.05 {
            alerts.append(Alert(level: .critical,
                              message: "에러율 임계치 초과: \(errorRate * 100)%"))
        }
        
        return alerts
    }
}
```

## A/B 테스트

```swift
class ABTestManager {
    enum TestVariant: String {
        case control = "A"
        case variant = "B"
    }
    
    func getVariant(for feature: String) -> TestVariant {
        // 사용자 ID 기반 일관된 할당
        let hash = "\(userId)-\(feature)".hashValue
        return hash % 2 == 0 ? .control : .variant
    }
    
    func applyCodecTest() {
        let variant = getVariant(for: "codec_test")
        
        switch variant {
        case .control:
            // Opus 128kbps
            configureOpus(bitrate: 128000)
            
        case .variant:
            // Opus 64kbps with FEC
            configureOpus(bitrate: 64000, fec: true)
        }
        
        // 결과 추적
        Analytics.logEvent("ab_test_exposure", parameters: [
            "test_name": "codec_test",
            "variant": variant.rawValue
        ])
    }
}
```

## 사용자 피드백 수집

```swift
class FeedbackCollector {
    func showCallQualitySurvey(after call: Call) {
        guard call.duration > 30 else { return }  // 30초 이상 통화만
        
        let alert = UIAlertController(
            title: "통화 품질은 어떠셨나요?",
            message: nil,
            preferredStyle: .alert
        )
        
        let ratings = ["😞 나쁨", "😐 보통", "😊 좋음", "😄 매우 좋음"]
        
        for (index, rating) in ratings.enumerated() {
            alert.addAction(UIAlertAction(title: rating, style: .default) { _ in
                self.submitFeedback(callId: call.id, rating: index + 1)
            })
        }
        
        alert.addAction(UIAlertAction(title: "나중에", style: .cancel))
        
        presentAlert(alert)
    }
    
    private func submitFeedback(callId: String, rating: Int) {
        API.submitFeedback(
            callId: callId,
            rating: rating,
            metadata: [
                "device": UIDevice.current.model,
                "os_version": UIDevice.current.systemVersion,
                "app_version": Bundle.main.appVersion,
                "network": NetworkMonitor.currentType
            ]
        )
    }
}
```

## 버전 관리와 마이그레이션

```swift
class VersionManager {
    func checkForUpdates() {
        API.getLatestVersion { latestVersion in
            let currentVersion = Bundle.main.appVersion
            
            if latestVersion.isNewerThan(currentVersion) {
                if latestVersion.isCritical {
                    self.forceUpdate()
                } else {
                    self.suggestUpdate()
                }
            }
        }
    }
    
    func migrateData(from oldVersion: String, to newVersion: String) {
        // 버전별 마이그레이션
        if oldVersion < "2.0.0" && newVersion >= "2.0.0" {
            // v2 마이그레이션: 암호화 키 재생성
            regenerateEncryptionKeys()
        }
        
        if oldVersion < "2.1.0" && newVersion >= "2.1.0" {
            // v2.1 마이그레이션: 데이터베이스 스키마 변경
            migrateDatabase()
        }
    }
}
```

## 장애 대응 플레이북

```swift
struct IncidentPlaybook {
    enum IncidentType {
        case serverDown
        case highLatency
        case massivePacketLoss
        case authenticationFailure
        
        var runbook: String {
            switch self {
            case .serverDown:
                return """
                1. 백업 서버로 자동 전환
                2. 사용자에게 공지
                3. 주 서버 재시작 시도
                4. 로그 분석
                5. RCA (Root Cause Analysis)
                """
                
            case .highLatency:
                return """
                1. 리전별 레이턴시 체크
                2. CDN 캐시 확인
                3. 데이터베이스 쿼리 최적화
                4. 오토스케일링 트리거
                """
                
            default:
                return "표준 대응 절차 참조"
            }
        }
    }
}
```

## 규정 준수

```swift
class ComplianceManager {
    func ensureGDPRCompliance() {
        // 데이터 수집 동의
        requestDataCollectionConsent()
        
        // 데이터 삭제 요청 처리
        handleDataDeletionRequests()
        
        // 데이터 이동성
        provideDataPortability()
    }
    
    func ensureHIPAACompliance() {
        // 의료 정보 보호 (원격 진료 앱의 경우)
        encryptAllHealthData()
        implementAuditLogs()
        restrictDataAccess()
    }
}
```

## 마치며

배포는 시작일 뿐이다. 
실제 전투는 운영에서 시작된다.

사용자는 예상치 못한 방식으로 앱을 사용하고,
네트워크는 예상치 못한 방식으로 작동하고,
iOS는 예상치 못한 시점에 업데이트된다.

하지만 준비된 개발자는 이 모든 것을 기회로 만든다.

다음, 마지막 레벨에서는 이 모든 기술 너머의 본질,
인간 연결의 철학을 다룬다.

---

## Connections

→ [[L9_philosophy]] - 통신의 미래와 철학

← [[L7_optimization]] - 최적화로 돌아가기
← [[index]] - 목차

---

Level: L8
Date: 2025-08-15
Tags: #deployment #appstore #cicd #monitoring #operations