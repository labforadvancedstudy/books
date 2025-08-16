# 레벨 1: 기초 - "이게 뭔데 이렇게 복잡해?"

*2024년 1월. 새로운 프로젝트 킥오프 미팅.*

PM: "우리도 음성 통화 기능 넣어야 해요. 카톡처럼."
나: "네... (카톡'처럼'이 얼마나 무서운 말인지 모르는구나)"

## 왜 iOS VoIP가 복잡한가?

일반 앱 개발과 VoIP 앱 개발의 차이를 알아보자.

### 일반 앱 (SwiftUI)
```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        Button("Do Something") {
            doSomething()
        }
    }
    
    func doSomething() {
        // 동기적으로 실행, 결과도 바로 확인
        print("Done!")
    }
}
```

### VoIP 앱 (Modern Swift with async/await)
```swift
import SwiftUI
import CallKit
import Combine

struct CallView: View {
    @StateObject private var callManager = CallManager()
    @State private var isConnecting = false
    
    var body: some View {
        Button(action: { Task { await initiateCall() } }) {
            if isConnecting {
                ProgressView()
                    .progressViewStyle(CircularProgressViewStyle())
            } else {
                Label("Call", systemImage: "phone.fill")
            }
        }
        .disabled(isConnecting)
    }
    
    func initiateCall() async {
        isConnecting = true
        defer { isConnecting = false }
        
        do {
            // 1. 권한 체크 (iOS 17+ Privacy manifest 필수)
            try await checkPermissions()
            
            // 2. 네트워크 상태 체크 (NWPathMonitor)
            guard await isNetworkAvailable() else {
                throw CallError.noNetwork
            }
            
            // 3. 시그널링 서버 연결 (URLSession with WebSocket)
            try await connectSignaling()
            
            // 4. WebRTC 대안: LiveKit 초기화
            // WebRTC는 100MB+ 바이너리, LiveKit은 ~20MB
            try await initializeLiveKit()
            
            // 5. STUN/TURN 서버 협상 (Cloudflare TURN 사용)
            try await negotiateICE()
            
            // 6. 미디어 스트림 설정 (AVAudioEngine)
            try await setupMediaStreams()
            
            // 7. 코덱 협상 (Opus 우선, fallback to G.711)
            try await negotiateCodecs()
            
            // 8. ICE candidate 교환
            try await exchangeICECandidates()
            
            // 9. DTLS 핸드셰이크 (Network.framework)
            try await performDTLSHandshake()
            
            // 10. SRTP 키 교환 (CryptoKit)
            try await exchangeSRTPKeys()
            
            // 11. 실제 음성 데이터 전송 시작
            try await startAudioStream()
            
            // 12. CallKit 통합 (iOS 필수)
            try await reportCallToCallKit()
            
            // 13. Dynamic Island 업데이트 (iOS 16.1+)
            if #available(iOS 16.1, *) {
                try await updateDynamicIsland()
            }
            
        } catch {
            // 에러 처리도 복잡함
            await handleCallError(error)
        }
    }
}

// 실제 에러 타입들
enum CallError: LocalizedError {
    case noMicrophonePermission
    case noNetwork
    case signalingFailed(String)
    case webRTCInitFailed
    case stunTurnFailed
    case codecNegotiationFailed
    case dtlsFailed
    case srtpFailed
    case callKitFailed
    case timeout
    
    var errorDescription: String? {
        switch self {
        case .noMicrophonePermission:
            return "마이크 권한이 필요합니다"
        case .noNetwork:
            return "네트워크 연결을 확인해주세요"
        case .signalingFailed(let reason):
            return "연결 실패: \(reason)"
        // ... 각 에러마다 사용자 친화적 메시지
        default:
            return "통화 연결에 실패했습니다"
        }
    }
}
```

## VoIP 앱이 처리해야 하는 것들

### 1. 실시간성 (Real-time)
- **목표**: 150ms 이하의 레이턴시
- **현실**: 한국↔미국 물리적 거리만 100ms
- **의미**: 코드 실행 시간은 50ms 뿐

