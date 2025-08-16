# L3: iOS 프레임워크 이해 (CallKit, PushKit)

## Core Insight
iOS는 VoIP 앱을 시스템의 일급 시민으로 만들어주는 프레임워크를 제공한다. CallKit은 UI와 시스템 통합을, PushKit은 백그라운드 깨우기를 담당한다.

---

## CallKit: 시스템과 하나되기

CallKit이 없던 시절, VoIP 앱들은 각자의 UI를 만들었다.
CallKit이 생기고, 모든 것이 바뀌었다.

### CallKit이 주는 것

```swift
// 1. 네이티브 전화 UI
// 2. 최근 통화 목록 통합
// 3. 연락처 연동
// 4. 차량 블루투스 지원
// 5. 잠금 화면 UI
// 6. 방해 금지 모드 준수
```

### CXProvider 설정

```swift
import CallKit

class CallKitManager: NSObject {
    private let provider: CXProvider
    private let callController = CXCallController()
    
    override init() {
        // Provider 설정
        let config = CXProviderConfiguration()
        config.localizedName = "My VoIP App"
        config.maximumCallGroups = 1
        config.maximumCallsPerCallGroup = 1
        
        // 지원하는 핸들 타입
        config.supportedHandleTypes = [.phoneNumber, .emailAddress]
        
        // 아이콘 (선택사항)
        config.iconTemplateImageData = UIImage(named: "CallIcon")?.pngData()
        
        // 벨소리 (선택사항)
        config.ringtoneSound = "ringtone.caf"
        
        // Video 지원
        config.supportsVideo = true
        
        self.provider = CXProvider(configuration: config)
        
        super.init()
        
        provider.setDelegate(self, queue: nil)
    }
}
```

### CXProviderDelegate 구현

```swift
extension CallKitManager: CXProviderDelegate {
    
    // 발신 전화 시작
    func provider(_ provider: CXProvider, perform action: CXStartCallAction) {
        // 1. 통화 준비
        let call = Call(uuid: action.callUUID, isOutgoing: true)
        call.handle = action.handle.value
        
        // 2. 오디오 세션 설정
        configureAudioSession()
        
        // 3. 네트워크 연결 시작
        WebRTCClient.shared.startCall(to: call.handle) { success in
            if success {
                // 4. 성공 시 액션 완료
                action.fulfill()
                
                // 5. 시스템에 통화 시작 알림
                provider.reportOutgoingCall(with: action.callUUID, 
                                          startedConnectingAt: Date())
            } else {
                // 실패 시
                action.fail()
            }
        }
    }
    
    // 수신 전화 응답
    func provider(_ provider: CXProvider, perform action: CXAnswerCallAction) {
        guard let call = callManager.call(with: action.callUUID) else {
            action.fail()
            return
        }
        
        // 1. 오디오 활성화
        configureAudioSession()
        
        // 2. 통화 수락 처리
        WebRTCClient.shared.answerCall(call) { success in
            if success {
                action.fulfill()
            } else {
                action.fail()
            }
        }
    }
    
    // 전화 끊기
    func provider(_ provider: CXProvider, perform action: CXEndCallAction) {
        guard let call = callManager.call(with: action.callUUID) else {
            action.fail()
            return
        }
        
        // 1. 네트워크 연결 종료
        WebRTCClient.shared.endCall(call)
        
        // 2. 오디오 세션 비활성화
        deactivateAudioSession()
        
        // 3. 통화 객체 정리
        callManager.remove(call)
        
        action.fulfill()
    }
    
    // 홀드 (대기)
    func provider(_ provider: CXProvider, perform action: CXSetHeldCallAction) {
        guard let call = callManager.call(with: action.callUUID) else {
            action.fail()
            return
        }
        
        call.isOnHold = action.isOnHold
        
        if action.isOnHold {
            WebRTCClient.shared.holdCall(call)
        } else {
            WebRTCClient.shared.unholdCall(call)
        }
        
        action.fulfill()
    }
    
    // 음소거
    func provider(_ provider: CXProvider, perform action: CXSetMutedCallAction) {
        guard let call = callManager.call(with: action.callUUID) else {
            action.fail()
            return
        }
        
        call.isMuted = action.isMuted
        WebRTCClient.shared.setMuted(action.isMuted, for: call)
        
        action.fulfill()
    }
    
    // Provider 리셋 (중요!)
    func providerDidReset(_ provider: CXProvider) {
        // 모든 통화 정리
        WebRTCClient.shared.endAllCalls()
        callManager.removeAllCalls()
        deactivateAudioSession()
    }
}
```

