# L4: 네트워킹과 프로토콜

## Core Insight
WebRTC는 브라우저를 위해 태어났지만, 모바일 실시간 통신의 표준이 되었다. P2P 연결의 복잡함을 추상화하여 개발자에게 간단한 API를 제공한다.

---

## WebRTC: 실시간 통신의 완전체

WebRTC는 세 가지 핵심 API로 구성된다:

```
MediaStream: 오디오/비디오 캡처
RTCPeerConnection: P2P 연결 관리  
RTCDataChannel: 임의 데이터 전송
```

iOS에서 GoogleWebRTC 프레임워크를 사용한다:

```swift
import WebRTC

class WebRTCClient {
    private let factory: RTCPeerConnectionFactory
    private var peerConnection: RTCPeerConnection?
    private var localAudioTrack: RTCAudioTrack?
    private var localVideoTrack: RTCVideoTrack?
    
    init() {
        // WebRTC 초기화
        RTCInitializeSSL()
        
        // 팩토리 생성
        let encoderFactory = RTCDefaultVideoEncoderFactory()
        let decoderFactory = RTCDefaultVideoDecoderFactory()
        
        factory = RTCPeerConnectionFactory(
            encoderFactory: encoderFactory,
            decoderFactory: decoderFactory
        )
    }
}
```

## 연결 수립: The Dance of ICE and SDP

WebRTC 연결은 복잡한 춤과 같다:

### 1. Offer/Answer 모델

```swift
extension WebRTCClient {
    // 발신자: Offer 생성
    func createOffer() async throws -> RTCSessionDescription {
        guard let pc = peerConnection else { throw WebRTCError.noPeerConnection }
        
        let constraints = RTCMediaConstraints(
            mandatoryConstraints: [
                "OfferToReceiveAudio": "true",
                "OfferToReceiveVideo": "true"
            ],
            optionalConstraints: nil
        )
        
        return try await withCheckedThrowingContinuation { continuation in
            pc.offer(for: constraints) { sdp, error in
                if let error = error {
                    continuation.resume(throwing: error)
                } else if let sdp = sdp {
                    // 로컬 SDP 설정
                    pc.setLocalDescription(sdp) { error in
                        if let error = error {
                            continuation.resume(throwing: error)
                        } else {
                            continuation.resume(returning: sdp)
                        }
                    }
                }
            }
        }
    }
    
    // 수신자: Answer 생성
    func createAnswer() async throws -> RTCSessionDescription {
        guard let pc = peerConnection else { throw WebRTCError.noPeerConnection }
        
        let constraints = RTCMediaConstraints(
            mandatoryConstraints: [:],
            optionalConstraints: nil
        )
        
        return try await withCheckedThrowingContinuation { continuation in
            pc.answer(for: constraints) { sdp, error in
                if let error = error {
                    continuation.resume(throwing: error)
                } else if let sdp = sdp {
                    pc.setLocalDescription(sdp) { error in
                        if let error = error {
                            continuation.resume(throwing: error)
                        } else {
                            continuation.resume(returning: sdp)
                        }
                    }
                }
            }
        }
    }
}
```

### 2. ICE Candidates 수집

```swift
extension WebRTCClient: RTCPeerConnectionDelegate {
    func peerConnection(_ peerConnection: RTCPeerConnection, 
                       didGenerate candidate: RTCIceCandidate) {
        // ICE candidate 발견
        let candidateData = IceCandidateData(
            sdp: candidate.sdp,
            sdpMLineIndex: candidate.sdpMLineIndex,
            sdpMid: candidate.sdpMid
        )
        
        // 시그널링 서버로 전송
        signalingClient.send(.iceCandidate(candidateData))
    }
    
    func peerConnection(_ peerConnection: RTCPeerConnection, 
                       didChange newState: RTCIceConnectionState) {
        switch newState {
        case .new:
            print("ICE: 새 연결")
        case .checking:
            print("ICE: 연결 확인 중...")
        case .connected:
            print("ICE: 연결됨! 🎉")
        case .completed:
            print("ICE: 최적 경로 확정")
        case .failed:
            print("ICE: 연결 실패 😢")
            handleConnectionFailure()
        case .disconnected:
            print("ICE: 연결 끊김")
        case .closed:
            print("ICE: 연결 종료")
        case .count:
            break
        @unknown default:
            break
        }
    }
}
```

