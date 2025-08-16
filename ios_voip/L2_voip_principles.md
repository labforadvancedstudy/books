# 레벨 2: VoIP 원리 - "목소리가 비트가 되는 여정"

*2024년 2월. 첫 프로토타입 데모 실패.*

PM: "왜 이렇게 끊기고 지연되죠?"
나: "네트워크가..."
PM: "카톡은 안 그런데요?"
나: (아, 이 사람이 카카오의 인프라를 모르는구나)

## 음성이 데이터가 되는 과정 (2024년 버전)

### 1단계: 아날로그 → 디지털 (Sampling)

```swift
import AVFoundation
import Accelerate

class AudioCapture {
    private let audioEngine = AVAudioEngine()
    private let inputNode: AVAudioInputNode
    
    // 현대적 설정: 48kHz가 표준
    private let format = AVAudioFormat(
        commonFormat: .pcmFormatFloat32,
        sampleRate: 48000,  // 48kHz (과거 44.1kHz에서 업그레이드)
        channels: 1,        // 모노 (스테레오는 불필요)
        interleaved: false
    )!
    
    func startCapture() async throws {
        // iOS 17+ 새로운 권한 체크
        let session = AVAudioSession.sharedInstance()
        try await session.requestRecordPermission()
        
        // 마이크 입력 설정
        inputNode = audioEngine.inputNode
        
        // 실시간 오디오 처리
        inputNode.installTap(
            onBus: 0,
            bufferSize: 256,  // 낮은 레이턴시 (5.3ms @ 48kHz)
            format: format
        ) { [weak self] buffer, time in
            Task {
                await self?.processAudioBuffer(buffer, at: time)
            }
        }
        
        try audioEngine.start()
    }
    
    private func processAudioBuffer(
        _ buffer: AVAudioPCMBuffer,
        at time: AVAudioTime
    ) async {
        // 1. 원시 PCM 데이터
        guard let channelData = buffer.floatChannelData else { return }
        let frameCount = Int(buffer.frameLength)
        
        // 2. Voice Activity Detection (VAD)
        let hasVoice = await detectVoiceActivity(channelData[0], frameCount)
        
        if hasVoice {
            // 3. 전처리: 노이즈 제거, 에코 제거
            await preprocessAudio(channelData[0], frameCount)
            
            // 4. 인코딩 (Opus 선호)
            let encodedData = await encodeWithOpus(channelData[0], frameCount)
            
            // 5. 패킷화
            let packet = createRTPPacket(encodedData, timestamp: time)
            
            // 6. 전송
            await transmitPacket(packet)
        }
    }
}
```

### 나이퀴스트 정리 실제 적용

```swift
// 왜 48kHz인가?
struct SamplingTheory {
    // 인간 가청 주파수: 20Hz ~ 20kHz
    static let humanHearingRange = 20...20000
    
    // 나이퀴스트 정리: 샘플링 레이트 >= 2 * 최대 주파수
    static let nyquistRate = 20000 * 2  // 40kHz
    
    // 실제 사용 레이트
    static let practicalRates = [
        8000,   // 전화 품질 (협대역)
        16000,  // 광대역 음성
        48000   // 고품질 (현대 표준)
    ]
    
    // 왜 48kHz?
    static let why48kHz = """
    1. 20kHz까지 완벽 재현 (나이퀴스트 만족)
    2. 비디오 프레임레이트와 호환 (48000 / 1000 = 48)
    3. 하드웨어 최적화됨
    4. Opus 코덱 기본값
    """
}
```

## 2단계: 압축 (Codec) - 2024년 현실

### Opus: 현재 최고의 선택