## PushKit: 백그라운드의 마법

일반 푸시는 알림만 보여준다. PushKit은 앱을 깨운다.

### PushKit 설정

```swift
import PushKit

class PushKitManager: NSObject {
    private let voipRegistry = PKPushRegistry(queue: .main)
    
    func setupPushKit() {
        voipRegistry.delegate = self
        voipRegistry.desiredPushTypes = [.voIP]
    }
}

extension PushKitManager: PKPushRegistryDelegate {
    // 토큰 받기
    func pushRegistry(_ registry: PKPushRegistry, 
                     didUpdate pushCredentials: PKPushCredentials, 
                     for type: PKPushType) {
        guard type == .voIP else { return }
        
        let token = pushCredentials.token.map { 
            String(format: "%02.2hhx", $0) 
        }.joined()
        
        print("VoIP Push Token: \(token)")
        
        // 서버에 토큰 전송
        ServerAPI.registerPushToken(token)
    }
    
    // VoIP 푸시 수신 (중요!)
    func pushRegistry(_ registry: PKPushRegistry,
                     didReceiveIncomingPushWith payload: PKPushPayload,
                     for type: PKPushType,
                     completion: @escaping () -> Void) {
        guard type == .voIP else { 
            completion()
            return 
        }
        
        // ⚠️ iOS 13+: 반드시 CallKit을 호출해야 함!
        // 안 그러면 앱이 크래시됨
        
        let callUUID = UUID()
        let handle = payload.dictionaryPayload["handle"] as? String ?? "Unknown"
        let hasVideo = payload.dictionaryPayload["hasVideo"] as? Bool ?? false
        
        // CallKit으로 수신 전화 표시
        let update = CXCallUpdate()
        update.remoteHandle = CXHandle(type: .phoneNumber, value: handle)
        update.hasVideo = hasVideo
        update.localizedCallerName = payload.dictionaryPayload["callerName"] as? String
        
        provider.reportNewIncomingCall(with: callUUID,
                                       update: update) { error in
            if error == nil {
                // 통화 객체 생성
                let call = Call(uuid: callUUID, isOutgoing: false)
                self.callManager.add(call)
            }
            completion()
        }
    }
}
```

### iOS 13+ 주의사항

```swift
// ❌ 잘못된 예: CallKit 없이 PushKit만 사용
func pushRegistry(_ registry: PKPushRegistry,
                 didReceiveIncomingPushWith payload: PKPushPayload,
                 for type: PKPushType) {
    // 그냥 로컬 알림만 보여주기
    showLocalNotification()  // 앱 크래시! 💥
}

// ✅ 올바른 예: 반드시 CallKit 호출
func pushRegistry(_ registry: PKPushRegistry,
                 didReceiveIncomingPushWith payload: PKPushPayload,
                 for type: PKPushType,
                 completion: @escaping () -> Void) {
    // 반드시 reportNewIncomingCall 호출
    provider.reportNewIncomingCall(with: UUID(), 
                                   update: CXCallUpdate()) { _ in
        completion()
    }
}
```

## AVAudioSession: 오디오의 핵심

CallKit과 함께 작동하는 오디오 세션 관리:

```swift
import AVFoundation

class AudioSessionManager {
    
    func configureAudioSession() {
        let session = AVAudioSession.sharedInstance()
        
        do {
            // CallKit을 위한 설정
            try session.setCategory(.playAndRecord,
                                   mode: .voiceChat,
                                   options: [.allowBluetooth, 
                                           .allowBluetoothA2DP,
                                           .defaultToSpeaker])
            
            // 샘플레이트와 버퍼 크기 설정
            try session.setPreferredSampleRate(48000)
            try session.setPreferredIOBufferDuration(0.005) // 5ms
            
            // 활성화
            try session.setActive(true)
            
        } catch {
            print("Audio Session Error: \(error)")
        }
    }
    
    func handleRouteChange() {
        // 오디오 라우트 변경 감지
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(routeChanged),
            name: AVAudioSession.routeChangeNotification,
            object: nil
        )
    }
    
    @objc private func routeChanged(_ notification: Notification) {
        guard let info = notification.userInfo,
              let reason = info[AVAudioSession.routeChangeReasonKey] as? UInt else {
            return
        }
        
        let reasonEnum = AVAudioSession.RouteChangeReason(rawValue: reason)
        
        switch reasonEnum {
        case .newDeviceAvailable:
            print("새 오디오 장치 연결 (이어폰 등)")
            
        case .oldDeviceUnavailable:
            print("오디오 장치 연결 해제")
            // 스피커로 자동 전환
            
        case .categoryChange:
            print("오디오 카테고리 변경")
            
        default:
            break
        }
    }
}
```

## 실전: CallKit + PushKit + WebRTC

모든 것을 통합한 실제 구현:

```swift
class VoIPCallManager: NSObject {
    static let shared = VoIPCallManager()
    
    private let provider: CXProvider
    private let callController = CXCallController()
    private var calls: [UUID: Call] = [:]
    
    // WebRTC
    private let webRTCClient = WebRTCClient()
    
    // PushKit
    private let voipRegistry = PKPushRegistry(queue: .main)
    
    override init() {
        // CallKit 설정
        let config = CXProviderConfiguration()
        config.localizedName = "My VoIP"
        config.supportsVideo = true
        config.maximumCallsPerCallGroup = 1
        config.supportedHandleTypes = [.phoneNumber]
        
        self.provider = CXProvider(configuration: config)
        
        super.init()
        
        provider.setDelegate(self, queue: nil)
        setupPushKit()
    }
    
    // 발신 전화
    func startCall(to number: String, hasVideo: Bool = false) {
        let handle = CXHandle(type: .phoneNumber, value: number)
        let callUUID = UUID()
        
        let startCallAction = CXStartCallAction(call: callUUID, 
                                                handle: handle)
        startCallAction.isVideo = hasVideo
        
        let transaction = CXTransaction(action: startCallAction)
        
        callController.request(transaction) { error in
            if error == nil {
                let call = Call(uuid: callUUID, 
                              isOutgoing: true,
                              handle: number)
                self.calls[callUUID] = call
            }
        }
    }
    
    // 전화 끊기
    func endCall(_ call: Call) {
        let endCallAction = CXEndCallAction(call: call.uuid)
        let transaction = CXTransaction(action: endCallAction)
        
        callController.request(transaction) { error in
            if error == nil {
                self.calls.removeValue(forKey: call.uuid)
            }
        }
    }
}
```

## 백그라운드 모드 처리

```swift
// AppDelegate.swift
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    var backgroundTask: UIBackgroundTaskIdentifier = .invalid
    
    func applicationDidEnterBackground(_ application: UIApplication) {
        // VoIP 앱은 백그라운드에서도 실행 가능
        // 하지만 통화 중이 아니면 suspended 됨
        
        if VoIPCallManager.shared.hasActiveCalls {
            // 통화 중이면 계속 실행
            startBackgroundTask()
        }
    }
    
    func startBackgroundTask() {
        backgroundTask = UIApplication.shared.beginBackgroundTask {
            // 시간 초과 시
            self.endBackgroundTask()
        }
    }
    
    func endBackgroundTask() {
        UIApplication.shared.endBackgroundTask(backgroundTask)
        backgroundTask = .invalid
    }
}
```

## 디버깅과 테스트

### Console 로그 필터링
```bash
# CallKit 로그 보기
log stream --predicate 'subsystem == "com.apple.callkit"'

# PushKit 로그 보기
log stream --predicate 'subsystem == "com.apple.pushkit"'
```

### 시뮬레이터 제한사항
```swift
#if targetEnvironment(simulator)
    print("⚠️ 시뮬레이터에서 불가능:")
    print("- PushKit 토큰 받기")
    print("- 실제 오디오 입출력")
    print("- 일부 CallKit 기능")
    
    // 시뮬레이터용 모의 구현
    mockIncomingCall()
#else
    // 실제 기기 코드
    setupPushKit()
#endif
```