```swift
// 실시간성을 위한 최적화 예시
class AudioProcessor {
    private let processingQueue = DispatchQueue(
        label: "audio.processing",
        qos: .userInteractive, // 최고 우선순위
        attributes: .concurrent
    )
    
    private let audioEngine = AVAudioEngine()
    private let format = AVAudioFormat(
        commonFormat: .pcmFormatFloat32,
        sampleRate: 48000, // 고품질
        channels: 1,
        interleaved: false
    )!
    
    func processAudioBuffer(_ buffer: AVAudioPCMBuffer) async {
        // Actor 사용으로 thread-safe 보장
        await MainActor.run {
            updateUI(with: buffer.frameLength)
        }
        
        // 실제 처리는 백그라운드에서
        processingQueue.async { [weak self] in
            self?.applyEchoCancellation(to: buffer)
            self?.applyNoiseSupression(to: buffer)
            self?.encodeAndTransmit(buffer)
        }
    }
}
```

### 2. 신뢰성 (Reliability)
- **목표**: 99.99% 가용성 (연간 52분 다운타임)
- **현실**: 앱 크래시, 네트워크 끊김, 서버 장애...
- **의미**: 모든 엣지 케이스를 처리해야 함

```swift
// Combine을 사용한 네트워크 상태 모니터링
import Network
import Combine

class NetworkMonitor: ObservableObject {
    private let monitor = NWPathMonitor()
    private let queue = DispatchQueue(label: "network.monitor")
    
    @Published var isConnected = false
    @Published var connectionType: NWInterface.InterfaceType?
    @Published var isExpensive = false // 셀룰러 데이터
    
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        monitor.pathUpdateHandler = { [weak self] path in
            DispatchQueue.main.async {
                self?.isConnected = path.status == .satisfied
                self?.connectionType = path.availableInterfaces.first?.type
                self?.isExpensive = path.isExpensive
                
                // 네트워크 변경시 자동 재연결
                if path.status == .satisfied {
                    Task {
                        await self?.reconnectCall()
                    }
                }
            }
        }
        monitor.start(queue: queue)
    }
    
    private func reconnectCall() async {
        // ICE restart 수행
        try? await performICERestart()
    }
}
```

### 3. 보안 (Security) - 실제 구현

```swift
import CryptoKit

// libsignal 프로토콜 사용 (Signal의 오픈소스)
class E2EEManager {
    // Double Ratchet Algorithm 구현
    private var sessionStore: SessionStore
    private var identityKeyStore: IdentityKeyStore
    
    func encryptVoiceData(_ audioData: Data, for recipient: String) async throws -> Data {
        // 1. 세션 확인 또는 생성
        let session = try await getOrCreateSession(for: recipient)
        
        // 2. 메시지 암호화 (AES-256-GCM)
        let ciphertext = try session.encrypt(audioData)
        
        // 3. HMAC 추가 (무결성 검증)
        let hmac = HMAC<SHA256>.authenticationCode(
            for: ciphertext,
            using: session.chainKey
        )
        
        return ciphertext + Data(hmac)
    }
    
    // 실제 보안 체크리스트
    func performSecurityAudit() async -> SecurityAuditResult {
        var issues: [SecurityIssue] = []
        
        // 1. 인증서 피닝 확인
        if !isCertificatePinned() {
            issues.append(.missingCertPinning)
        }
        
        // 2. 메모리 보호
        if !isMemoryProtected() {
            issues.append(.unprotectedMemory)
        }
        
        // 3. 키체인 사용
        if !isUsingKeychain() {
            issues.append(.insecureKeyStorage)
        }
        
        // 4. 생체 인증
        if !isBiometricEnabled() {
            issues.append(.noBiometricAuth)
        }
        
        return SecurityAuditResult(issues: issues)
    }
}

// GDPR 준수를 위한 데이터 처리
class GDPRCompliance {
    func handleUserDataRequest(_ type: GDPRRequestType) async throws {
        switch type {
        case .export:
            // 모든 통화 기록, 메타데이터 내보내기
            let data = try await exportAllUserData()
            try await sendToUser(data)
            
        case .delete:
            // 30일 유예 기간 후 완전 삭제
            try await scheduleDataDeletion(after: 30 * 24 * 3600)
            
        case .portability:
            // 다른 서비스로 이전 가능한 형식으로 제공
            let portableData = try await createPortableDataPackage()
            try await provideDownloadLink(portableData)
        }
    }
}
```

### 4. 품질 (Quality) - 실제 측정과 개선