```swift
import OpusCodec  // 실제로는 C 라이브러리 브리징

class OpusEncoder {
    private var encoder: OpaquePointer?
    
    init() throws {
        // Opus 인코더 생성
        var error: Int32 = 0
        encoder = opus_encoder_create(
            48000,      // 샘플 레이트
            1,          // 채널 (모노)
            OPUS_APPLICATION_VOIP,  // VoIP 최적화
            &error
        )
        
        guard error == OPUS_OK else {
            throw CodecError.initFailed(error)
        }
        
        // 2024년 최적 설정
        opus_encoder_ctl(encoder, OPUS_SET_BITRATE(24000))  // 24kbps
        opus_encoder_ctl(encoder, OPUS_SET_VBR(1))          // 가변 비트레이트
        opus_encoder_ctl(encoder, OPUS_SET_DTX(1))          // 무음 감지
        opus_encoder_ctl(encoder, OPUS_SET_INBAND_FEC(1))   // 전방 오류 정정
        opus_encoder_ctl(encoder, OPUS_SET_PACKET_LOSS_PERC(10))  // 10% 손실 가정
    }
    
    func encode(_ pcmData: UnsafePointer<Float>, frameSize: Int) -> Data? {
        let maxPacketSize = 4000
        var outputBuffer = [UInt8](repeating: 0, count: maxPacketSize)
        
        // Float PCM을 Int16으로 변환 (Opus 요구사항)
        var int16Buffer = [Int16](repeating: 0, count: frameSize)
        vDSP_vfixr16(
            pcmData, 1,
            &int16Buffer, 1,
            vDSP_Length(frameSize)
        )
        
        let encodedBytes = opus_encode(
            encoder,
            &int16Buffer,
            Int32(frameSize),
            &outputBuffer,
            Int32(maxPacketSize)
        )
        
        guard encodedBytes > 0 else { return nil }
        
        return Data(bytes: outputBuffer, count: Int(encodedBytes))
    }
}

// 실제 코덱 비교 (2024년 기준)
struct CodecComparison {
    enum Codec {
        case opus
        case aac
        case g711
        case g722
        
        var stats: CodecStats {
            switch self {
            case .opus:
                return CodecStats(
                    bitrate: "6-510 kbps",
                    latency: "5-25ms",
                    quality: "최고",
                    complexity: "중간",
                    usage: "Discord, WhatsApp, Zoom"
                )
            case .aac:
                return CodecStats(
                    bitrate: "24-320 kbps",
                    latency: "20-50ms",
                    quality: "높음",
                    complexity: "높음",
                    usage: "FaceTime, Apple Music"
                )
            case .g711:
                return CodecStats(
                    bitrate: "64 kbps 고정",
                    latency: "0.125ms",
                    quality: "전화 수준",
                    complexity: "매우 낮음",
                    usage: "전통 VoIP, SIP"
                )
            case .g722:
                return CodecStats(
                    bitrate: "48-64 kbps",
                    latency: "2ms",
                    quality: "광대역",
                    complexity: "낮음",
                    usage: "HD Voice"
                )
            }
        }
    }
}
```

## 3단계: 패킷화 (RTP) - 실제 구현

```swift
// RTP (Real-time Transport Protocol) 패킷 구조
struct RTPPacket {
    // RTP 헤더 (12 bytes)
    var version: UInt8 = 2           // RTP 버전
    var padding: Bool = false         // 패딩 여부
    var extension: Bool = false       // 확장 헤더
    var csrcCount: UInt8 = 0         // CSRC 개수
    var marker: Bool = false          // 마커 비트
    var payloadType: UInt8 = 111     // Opus = 111
    var sequenceNumber: UInt16        // 시퀀스 번호
    var timestamp: UInt32             // 타임스탬프
    var ssrc: UInt32                  // 동기화 소스
    
    // 페이로드
    var payload: Data
    
    func serialize() -> Data {
        var data = Data()
        
        // 첫 번째 바이트: V(2), P(1), X(1), CC(4)
        var byte1: UInt8 = (version << 6)
        byte1 |= (padding ? 1 : 0) << 5
        byte1 |= (extension ? 1 : 0) << 4
        byte1 |= csrcCount & 0x0F
        data.append(byte1)
        
        // 두 번째 바이트: M(1), PT(7)
        var byte2: UInt8 = (marker ? 1 : 0) << 7
        byte2 |= payloadType & 0x7F
        data.append(byte2)
        
        // 시퀀스 번호 (2 bytes, big-endian)
        data.append(UInt8((sequenceNumber >> 8) & 0xFF))
        data.append(UInt8(sequenceNumber & 0xFF))
        
        // 타임스탬프 (4 bytes, big-endian)
        data.append(contentsOf: withUnsafeBytes(of: timestamp.bigEndian) { 
            Array($0) 
        })
        
        // SSRC (4 bytes, big-endian)
        data.append(contentsOf: withUnsafeBytes(of: ssrc.bigEndian) { 
            Array($0) 
        })
        
        // 페이로드 추가
        data.append(payload)
        
        return data
    }
}

// 실제 패킷 생성 및 전송
class RTPStreamer {
    private var sequenceNumber: UInt16 = 0
    private let ssrc = UInt32.random(in: 0..<UInt32.max)
    private var timestamp: UInt32 = 0
    
    func createPacket(from audioData: Data) -> RTPPacket {
        defer {
            sequenceNumber = sequenceNumber &+ 1  // 오버플로우 허용
            timestamp += 960  // 20ms @ 48kHz
        }
        
        return RTPPacket(
            sequenceNumber: sequenceNumber,
            timestamp: timestamp,
            ssrc: ssrc,
            payload: audioData
        )
    }
}
```