## STUN/TURN: NAT 통과의 영웅들

대부분의 디바이스는 NAT 뒤에 있다. 직접 연결이 불가능하다.

### STUN (Session Traversal Utilities for NAT)

```swift
struct STUNServer {
    let urls = [
        "stun:stun.l.google.com:19302",
        "stun:stun1.l.google.com:19302",
        "stun:stun2.l.google.com:19302"
    ]
}

// PeerConnection 설정에 STUN 서버 추가
func createPeerConnection() -> RTCPeerConnection? {
    let config = RTCConfiguration()
    
    // STUN 서버들
    config.iceServers = [
        RTCIceServer(urlStrings: STUNServer().urls)
    ]
    
    // 연결 정책
    config.iceTransportPolicy = .all
    config.bundlePolicy = .maxBundle
    config.rtcpMuxPolicy = .require
    
    let constraints = RTCMediaConstraints(
        mandatoryConstraints: nil,
        optionalConstraints: ["DtlsSrtpKeyAgreement": "true"]
    )
    
    return factory.peerConnection(with: config, 
                                 constraints: constraints, 
                                 delegate: self)
}
```

### TURN (Traversal Using Relays around NAT)

STUN이 실패하면 TURN이 나선다:

```swift
struct TURNServer {
    let url = "turn:turn.example.com:3478"
    let username = "user"
    let password = "pass"
}

func createPeerConnectionWithTURN() -> RTCPeerConnection? {
    let config = RTCConfiguration()
    
    config.iceServers = [
        // STUN 먼저 시도
        RTCIceServer(urlStrings: ["stun:stun.l.google.com:19302"]),
        
        // TURN 백업
        RTCIceServer(urlStrings: [TURNServer().url],
                    username: TURNServer().username,
                    credential: TURNServer().password)
    ]
    
    // TURN 강제 (테스트용)
    // config.iceTransportPolicy = .relay
    
    return factory.peerConnection(with: config, 
                                 constraints: nil, 
                                 delegate: self)
}
```

## 시그널링: 연결의 중매인

WebRTC는 시그널링 방법을 정의하지 않는다. 우리가 선택한다.

### WebSocket 시그널링

```swift
import Starscream

class SignalingClient: WebSocketDelegate {
    private var socket: WebSocket?
    private let decoder = JSONDecoder()
    private let encoder = JSONEncoder()
    
    func connect(to url: URL) {
        var request = URLRequest(url: url)
        request.timeoutInterval = 5
        
        socket = WebSocket(request: request)
        socket?.delegate = self
        socket?.connect()
    }
    
    // 메시지 전송
    func send<T: Encodable>(_ message: SignalingMessage<T>) {
        guard let socket = socket,
              let data = try? encoder.encode(message),
              let string = String(data: data, encoding: .utf8) else {
            return
        }
        
        socket.write(string: string)
    }
    
    // WebSocket 델리게이트
    func didReceive(event: WebSocketEvent, client: WebSocket) {
        switch event {
        case .connected(let headers):
            print("WebSocket 연결됨: \(headers)")
            
        case .disconnected(let reason, let code):
            print("WebSocket 연결 끊김: \(reason) (\(code))")
            
        case .text(let string):
            handleMessage(string)
            
        case .binary(let data):
            handleBinaryMessage(data)
            
        case .error(let error):
            print("WebSocket 에러: \(error?.localizedDescription ?? "")")
            
        default:
            break
        }
    }
}

// 시그널링 메시지 타입
enum SignalingMessage<T: Codable>: Codable {
    case join(roomId: String)
    case offer(sdp: String, from: String)
    case answer(sdp: String, from: String)
    case ice(candidate: T, from: String)
    case leave(from: String)
    
    enum CodingKeys: String, CodingKey {
        case type, payload
    }
}
```

### 시그널링 플로우