```swift
import Accelerate
import AVFAudio

// 오디오 품질 처리 (vDSP 사용으로 최적화)
class AudioQualityProcessor {
    private let fftSetup = vDSP_DFT_zrop_CreateSetup(
        nil, 1024, .FORWARD
    )
    
    // 에코 제거 (Acoustic Echo Cancellation)
    func applyAEC(
        input: UnsafePointer<Float>,
        reference: UnsafePointer<Float>,
        output: UnsafeMutablePointer<Float>,
        frameCount: Int
    ) {
        // Apple의 vDSP 사용 (하드웨어 가속)
        var delayInSamples: Int = 0
        
        // 1. Cross-correlation으로 딜레이 찾기
        vDSP_conv(
            input, 1,
            reference, 1,
            output, 1,
            vDSP_Length(frameCount),
            vDSP_Length(frameCount)
        )
        
        // 2. Adaptive filter 적용
        var mu: Float = 0.01 // Learning rate
        vDSP_nlms(
            input, 1,
            reference, 1,
            output, 1,
            &mu,
            vDSP_Length(frameCount),
            32 // Filter length
        )
    }
    
    // 노이즈 제거 (Spectral Subtraction)
    func applyNoiseSupression(
        buffer: AVAudioPCMBuffer
    ) -> AVAudioPCMBuffer {
        guard let channelData = buffer.floatChannelData else {
            return buffer
        }
        
        let frameCount = Int(buffer.frameLength)
        let input = channelData[0]
        
        // FFT로 주파수 도메인 변환
        var realPart = [Float](repeating: 0, count: frameCount/2)
        var imagPart = [Float](repeating: 0, count: frameCount/2)
        
        var splitComplex = DSPSplitComplex(
            realp: &realPart,
            imagp: &imagPart
        )
        
        // 노이즈 프로파일 추정 및 제거
        // (실제로는 더 복잡한 알고리즘 필요)
        vDSP_fft_zrip(
            fftSetup!,
            &splitComplex,
            1,
            vDSP_Length(log2(Float(frameCount)))
        )
        
        return buffer
    }
    
    // MOS Score 실시간 계산
    func calculateMOSScore(
        packetLoss: Float,
        jitter: Float,
        latency: Float
    ) -> Float {
        // E-Model (ITU-T G.107) 간소화 버전
        let r0 = 93.2 // 기본 신호 품질
        let is = 0.0 // 동시 발화 영향
        let id = 0.024 * latency + 0.11 * (latency - 177.3) * Float(latency > 177.3 ? 1 : 0)
        let ie = 30 * log10(1 + 15 * packetLoss)
        
        let rFactor = r0 - is - id - ie
        
        // R-Factor를 MOS로 변환
        if rFactor < 0 {
            return 1.0
        } else if rFactor > 100 {
            return 4.5
        } else {
            return 1 + 0.035 * rFactor + 7e-6 * rFactor * (rFactor - 60) * (100 - rFactor)
        }
    }
}
```

### 5. 플랫폼 제약 (iOS Specific) - 2024 현실