## 4단계: 전송 - UDP vs TCP vs QUIC (2024)

### 프로토콜 선택의 현실

```swift
// 2024년 프로토콜 비교
enum TransportProtocol {
    case udp        // 전통적 선택
    case tcp        // 신뢰성 필요시
    case quic       // 현대적 대안
    case webTransport  // 웹 호환성
    
    var characteristics: ProtocolCharacteristics {
        switch self {
        case .udp:
            return ProtocolCharacteristics(
                latency: "최소",
                reliability: "없음",
                congestionControl: "없음",
                holBlocking: false,
                implementation: "간단",
                useCase: "실시간 미디어"
            )
        case .tcp:
            return ProtocolCharacteristics(
                latency: "높음",
                reliability: "보장",
                congestionControl: "있음",
                holBlocking: true,  // Head-of-Line Blocking
                implementation: "표준",
                useCase: "시그널링"
            )
        case .quic:
            return ProtocolCharacteristics(
                latency: "낮음",
                reliability: "선택적",
                congestionControl: "개선됨",
                holBlocking: false,
                implementation: "복잡",
                useCase: "차세대 VoIP"
            )
        case .webTransport:
            return ProtocolCharacteristics(
                latency: "낮음",
                reliability: "선택적",
                congestionControl: "QUIC 기반",
                holBlocking: false,
                implementation: "웹 친화적",
                useCase: "브라우저 호환"
            )
        }
    }
}

// QUIC 구현 예시 (iOS 16+)
import Network

class QUICTransport {
    private var connection: NWConnection?
    
    func connect(to endpoint: NWEndpoint) async throws {
        // QUIC 파라미터 설정
        let parameters = NWParameters.quic(alpn: ["voip/1.0"])
        
        // 보안 설정
        let tlsOptions = NWProtocolQUIC.Options()
        tlsOptions.direction = .bidirectional
        tlsOptions.idleTimeout = 30  // 30초
        
        parameters.defaultProtocolStack.applicationProtocols.insert(
            tlsOptions, at: 0
        )
        
        connection = NWConnection(to: endpoint, using: parameters)
        
        // 연결 상태 모니터링
        connection?.stateUpdateHandler = { [weak self] state in
            switch state {
            case .ready:
                print("QUIC 연결 성공")
                Task {
                    await self?.startStreaming()
                }
            case .failed(let error):
                print("QUIC 연결 실패: \(error)")
            default:
                break
            }
        }
        
        connection?.start(queue: .main)
    }
    
    private func startStreaming() async {
        // QUIC 스트림 생성 (멀티플렉싱)
        let audioStream = connection?.multiplexGroup?.createStream()
        let controlStream = connection?.multiplexGroup?.createStream()
        
        // 각 스트림별로 다른 우선순위 설정 가능
        audioStream?.setPriority(.high)
        controlStream?.setPriority(.low)
    }
}
```

## 지터 버퍼: 패킷 도착 순서 문제 해결

```swift
// 적응형 지터 버퍼 구현
class AdaptiveJitterBuffer {
    private var buffer: [UInt16: Data] = [:]  // 시퀀스 번호: 데이터
    private var playoutDelay: TimeInterval = 0.040  // 초기 40ms
    private var statistics = JitterStatistics()
    
    // 네트워크 상태에 따라 버퍼 크기 자동 조절
    func adjustBufferSize() {
        let jitter = statistics.calculateJitter()
        let packetLoss = statistics.packetLossRate
        
        if jitter < 10 && packetLoss < 0.01 {
            // 네트워크 좋음: 버퍼 줄이기 (레이턴시 개선)
            playoutDelay = max(0.020, playoutDelay - 0.005)
        } else if jitter > 30 || packetLoss > 0.05 {
            // 네트워크 나쁨: 버퍼 늘리기 (안정성 개선)
            playoutDelay = min(0.200, playoutDelay + 0.010)
        }
    }
    
    func insertPacket(_ packet: RTPPacket) {
        buffer[packet.sequenceNumber] = packet.payload
        statistics.recordPacket(packet)
        
        // 적응형 조절
        if statistics.packetCount % 100 == 0 {
            adjustBufferSize()
        }
    }
    
    func getNextFrame() -> Data? {
        let targetSequence = statistics.expectedSequence
        
        if let data = buffer[targetSequence] {
            buffer.removeValue(forKey: targetSequence)
            statistics.expectedSequence += 1
            return data
        } else {
            // 패킷 손실: PLC (Packet Loss Concealment) 적용
            return applyPacketLossConcealment()
        }
    }
    
    private func applyPacketLossConcealment() -> Data? {
        // 이전 프레임 반복 or 보간
        return statistics.lastGoodFrame
    }
}

struct JitterStatistics {
    var packetCount: Int = 0
    var lostPackets: Int = 0
    var expectedSequence: UInt16 = 0
    var lastGoodFrame: Data?
    var arrivalTimes: [TimeInterval] = []
    
    var packetLossRate: Double {
        guard packetCount > 0 else { return 0 }
        return Double(lostPackets) / Double(packetCount)
    }
    
    func calculateJitter() -> Double {
        guard arrivalTimes.count > 1 else { return 0 }
        
        var jitter: Double = 0
        for i in 1..<arrivalTimes.count {
            let diff = abs(arrivalTimes[i] - arrivalTimes[i-1] - 0.020)
            jitter += diff
        }
        
        return jitter / Double(arrivalTimes.count - 1) * 1000  // ms
    }
}
```