```swift
class CallSignalingFlow {
    private let signaling = SignalingClient()
    private let webrtc = WebRTCClient()
    
    // 발신자 플로우
    func initiateCall(to recipient: String) async {
        // 1. 방 생성/참여
        signaling.send(.join(roomId: generateRoomId()))
        
        // 2. Offer 생성
        let offer = try await webrtc.createOffer()
        
        // 3. Offer 전송
        signaling.send(.offer(sdp: offer.sdp, from: myId))
        
        // 4. Answer 대기
        // (signaling delegate에서 처리)
    }
    
    // 수신자 플로우
    func handleOffer(_ offer: String, from sender: String) async {
        // 1. Remote description 설정
        let sdp = RTCSessionDescription(type: .offer, sdp: offer)
        try await webrtc.setRemoteDescription(sdp)
        
        // 2. Answer 생성
        let answer = try await webrtc.createAnswer()
        
        // 3. Answer 전송
        signaling.send(.answer(sdp: answer.sdp, from: myId))
    }
}
```

## 미디어 스트림 관리

### 오디오 트랙

```swift
extension WebRTCClient {
    func createAudioTrack() -> RTCAudioTrack {
        let audioConstraints = RTCMediaConstraints(
            mandatoryConstraints: nil,
            optionalConstraints: nil
        )
        
        let audioSource = factory.audioSource(with: audioConstraints)
        
        // 오디오 처리 설정
        audioSource.volume = 1.0
        
        let audioTrack = factory.audioTrack(
            with: audioSource,
            trackId: "audio-\(UUID().uuidString)"
        )
        
        return audioTrack
    }
    
    func setupLocalAudio() {
        localAudioTrack = createAudioTrack()
        
        // PeerConnection에 추가
        if let track = localAudioTrack {
            peerConnection?.add(track, streamIds: ["stream"])
        }
    }
}
```

### 비디오 트랙

```swift
extension WebRTCClient {
    func createVideoTrack() -> RTCVideoTrack? {
        let videoSource = factory.videoSource()
        
        // 카메라 캡처 설정
        let capturer = RTCCameraVideoCapturer(delegate: videoSource)
        
        // 카메라 선택
        guard let camera = RTCCameraVideoCapturer.captureDevices().first(where: {
            $0.position == .front
        }) else { return nil }
        
        // 해상도와 FPS 설정
        let format = camera.formats.first { format in
            let dimension = CMVideoFormatDescriptionGetDimensions(
                format.formatDescription
            )
            return dimension.width == 1280 && dimension.height == 720
        }
        
        let fps = format?.videoSupportedFrameRateRanges.first?.maxFrameRate ?? 30
        
        capturer.startCapture(with: camera,
                             format: format ?? camera.activeFormat,
                             fps: Int(fps))
        
        let videoTrack = factory.videoTrack(
            with: videoSource,
            trackId: "video-\(UUID().uuidString)"
        )
        
        return videoTrack
    }
}
```

## 데이터 채널: 메타데이터 전송

음성/영상 외에도 데이터를 보낼 수 있다:

```swift
class DataChannelManager {
    private var dataChannel: RTCDataChannel?
    
    func createDataChannel(on peerConnection: RTCPeerConnection) {
        let config = RTCDataChannelConfiguration()
        config.isOrdered = true
        config.isNegotiated = false
        
        dataChannel = peerConnection.dataChannel(
            forLabel: "metadata",
            configuration: config
        )
        
        dataChannel?.delegate = self
    }
    
    // 메시지 전송
    func send(_ message: String) {
        guard let data = message.data(using: .utf8) else { return }
        
        let buffer = RTCDataBuffer(data: data, isBinary: false)
        dataChannel?.sendData(buffer)
    }
    
    // 구조화된 데이터 전송
    func sendJSON<T: Encodable>(_ object: T) {
        guard let data = try? JSONEncoder().encode(object) else { return }
        
        let buffer = RTCDataBuffer(data: data, isBinary: true)
        dataChannel?.sendData(buffer)
    }
}

extension DataChannelManager: RTCDataChannelDelegate {
    func dataChannel(_ dataChannel: RTCDataChannel, 
                    didReceiveMessageWith buffer: RTCDataBuffer) {
        if buffer.isBinary {
            // 바이너리 데이터 처리
            handleBinaryData(buffer.data)
        } else {
            // 텍스트 데이터 처리
            if let message = String(data: buffer.data, encoding: .utf8) {
                handleTextMessage(message)
            }
        }
    }
    
    func dataChannelDidChangeState(_ dataChannel: RTCDataChannel) {
        switch dataChannel.readyState {
        case .connecting:
            print("데이터 채널: 연결 중")
        case .open:
            print("데이터 채널: 열림")
            sendInitialMetadata()
        case .closing:
            print("데이터 채널: 닫히는 중")
        case .closed:
            print("데이터 채널: 닫힘")
        @unknown default:
            break
        }
    }
}
```