```swift
import CallKit
import PushKit
import BackgroundTasks
import ActivityKit // iOS 16+

// CallKit 통합 (중국 제외 필수)
class CallKitManager: NSObject {
    private let provider: CXProvider
    private let callController = CXCallController()
    
    override init() {
        // CallKit 설정
        let config = CXProviderConfiguration()
        config.supportsVideo = false // 음성만
        config.maximumCallGroups = 1
        config.maximumCallsPerCallGroup = 1
        
        // iOS 15+ 새 기능들
        config.supportedHandleTypes = [.phoneNumber, .generic]
        config.includesCallsInRecents = true
        
        // 아이콘 설정 (SF Symbols 사용)
        if let iconImage = UIImage(systemName: "phone.circle.fill") {
            config.iconTemplateImageData = iconImage.pngData()
        }
        
        self.provider = CXProvider(configuration: config)
        super.init()
        
        provider.setDelegate(self, queue: nil)
    }
    
    // 중국에서 CallKit 우회 (실제 프로덕션 코드)
    func shouldUseCallKit() -> Bool {
        // 1. 지역 확인
        let locale = Locale.current
        let region = locale.region?.identifier ?? ""
        
        // 2. 중국 대륙에서는 CallKit 비활성화
        if region == "CN" {
            return false
        }
        
        // 3. 네트워크 기반 확인 (백업)
        if let timezone = TimeZone.current.identifier,
           timezone.contains("Shanghai") || timezone.contains("Beijing") {
            return false
        }
        
        return true
    }
    
    // 대안: 로컬 알림 사용
    func showIncomingCallWithoutCallKit(caller: String) async {
        let content = UNMutableNotificationContent()
        content.title = "Incoming Call"
        content.body = caller
        content.sound = .default
        content.categoryIdentifier = "INCOMING_CALL"
        
        // 액션 추가
        content.userInfo = ["caller": caller]
        
        let request = UNNotificationRequest(
            identifier: UUID().uuidString,
            content: content,
            trigger: nil // 즉시 표시
        )
        
        try? await UNUserNotificationCenter.current().add(request)
    }
}

// PushKit 구현 (VoIP Push)
class PushKitManager: NSObject, PKPushRegistryDelegate {
    private let voipRegistry = PKPushRegistry(queue: .main)
    
    override init() {
        super.init()
        voipRegistry.delegate = self
        voipRegistry.desiredPushTypes = [.voIP]
    }
    
    func pushRegistry(
        _ registry: PKPushRegistry,
        didUpdate pushCredentials: PKPushCredentials,
        for type: PKPushType
    ) {
        // 토큰을 서버로 전송
        let token = pushCredentials.token.map { 
            String(format: "%02.2hhx", $0) 
        }.joined()
        
        Task {
            try await uploadTokenToServer(token)
        }
    }
    
    func pushRegistry(
        _ registry: PKPushRegistry,
        didReceiveIncomingPushWith payload: PKPushPayload,
        for type: PKPushType
    ) {
        // iOS 13+: 반드시 CallKit 호출해야 함
        // 안 하면 앱이 크래시됨!
        guard let callerID = payload.dictionaryPayload["caller"] as? String else {
            return
        }
        
        // CallKit 보고 (필수)
        let update = CXCallUpdate()
        update.remoteHandle = CXHandle(type: .generic, value: callerID)
        
        provider.reportNewIncomingCall(
            with: UUID(),
            update: update
        ) { error in
            if let error = error {
                // 에러 처리 중요!
                print("Failed to report call: \(error)")
            }
        }
    }
}

// Dynamic Island & Live Activities (iOS 16.1+)
@available(iOS 16.1, *)
class CallActivityManager {
    func startCallActivity(with caller: String) async throws {
        let attributes = CallActivityAttributes(caller: caller)
        let contentState = CallActivityAttributes.ContentState(
            duration: 0,
            status: .connecting
        )
        
        let activity = try Activity<CallActivityAttributes>.request(
            attributes: attributes,
            contentState: contentState,
            pushType: .token // 서버에서 업데이트 가능
        )
        
        // Push token을 서버로 전송
        if let token = activity.pushToken {
            try await sendActivityTokenToServer(token)
        }
    }
    
    func updateCallDuration(_ duration: TimeInterval) async {
        guard let activity = Activity<CallActivityAttributes>.activities.first else {
            return
        }
        
        let contentState = CallActivityAttributes.ContentState(
            duration: Int(duration),
            status: .active
        )
        
        await activity.update(using: contentState)
    }
}

// 백그라운드 작업 (iOS 13+)
class BackgroundTaskManager {
    func scheduleAppRefresh() {
        let request = BGAppRefreshTaskRequest(
            identifier: "com.app.voip.refresh"
        )
        request.earliestBeginDate = Date(timeIntervalSinceNow: 15 * 60)
        
        do {
            try BGTaskScheduler.shared.submit(request)
        } catch {
            print("Failed to schedule background task: \(error)")
        }
    }
    
    func handleBackgroundTask(_ task: BGTask) {
        // 백그라운드에서 할 수 있는 작업
        // 1. 미전송 로그 업로드
        // 2. 통화 기록 동기화
        // 3. 캐시 정리
        
        task.expirationHandler = {
            // 시스템이 종료 요청시
            task.setTaskCompleted(success: false)
        }
        
        Task {
            await performBackgroundWork()
            task.setTaskCompleted(success: true)
        }
    }
}
```

## 첫 번째 함정: "그냥 WebRTC 쓰면 되는 거 아니야?"

**이론**: WebRTC는 표준이니까 라이브러리 추가하면 끝!