## 일반적인 문제와 해결

### 1. PushKit 토큰을 못 받음
```swift
// 체크리스트:
// ✅ Capabilities에서 Push Notifications 활성화
// ✅ Background Modes에서 Voice over IP 활성화
// ✅ 유효한 APNs 인증서
// ✅ 실제 기기에서 테스트
```

### 2. 수신 전화가 안 보임
```swift
// iOS 13+ 필수 요구사항
func pushRegistry(_ registry: PKPushRegistry,
                 didReceiveIncomingPushWith payload: PKPushPayload,
                 for type: PKPushType,
                 completion: @escaping () -> Void) {
    
    // ⏱ 시간 제한: completion을 빨리 호출해야 함
    let deadline = DispatchTime.now() + .seconds(20)
    
    // 반드시 reportNewIncomingCall 호출
    provider.reportNewIncomingCall(with: UUID(),
                                   update: CXCallUpdate()) { error in
        // 여기서 completion() 호출 필수!
        completion()
    }
    
    // 타임아웃 방지
    DispatchQueue.main.asyncAfter(deadline: deadline) {
        completion()
    }
}
```

### 3. 오디오가 안 들림
```swift
// CallKit 델리게이트에서 오디오 세션 설정
func provider(_ provider: CXProvider, 
             didActivate audioSession: AVAudioSession) {
    // 여기서 오디오 시작
    webRTCClient.startAudio()
}

func provider(_ provider: CXProvider, 
             didDeactivate audioSession: AVAudioSession) {
    // 여기서 오디오 정지
    webRTCClient.stopAudio()
}
```

## 고급 기능

### 통화 그룹 (Conference)
```swift
let config = CXProviderConfiguration()
config.maximumCallGroups = 2
config.maximumCallsPerCallGroup = 5  // 그룹당 최대 5명
```

### 통화 녹음
```swift
// ⚠️ 법적 문제 주의!
// 상대방 동의 필수

class CallRecorder {
    private var audioRecorder: AVAudioRecorder?
    
    func startRecording(for call: Call) {
        // 1. 권한 확인
        AVAudioSession.sharedInstance().requestRecordPermission { granted in
            guard granted else { return }
            
            // 2. 녹음 시작
            let url = self.recordingURL(for: call)
            let settings = [
                AVFormatIDKey: Int(kAudioFormatMPEG4AAC),
                AVSampleRateKey: 44100,
                AVNumberOfChannelsKey: 2,
                AVEncoderAudioQualityKey: AVAudioQuality.high.rawValue
            ]
            
            self.audioRecorder = try? AVAudioRecorder(url: url, 
                                                      settings: settings)
            self.audioRecorder?.record()
        }
    }
}
```

## 성능 최적화

### 메모리 관리
```swift
class CallManager {
    // Weak 참조로 순환 참조 방지
    private weak var delegate: CallManagerDelegate?
    
    // 통화 종료 시 즉시 정리
    func endCall(_ call: Call) {
        calls.removeValue(forKey: call.uuid)
        call.cleanup()  // 명시적 정리
    }
}
```

### 배터리 최적화
```swift
// 통화 중이 아닐 때는 최소한의 리소스 사용
func enterIdleMode() {
    webRTCClient.disconnect()
    audioSession.setActive(false)
    // CPU 사용량 감소
}
```

## 마치며

CallKit과 PushKit은 iOS VoIP 앱의 심장이다.
이 둘을 제대로 이해하고 구현하면, 당신의 앱은 시스템 전화 앱과 구별되지 않을 것이다.

하지만 아직 끝이 아니다.
실제 음성 데이터를 주고받는 네트워킹 프로토콜이 기다리고 있다.

---

## Connections

→ [[L4_networking]] - WebRTC와 네트워킹 프로토콜
→ [[L5_audio_processing]] - 오디오 처리 심화

← [[L2_voip_principles]] - VoIP 원리로 돌아가기
← [[index]] - 목차

---

Level: L3
Date: 2025-08-15
Tags: #ios #callkit #pushkit #framework #voip