## NAT 통과: STUN/TURN 실제 구현

```swift
// STUN (Session Traversal Utilities for NAT)
class STUNClient {
    private let stunServer = "stun.l.google.com:19302"
    
    struct STUNMessage {
        let type: MessageType
        let transactionID: Data  // 96-bit
        let attributes: [Attribute]
        
        enum MessageType: UInt16 {
            case bindingRequest = 0x0001
            case bindingResponse = 0x0101
        }
        
        struct Attribute {
            let type: AttributeType
            let value: Data
            
            enum AttributeType: UInt16 {
                case mappedAddress = 0x0001
                case xorMappedAddress = 0x0020
            }
        }
    }
    
    func discoverPublicAddress() async throws -> (String, UInt16) {
        // STUN 바인딩 요청 생성
        let transactionID = Data((0..<12).map { _ in 
            UInt8.random(in: 0...255) 
        })
        
        let request = STUNMessage(
            type: .bindingRequest,
            transactionID: transactionID,
            attributes: []
        )
        
        // UDP 소켓으로 전송
        let response = try await sendSTUNRequest(request)
        
        // XOR-MAPPED-ADDRESS 파싱
        guard let addressAttr = response.attributes.first(where: { 
            $0.type == .xorMappedAddress 
        }) else {
            throw STUNError.noMappedAddress
        }
        
        let (ip, port) = parseXORMappedAddress(addressAttr.value)
        return (ip, port)
    }
}

// TURN (Traversal Using Relays around NAT) - 최후의 수단
class TURNClient {
    private let turnServer = "turn.example.com:3478"
    private let username = "user"
    private let password = "pass"
    
    func allocateRelay() async throws -> RelayAddress {
        // TURN은 인증 필요
        let auth = createLongTermCredential(username, password)
        
        // Allocate 요청
        let request = TURNMessage(
            type: .allocate,
            credential: auth,
            requestedTransport: .udp
        )
        
        let response = try await sendTURNRequest(request)
        
        return RelayAddress(
            ip: response.relayedAddress,
            port: response.relayedPort,
            lifetime: response.lifetime
        )
    }
    
    // 실제 비용 계산
    func calculateMonthlyCost(
        avgCallDuration: TimeInterval,
        dailyCalls: Int,
        dataRate: Double  // kbps
    ) -> Double {
        let bytesPerSecond = dataRate * 125  // kbps to bytes/s
        let monthlyBytes = bytesPerSecond * avgCallDuration * 
                          Double(dailyCalls) * 30
        let monthlyGB = monthlyBytes / 1_000_000_000
        
        // AWS 요금: $0.09/GB (실제 2024년 요금)
        return monthlyGB * 0.09
    }
}
```

## 실제 네트워크 조건별 최적화