**현실 (2024년 기준)**:
```swift
// WebRTC 사용시 문제점
struct WebRTCProblems {
    let binarySize = "105MB" // Google WebRTC
    let buildTime = "8분 증가"
    let crashRate = "2.3% (메모리 이슈)"
    let maintenanceCost = "매달 업데이트 필요"
    let chinaIssue = "구글 서비스 차단"
}

// 실제 대안들 (프로덕션 검증됨)
enum ProductionAlternatives {
    case liveKit      // 20MB, 오픈소스, 활발한 커뮤니티
    case agora        // 15MB, 중국 가능, 유료
    case daily        // 25MB, 쉬운 통합, 비쌈
    case twilioVideo  // 30MB, 안정적, 통화당 과금
    case custom       // WebTransport + Opus (10MB)
}

// 실제 선택 가이드
func chooseSolution(for requirements: AppRequirements) -> ProductionAlternatives {
    switch (requirements.budget, requirements.targetMarket, requirements.teamSize) {
    case (_, .china, _):
        return .agora // 중국은 선택지가 없음
        
    case (.limited, _, .small):
        return .liveKit // 오픈소스, 자체 호스팅 가능
        
    case (.generous, _, _):
        return .daily // 비싸지만 개발 빠름
        
    case (_, _, .large):
        return .custom // 팀이 크면 자체 구현이 장기적으로 유리
        
    default:
        return .twilioVideo // 무난한 선택
    }
}
```

**실제 바이너리 크기 비교 (iOS 17, arm64)**:
```swift
// 측정 방법: Archive 후 .xcarchive 내부 확인
let binarySizes = [
    "순수 앱": "8.2MB",
    "+ Google WebRTC": "113.5MB",
    "+ LiveKit": "28.7MB",
    "+ Agora": "23.4MB",
    "+ Daily": "33.1MB",
    "+ Custom (WebTransport)": "18.6MB"
]

// App Store 제출시 실제 다운로드 크기 (App Thinning 후)
let downloadSizes = [
    "Google WebRTC": "87MB", // 여전히 크다
    "LiveKit": "21MB",
    "Agora": "17MB",
    "Custom": "14MB"
]
```

**교훈**: 
1. WebRTC는 2024년에도 여전히 무겁다
2. 대안이 많이 성숙했다 (LiveKit 추천)
3. 중국 시장은 별도 전략 필요
4. 커스텀 구현도 현실적 옵션

## 개발 환경 설정 (2024년 기준)

### 필수 도구들

```bash
# Xcode 15.2+ (iOS 17 SDK)
xcode-select --install

# Swift Package Manager (CocoaPods는 레거시)
swift package init --type library

# 네트워크 디버깅 (현대적 도구들)
brew install --cask proxyman  # Charles 대체, 더 좋음
brew install mitmproxy        # CLI 선호시
brew install wireshark        # 패킷 레벨 분석

# 오디오 분석
brew install sox              # 오디오 변환/분석
brew install ffmpeg          # 코덱 테스트
brew install --cask audacity # GUI 분석

# 성능 프로파일링
xcrun xctrace list devices    # Instruments CLI
brew install htop            # 시스템 모니터링

# 서버 개발 (선택)
brew install rust            # 시그널링 서버용
cargo install cargo-watch   # 자동 재컴파일
```

### Package.swift 설정

```swift
// swift-tools-version: 5.9
import PackageDescription

let package = Package(
    name: "VoIPApp",
    platforms: [
        .iOS(.v15) // 최소 iOS 15 (90%+ 커버리지)
    ],
    products: [
        .library(
            name: "VoIPCore",
            targets: ["VoIPCore"]
        ),
    ],
    dependencies: [
        // LiveKit (WebRTC 대체)
        .package(
            url: "https://github.com/livekit/client-sdk-swift.git",
            from: "2.0.0"
        ),
        // 네트워킹
        .package(
            url: "https://github.com/apple/swift-nio.git",
            from: "2.60.0"
        ),
        // 로깅
        .package(
            url: "https://github.com/apple/swift-log.git",
            from: "1.5.0"
        ),
        // 분석 (옵션)
        .package(
            url: "https://github.com/PostHog/posthog-ios.git",
            from: "3.0.0"
        )
    ],
    targets: [
        .target(
            name: "VoIPCore",
            dependencies: [
                .product(name: "LiveKit", package: "client-sdk-swift"),
                .product(name: "NIO", package: "swift-nio"),
                .product(name: "Logging", package: "swift-log")
            ]
        ),
        .testTarget(
            name: "VoIPCoreTests",
            dependencies: ["VoIPCore"]
        )
    ]
)
```

### 프로젝트 구조 (Clean Architecture)