## 네트워크 품질 모니터링

### RTCStats 활용

```swift
class NetworkQualityMonitor {
    private var statsTimer: Timer?
    
    func startMonitoring(for peerConnection: RTCPeerConnection) {
        statsTimer = Timer.scheduledTimer(withTimeInterval: 1.0, 
                                         repeats: true) { _ in
            self.collectStats(from: peerConnection)
        }
    }
    
    private func collectStats(from pc: RTCPeerConnection) {
        pc.statistics { report in
            self.processStatsReport(report)
        }
    }
    
    private func processStatsReport(_ report: RTCStatisticsReport) {
        var stats = CallStatistics()
        
        for (_, value) in report.statistics {
            if value.type == "inbound-rtp" {
                // 수신 통계
                if let packetsLost = value.values["packetsLost"] as? Int,
                   let packetsReceived = value.values["packetsReceived"] as? Int {
                    stats.packetLossRate = Float(packetsLost) / 
                                          Float(packetsReceived + packetsLost)
                }
                
                if let jitter = value.values["jitter"] as? Double {
                    stats.jitter = Float(jitter * 1000) // ms
                }
                
                if let bytesReceived = value.values["bytesReceived"] as? Int {
                    stats.bytesReceived = bytesReceived
                }
            }
            
            if value.type == "candidate-pair" && 
               value.values["state"] as? String == "succeeded" {
                // RTT (Round Trip Time)
                if let rtt = value.values["currentRoundTripTime"] as? Double {
                    stats.rtt = Float(rtt * 1000) // ms
                }
            }
        }
        
        updateUI(with: stats)
    }
}

struct CallStatistics {
    var packetLossRate: Float = 0
    var jitter: Float = 0
    var rtt: Float = 0
    var bytesReceived: Int = 0
    var bytesSent: Int = 0
    
    var qualityScore: Float {
        // 간단한 품질 점수 계산
        var score: Float = 5.0
        
        // 패킷 손실 페널티
        score -= packetLossRate * 20
        
        // 지터 페널티
        if jitter > 30 { score -= 1 }
        if jitter > 50 { score -= 1 }
        
        // RTT 페널티
        if rtt > 150 { score -= 0.5 }
        if rtt > 300 { score -= 1 }
        
        return max(1, min(5, score))
    }
}
```

## 적응형 비트레이트

네트워크 상황에 따라 품질 조절:

```swift
class AdaptiveBitrateController {
    private var currentBitrate = 128000 // 128 kbps
    private let minBitrate = 32000      // 32 kbps
    private let maxBitrate = 256000     // 256 kbps
    
    func adjustBitrate(based on stats: CallStatistics) {
        if stats.packetLossRate > 0.05 {
            // 5% 이상 손실: 비트레이트 감소
            decreaseBitrate()
        } else if stats.packetLossRate < 0.01 && stats.rtt < 100 {
            // 좋은 상태: 비트레이트 증가
            increaseBitrate()
        }
    }
    
    private func decreaseBitrate() {
        currentBitrate = max(minBitrate, Int(Float(currentBitrate) * 0.8))
        applyBitrate()
    }
    
    private func increaseBitrate() {
        currentBitrate = min(maxBitrate, Int(Float(currentBitrate) * 1.2))
        applyBitrate()
    }
    
    private func applyBitrate() {
        // 오디오 인코더 비트레이트 조절
        if let sender = peerConnection?.senders.first(where: { 
            $0.track?.kind == "audio" 
        }) {
            let parameters = sender.parameters
            parameters.encodings.first?.maxBitrateBps = NSNumber(value: currentBitrate)
            sender.parameters = parameters
        }
    }
}
```

## 보안: DTLS와 SRTP

WebRTC는 기본적으로 암호화된다:

```swift
// DTLS (Datagram Transport Layer Security) 설정
func configureSecurity(for config: inout RTCConfiguration) {
    // DTLS 필수
    config.enableDtlsSrtp = true
    
    // 인증서 생성
    let certParams = RTCCertificate.generateParams()
    certParams.expires = 100000 // 초 단위
    
    RTCCertificate.generate(with: certParams) { certificate in
        guard let cert = certificate else { return }
        config.certificate = cert
    }
}
```

## 실전 구현: 완전한 WebRTC 클라이언트

```swift
class CompleteWebRTCClient: NSObject {
    // 싱글톤
    static let shared = CompleteWebRTCClient()
    
    // WebRTC
    private let factory: RTCPeerConnectionFactory
    private var peerConnection: RTCPeerConnection?
    
    // 미디어
    private var localAudioTrack: RTCAudioTrack?
    private var localVideoTrack: RTCVideoTrack?
    private var remoteAudioTrack: RTCAudioTrack?
    private var remoteVideoTrack: RTCVideoTrack?
    
    // 시그널링
    private let signaling = SignalingClient()
    
    // 상태
    private(set) var connectionState: RTCPeerConnectionState = .new
    
    override init() {
        // 초기화
        RTCInitializeSSL()
        
        let encoderFactory = RTCDefaultVideoEncoderFactory()
        let decoderFactory = RTCDefaultVideoDecoderFactory()
        
        factory = RTCPeerConnectionFactory(
            encoderFactory: encoderFactory,
            decoderFactory: decoderFactory
        )
        
        super.init()
    }
    
    // 통화 시작
    func startCall(to recipient: String, 
                  hasVideo: Bool = false) async throws {
        // 1. PeerConnection 생성
        createPeerConnection()
        
        // 2. 로컬 미디어 추가
        await setupLocalMedia(hasVideo: hasVideo)
        
        // 3. Offer 생성
        let offer = try await createOffer()
        
        // 4. 시그널링
        signaling.send(.offer(sdp: offer.sdp, from: myId))
        
        // 5. Answer 대기 (delegate에서 처리)
    }
    
    // 통화 수신
    func answerCall(offer: RTCSessionDescription,
                   hasVideo: Bool = false) async throws {
        // 1. PeerConnection 생성
        createPeerConnection()
        
        // 2. 로컬 미디어 추가
        await setupLocalMedia(hasVideo: hasVideo)
        
        // 3. Remote description 설정
        try await setRemoteDescription(offer)
        
        // 4. Answer 생성
        let answer = try await createAnswer()
        
        // 5. 시그널링
        signaling.send(.answer(sdp: answer.sdp, from: myId))
    }
    
    // 통화 종료
    func endCall() {
        // 1. 미디어 정지
        localAudioTrack?.isEnabled = false
        localVideoTrack?.isEnabled = false
        
        // 2. PeerConnection 종료
        peerConnection?.close()
        peerConnection = nil
        
        // 3. 시그널링 종료
        signaling.send(.leave(from: myId))
        signaling.disconnect()
    }
}
```

## 디버깅 도구

### Chrome WebRTC Internals

```swift
// SDP를 로그로 출력하여 chrome://webrtc-internals에서 분석
func logSDP(_ sdp: RTCSessionDescription) {
    print("=== SDP ===")
    print("Type: \(sdp.type.rawValue)")
    print(sdp.sdp)
    print("==========")
}
```

### Wireshark로 패킷 분석

```bash
# STUN/TURN 패킷 필터
udp.port == 3478

# RTP 패킷 필터  
rtp

# DTLS 핸드셰이크
dtls
```

## 마치며

네트워킹은 VoIP의 혈관이다. WebRTC는 이 복잡한 혈관계를 우아하게 추상화한다.

하지만 네트워크로 전송되는 것은 결국 오디오 데이터다.
다음 레벨에서는 이 오디오를 어떻게 처리하고 최적화하는지 알아볼 것이다.

---

## Connections

→ [[L5_audio_processing]] - 실시간 음성 처리
→ [[L6_security]] - 보안과 암호화

← [[L3_ios_frameworks]] - iOS 프레임워크로 돌아가기
← [[index]] - 목차

---

Level: L4
Date: 2025-08-15
Tags: #webrtc #networking #signaling #ice #stun #turn