```swift
// 적응형 비트레이트 (ABR)
class AdaptiveBitrateController {
    private var currentBitrate: Int = 24000  // 24kbps 시작
    private let minBitrate = 6000           // 6kbps
    private let maxBitrate = 64000          // 64kbps
    
    func adjustBitrate(based on: NetworkConditions) -> Int {
        switch on.quality {
        case .excellent:  // 5G, WiFi
            currentBitrate = min(maxBitrate, currentBitrate + 4000)
            
        case .good:  // LTE
            // 현재 상태 유지
            break
            
        case .fair:  // 3G
            currentBitrate = max(16000, currentBitrate - 2000)
            
        case .poor:  // 2G, 혼잡
            currentBitrate = max(minBitrate, currentBitrate - 4000)
        }
        
        // Opus 코덱에 적용
        opus_encoder_ctl(encoder, OPUS_SET_BITRATE(currentBitrate))
        
        return currentBitrate
    }
}

struct NetworkConditions {
    let bandwidth: Double     // Mbps
    let latency: Double      // ms
    let packetLoss: Double   // 0-1
    let jitter: Double       // ms
    
    var quality: Quality {
        if bandwidth > 10 && latency < 50 && packetLoss < 0.01 {
            return .excellent
        } else if bandwidth > 1 && latency < 150 && packetLoss < 0.03 {
            return .good
        } else if bandwidth > 0.3 && latency < 300 && packetLoss < 0.05 {
            return .fair
        } else {
            return .poor
        }
    }
    
    enum Quality {
        case excellent, good, fair, poor
    }
}
```

## 중국 Great Firewall 우회 전략 (실전)

```swift
// 중국에서 작동하는 VoIP 구현
class ChinaCompatibleVoIP {
    // 1. 구글 STUN 서버 사용 불가
    private let stunServers = [
        "stun.qq.com:3478",        // Tencent
        "stun.ali.com:3478",       // Alibaba
        "stun.baidu.com:3478"      // Baidu
    ]
    
    // 2. WebRTC 대신 자체 프로토콜
    func useCustomProtocol() {
        // WebRTC는 구글 서비스 의존성 때문에 문제
        // 대신 직접 구현하거나 Agora SDK 사용
    }
    
    // 3. 도메인 프론팅
    func setupDomainFronting() -> URLRequest {
        var request = URLRequest(url: URL(string: "https://cdn.example.com")!)
        request.setValue("actual-server.com", forHTTPHeaderField: "Host")
        return request
    }
    
    // 4. 포트 선택
    let allowedPorts = [
        443,   // HTTPS
        8080,  // HTTP 대체
        3478,  // STUN/TURN
        5349   // STUNS/TURNS
    ]
    
    // 5. 프로토콜 난독화
    func obfuscateTraffic(_ data: Data) -> Data {
        // XOR with random key
        let key = Data([0x42, 0x69, 0x13, 0x37])
        return data.enumerated().map { index, byte in
            byte ^ key[index % key.count]
        }
    }
}
```

## 실패 사례와 교훈

### 실패 1: TCP 사용

```swift
// 잘못된 접근
class TCPVoIP {
    // TCP로 음성 전송 시도
    func sendAudioOverTCP() {
        // 문제점:
        // 1. Head-of-Line Blocking
        // 2. 재전송으로 인한 지연 누적
        // 3. 150ms 넘어가면 대화 불가능
    }
}

// 교훈: 실시간 미디어는 반드시 UDP 기반
```

### 실패 2: 지터 버퍼 없이 구현

```swift
// 초기 구현 (실패)
class NaivePlayer {
    func playPacket(_ packet: Data) {
        // 도착하는 대로 바로 재생
        audioPlayer.play(packet)  // 끊김, 순서 뒤바뀜
    }
}

// 교훈: 지터 버퍼는 필수, 40-80ms가 적정
```

### 실패 3: 고정 비트레이트

```swift
// 잘못된 접근
let fixedBitrate = 64000  // 항상 64kbps

// 문제:
// - 3G에서 끊김
// - WiFi에서 품질 낭비
// - 데이터 요금 폭탄

// 교훈: 반드시 적응형 비트레이트 구현
```

## 파인만식 설명

### "왜 영상 통화보다 음성 통화가 더 어려운가?"

영상은 1-2초 버퍼링해도 괜찮다. 유튜브 보면서 1초 늦게 봐도 문제없다.
하지만 전화는? 0.15초만 늦어도 대화가 불가능하다.

"여보세요?" (150ms 후) "네?" 
이미 대화 리듬이 깨진다.

### "왜 UDP를 쓰는가?"

택배(TCP) vs 편지(UDP) 비유:
- 택배: 분실되면 다시 보냄, 순서 보장, 느림
- 편지: 분실되면 그만, 순서 뒤바뀔 수 있음, 빠름

전화에서 "안녕하세요"를 "안...녕하...세요"로 듣는 것보다
"안X하세요"로 듣는 게 낫다. 빠진 건 뇌가 채운다.

## 다음 레벨 예고

레벨 3에서는 iOS 프레임워크 실전:
- CallKit 완벽 정복
- PushKit 함정들
- AVAudioSession 전쟁
- 백그라운드 모드 생존법

"카톡처럼"이 얼마나 어려운지 알게 될 것이다.