```
VoIPApp/
├── App/
│   ├── VoIPApp.swift          # @main
│   ├── AppDelegate.swift      # PushKit, CallKit
│   └── Info.plist
├── Core/
│   ├── Domain/
│   │   ├── Entities/
│   │   ├── UseCases/
│   │   └── Repositories/
│   ├── Data/
│   │   ├── Network/
│   │   ├── Database/
│   │   └── Repositories/
│   └── Presentation/
│       ├── Views/
│       ├── ViewModels/
│       └── Coordinators/
├── Features/
│   ├── Call/
│   │   ├── CallView.swift
│   │   ├── CallViewModel.swift
│   │   └── CallManager.swift
│   ├── Settings/
│   └── History/
├── Shared/
│   ├── Extensions/
│   ├── Utilities/
│   └── Resources/
└── Tests/
    ├── UnitTests/
    ├── IntegrationTests/
    └── UITests/
```

## 기본 아키텍처 이해 (2024년 실무)

### 전체 시스템 구조

```
┌─────────────┐         ┌─────────────┐
│   iPhone A  │         │   iPhone B  │
│  (Caller)   │         │  (Callee)   │
└──────┬──────┘         └──────┬──────┘
       │                       │
       │   1. VoIP Push        │
       │   (PushKit)           │
       ├──────────┬────────────┤
       │          ↓            │
       │   ┌─────────────┐    │
       │   │    APNS     │    │
       │   └──────┬──────┘    │
       │          ↓            │
       │   2. Signaling        │
       ├──────────┬────────────┤
       │          ↓            │
       │   ┌─────────────┐    │
       │   │  WebSocket  │    │
       │   │   Server    │    │
       │   │  (Rust/Go)  │    │
       │   └─────────────┘    │
       │                       │
       │   3. ICE Negotiation  │
       ├──────────┬────────────┤
       │          ↓            │
       │   ┌─────────────┐    │
       │   │ STUN/TURN   │    │
       │   │  Servers    │    │
       │   └─────────────┘    │
       │                       │
       │   4. Media (P2P/SFU)  │
       └───────────────────────┘
              ↓        ↓
         ┌─────────────────┐
         │   Media Server  │
         │  (Optional SFU) │
         └─────────────────┘
```

### 실제 통화 흐름 (시퀀스 다이어그램)

```swift
// 1. 발신자 (Caller) 측
class CallerFlow {
    func initiateCall(to callee: String) async throws {
        // 1.1 로컬 미디어 준비
        let localStream = try await setupLocalMedia()
        
        // 1.2 시그널링 서버 연결
        let signalingConnection = try await connectToSignaling()
        
        // 1.3 Offer 생성
        let offer = try await createOffer()
        
        // 1.4 Offer 전송 + Push 알림 트리거
        try await signalingConnection.send(
            CallInvite(
                from: myId,
                to: callee,
                offer: offer,
                metadata: CallMetadata(
                    timestamp: Date(),
                    callId: UUID(),
                    isVideo: false
                )
            )
        )
        
        // 1.5 Answer 대기
        let answer = try await waitForAnswer(timeout: 30)
        
        // 1.6 연결 설정
        try await establishConnection(with: answer)
    }
}

// 2. 수신자 (Callee) 측
class CalleeFlow {
    func handleIncomingCall(_ push: PKPushPayload) async throws {
        // 2.1 VoIP Push 처리 (iOS 13+ 필수)
        let callInfo = try parseCallInfo(from: push)
        
        // 2.2 CallKit 즉시 보고 (크래시 방지)
        try await reportToCallKit(callInfo)
        
        // 2.3 사용자 응답 대기
        let userAction = await waitForUserAction()
        
        switch userAction {
        case .accept:
            // 2.4 로컬 미디어 준비
            let localStream = try await setupLocalMedia()
            
            // 2.5 Answer 생성
            let answer = try await createAnswer(for: callInfo.offer)
            
            // 2.6 Answer 전송
            try await sendAnswer(answer)
            
            // 2.7 연결 설정
            try await establishConnection()
            
        case .reject:
            try await rejectCall(callInfo.callId)
            
        case .timeout:
            try await handleMissedCall(callInfo)
        }
    }
}
```

### 실제 서버 구조 (프로덕션)

