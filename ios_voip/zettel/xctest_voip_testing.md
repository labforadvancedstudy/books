# XCTest로 VoIP 테스트

## 개념
iOS VoIP 앱의 통화 로직, 네트워크 처리, CallKit 통합 등을 자동화된 테스트로 검증.

## CallKit 목 테스트
```swift
import XCTest
import CallKit

class CallKitTests: XCTestCase {
    var mockProvider: CXProvider!
    var mockCallController: CXCallController!
    var callManager: CallManager!
    
    override func setUp() {
        super.setUp()
        
        // Mock CallKit 구성
        let config = CXProviderConfiguration(localizedName: "Test VoIP")
        config.maximumCallsPerCallGroup = 1
        config.supportsVideo = false
        
        mockProvider = CXProvider(configuration: config)
        mockCallController = CXCallController()
        callManager = CallManager(provider: mockProvider, callController: mockCallController)
    }
    
    func testIncomingCall() async throws {
        // Given
        let callUUID = UUID()
        let handle = CXHandle(type: .phoneNumber, value: "+1234567890")
        
        // When
        let expectation = XCTestExpectation(description: "Incoming call reported")
        
        callManager.reportIncomingCall(
            uuid: callUUID,
            handle: handle
        ) { error in
            XCTAssertNil(error)
            expectation.fulfill()
        }
        
        // Then
        await fulfillment(of: [expectation], timeout: 5.0)
        XCTAssertEqual(callManager.calls.count, 1)
        XCTAssertEqual(callManager.calls.first?.uuid, callUUID)
    }
}
```

## WebRTC 연결 테스트
```swift
class WebRTCConnectionTests: XCTestCase {
    var peerConnection: RTCPeerConnection!
    var factory: RTCPeerConnectionFactory!
    
    func testICECandidateGathering() async {
        // ICE 후보 수집 테스트
        let expectation = XCTestExpectation(description: "ICE gathering complete")
        var candidates: [RTCIceCandidate] = []
        
        class MockDelegate: NSObject, RTCPeerConnectionDelegate {
            var onCandidate: ((RTCIceCandidate?) -> Void)?
            
            func peerConnection(_ peerConnection: RTCPeerConnection, 
                              didGenerate candidate: RTCIceCandidate?) {
                onCandidate?(candidate)
            }
        }
        
        let delegate = MockDelegate()
        delegate.onCandidate = { candidate in
            if let candidate = candidate {
                candidates.append(candidate)
            } else {
                // Gathering complete
                expectation.fulfill()
            }
        }
        
        peerConnection = createPeerConnection(delegate: delegate)
        
        // Create offer to trigger ICE gathering
        let offer = try await peerConnection.offer(for: RTCMediaConstraints())
        try await peerConnection.setLocalDescription(offer)
        
        await fulfillment(of: [expectation], timeout: 10.0)
        
        // Verify
        XCTAssertGreaterThan(candidates.count, 0)
        XCTAssertTrue(candidates.contains { $0.sdp.contains("typ host") })
    }
}
```

## 오디오 처리 테스트
```swift
class AudioProcessingTests: XCTestCase {
    func testOpusEncoding() throws {
        // Given: PCM 오디오 샘플
        let sampleRate: Double = 48000
        let duration: Double = 1.0
        let frequency: Double = 440.0 // A4 note
        
        let samples = generateSineWave(
            frequency: frequency,
            sampleRate: sampleRate,
            duration: duration
        )
        
        // When: Opus 인코딩
        let encoder = OpusEncoder(sampleRate: Int32(sampleRate))
        let encoded = try encoder.encode(samples)
        
        // Then: 압축률 확인
        let originalSize = samples.count * MemoryLayout<Float>.size
        let encodedSize = encoded.count
        let compressionRatio = Float(originalSize) / Float(encodedSize)
        
        XCTAssertGreaterThan(compressionRatio, 10.0) // 10:1 이상 압축
        XCTAssertLessThan(encodedSize, originalSize / 10)
    }
    
    func testEchoCancellation() {
        let echoFilter = EchoCancellationFilter()
        let input = generateWhiteNoise(samples: 1024)
        let echo = input.map { $0 * 0.3 } // 30% echo
        
        let mixed = zip(input, echo).map { $0 + $1 }
        let filtered = echoFilter.process(mixed, reference: input)
        
        // 에코 제거 효과 측정
        let echoLevel = measureEchoLevel(filtered, reference: input)
        XCTAssertLessThan(echoLevel, 0.1) // 90% 이상 에코 제거
    }
}
```

## 네트워크 시뮬레이션 테스트
```swift
class NetworkSimulationTests: XCTestCase {
    func testPacketLossHandling() async {
        // 패킷 손실 시뮬레이션
        let transport = MockNetworkTransport()
        transport.packetLossRate = 0.1 // 10% 손실
        
        let voipSession = VoIPSession(transport: transport)
        
        // 100개 패킷 전송
        for i in 0..<100 {
            await voipSession.sendPacket(Data([UInt8(i)]))
        }
        
        // 결과 검증
        let stats = await voipSession.getStatistics()
        XCTAssertGreaterThan(stats.packetsLost, 5)
        XCTAssertLessThan(stats.packetsLost, 15) // 10% ± 5%
        
        // FEC로 복구된 패킷 확인
        XCTAssertGreaterThan(stats.packetsRecovered, 0)
    }
    
    func testJitterBufferAdaptation() {
        let jitterBuffer = AdaptiveJitterBuffer()
        let networkJitter = 50.0 // 50ms jitter
        
        // 비규칙한 간격으로 패킷 수신
        for i in 0..<100 {
            let delay = Double.random(in: 0...networkJitter)
            jitterBuffer.addPacket(
                Packet(sequence: i, timestamp: Double(i) * 20 + delay)
            )
        }
        
        // 버퍼 크기 적응 확인
        XCTAssertGreaterThan(jitterBuffer.currentDepth, 30)
        XCTAssertLessThan(jitterBuffer.currentDepth, 100)
    }
}
```

## UI 테스트
```swift
class CallUITests: XCTestCase {
    var app: XCUIApplication!
    
    override func setUp() {
        continueAfterFailure = false
        app = XCUIApplication()
        app.launchArguments = ["--uitesting"]
        app.launch()
    }
    
    func testIncomingCallFlow() {
        // 수신 화면 표시
        let incomingCallView = app.otherElements["IncomingCallView"]
        XCTAssertTrue(incomingCallView.waitForExistence(timeout: 5))
        
        // 수락 버튼 탭
        let acceptButton = app.buttons["AcceptCallButton"]
        acceptButton.tap()
        
        // 통화 화면 표시 확인
        let activeCallView = app.otherElements["ActiveCallView"]
        XCTAssertTrue(activeCallView.exists)
        
        // 통화 시간 표시 확인
        let durationLabel = app.staticTexts["CallDurationLabel"]
        XCTAssertTrue(durationLabel.waitForExistence(timeout: 2))
        
        // 종료 버튼 탭
        let endButton = app.buttons["EndCallButton"]
        endButton.tap()
        
        // 통화 종료 확인
        XCTAssertFalse(activeCallView.exists)
    }
}
```

## 연관 개념
- [[network_link_conditioner]]
- [[memory_leak_detection]]
- [[crash_reporting]]
- [[testflight_deployment]]

## 태그
#testing #xctest #unittest #uitest #automation