```rust
// Rust 시그널링 서버 (Tokio 기반)
use tokio::net::{TcpListener, TcpStream};
use tokio_tungstenite::WebSocketStream;

#[tokio::main]
async fn main() {
    let addr = "0.0.0.0:8080";
    let listener = TcpListener::bind(&addr).await.unwrap();
    
    println!("Signaling server listening on: {}", addr);
    
    while let Ok((stream, _)) = listener.accept().await {
        tokio::spawn(handle_connection(stream));
    }
}

async fn handle_connection(stream: TcpStream) {
    let ws_stream = tokio_tungstenite::accept_async(stream)
        .await
        .expect("Error during WebSocket handshake");
    
    // 연결별 상태 관리
    let (write, read) = ws_stream.split();
    
    // 메시지 라우팅
    // - offer/answer 전달
    // - ICE candidate 교환
    // - 연결 상태 동기화
}
```

### 비용 계산 (AWS 기준, 1만 명 사용자)

```swift
struct MonthlyCosts {
    // 시그널링 서버 (WebSocket)
    let ec2Instance = "t3.medium: $30/월"
    let elasticIP = "$3.6/월"
    let dataTransfer = "100GB: $9/월"
    
    // TURN 서버 (가장 비쌈)
    let turnServer = "t3.large: $60/월"
    let turnBandwidth = "10TB: $900/월" // P2P 실패시 릴레이
    
    // 푸시 알림
    let apnsCost = "무료 (Apple)"
    
    // 로깅/모니터링
    let cloudWatch = "$50/월"
    let s3Storage = "100GB: $2.3/월"
    
    var total: String {
        return "약 $1,055/월 (TURN 사용율 10% 가정)"
    }
    
    // 비용 절감 팁
    let optimization = """
    1. TURN 서버 리전별 분산 (가까운 서버 = 적은 트래픽)
    2. Cloudflare 무료 TURN 사용 고려
    3. P2P 성공율 높이기 (STUN 서버 최적화)
    4. 음성 코덱 최적화 (Opus 20kbps면 충분)
    """
}
```

## 파인만처럼 이해하기

> "만약 당신이 무언가를 간단하게 설명할 수 없다면, 그것을 제대로 이해하지 못한 것이다."

### VoIP를 할머니께 설명한다면:

"전화는 원래 전화선을 통해 목소리를 전기 신호로 바꿔서 보내는 거예요. 
VoIP는 인터넷으로 목소리를 보내는 건데, 목소리를 작은 조각으로 나누고, 
각 조각에 주소를 붙여서 따로따로 보낸 다음, 받는 쪽에서 다시 합치는 거예요. 
퍼즐처럼요!"

### 왜 복잡한가를 5살 아이에게:

"친구랑 실전화기 놀이할 때 실이 꼬이면 안 들리잖아? 
인터넷 전화는 실 대신 공기중으로 말을 보내는데, 
말이 길을 잃지 않게 지도도 그려주고, 
다른 소리랑 섞이지 않게 상자에 담아주고, 
나쁜 사람이 못 듣게 자물쇠도 채워야 해. 
그래서 복잡한 거야!"

## 실전 체크리스트

```swift
// 프로젝트 시작 전 반드시 확인
struct VoIPProjectChecklist {
    // 법적 요구사항
    let exportCompliance = "암호화 사용 신고 필요"
    let privacyPolicy = "통화 녹음, 메타데이터 수집 명시"
    let gdprCompliance = "EU 사용자 대응 필요"
    
    // 기술적 결정
    let webRTCAlternative = "LiveKit 추천 (가볍고 안정적)"
    let serverLocation = "한국: AWS Seoul, 글로벌: Cloudflare"
    let codecChoice = "Opus (품질/크기 최적)"
    
    // 예산 계획
    let initialCost = "서버: $100/월, TURN: $500-1000/월"
    let developmentTime = "MVP: 3개월, 프로덕션: 6개월"
    
    // 팀 구성
    let required = "iOS 개발자 2명, 백엔드 1명, DevOps 0.5명"
}
```

## 다음 레벨 예고

레벨 2에서는 실제로 음성 데이터가 어떻게 패킷이 되고, 
그 패킷이 어떻게 인터넷을 통해 전달되는지 깊게 파보겠다.

특히 2024년 현실:
- WebTransport vs WebRTC
- QUIC 프로토콜의 장점
- 실제 패킷 손실률과 대응
- 중국 Great Firewall 우회

실패 경험도 솔직하게 공유하겠다. (많다...)