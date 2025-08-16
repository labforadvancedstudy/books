# 네트워크 시뮬레이션 (Network Link Conditioner)

## 개념
다양한 네트워크 환경을 시뮬레이션하여 VoIP 앱의 성능과 안정성을 테스트. 저대역폭, 높은 지연, 패킷 손실 등 재현.

## Network Link Conditioner 설정
```swift
// 프리셋 프로파일
struct NetworkProfile {
    let name: String
    let downlinkBandwidth: Int  // Kbps
    let uplinkBandwidth: Int    // Kbps
    let downlinkLatency: Int    // ms
    let uplinkLatency: Int      // ms
    let downlinkPacketLoss: Float  // %
    let uplinkPacketLoss: Float    // %
    
    static let profiles = [
        NetworkProfile(
            name: "3G",
            downlinkBandwidth: 780,
            uplinkBandwidth: 330,
            downlinkLatency: 100,
            uplinkLatency: 100,
            downlinkPacketLoss: 0,
            uplinkPacketLoss: 0
        ),
        NetworkProfile(
            name: "Poor WiFi",
            downlinkBandwidth: 1000,
            uplinkBandwidth: 1000,
            downlinkLatency: 50,
            uplinkLatency: 50,
            downlinkPacketLoss: 5.0,
            uplinkPacketLoss: 5.0
        ),
        NetworkProfile(
            name: "Edge",
            downlinkBandwidth: 240,
            uplinkBandwidth: 200,
            downlinkLatency: 400,
            uplinkLatency: 440,
            downlinkPacketLoss: 0,
            uplinkPacketLoss: 0
        )
    ]
}
```

## 커스텀 네트워크 시뮬레이터
```swift
class NetworkSimulator {
    private var bandwidth: Int = Int.max
    private var latency: TimeInterval = 0
    private var packetLossRate: Float = 0
    private var jitter: TimeInterval = 0
    
    func simulateNetwork(profile: NetworkProfile) {
        self.bandwidth = profile.uplinkBandwidth
        self.latency = TimeInterval(profile.uplinkLatency) / 1000.0
        self.packetLossRate = profile.uplinkPacketLoss / 100.0
    }
    
    func sendPacket(_ data: Data, completion: @escaping (Data?) -> Void) {
        // 패킷 손실 시뮬레이션
        if Float.random(in: 0...1) < packetLossRate {
            completion(nil)
            return
        }
        
        // 지연 시뮬레이션
        let delay = latency + (jitter * Double.random(in: -1...1))
        DispatchQueue.global().asyncAfter(deadline: .now() + delay) {
            // 대역폭 제한
            let transmissionTime = Double(data.count * 8) / Double(self.bandwidth * 1000)
            Thread.sleep(forTimeInterval: transmissionTime)
            
            completion(data)
        }
    }
}
```

## 적응형 비트레이트 테스트
```swift
class AdaptiveBitrateTest {
    func testBitrateAdaptation() {
        let simulator = NetworkSimulator()
        let voipCodec = AdaptiveOpusCodec()
        
        // 네트워크 상태 변화 시나리오
        let scenarios = [
            (time: 0, profile: NetworkProfile.profiles[0]),    // Good
            (time: 10, profile: NetworkProfile.profiles[1]),   // Poor
            (time: 20, profile: NetworkProfile.profiles[2]),   // Very Poor
            (time: 30, profile: NetworkProfile.profiles[0])    // Recovery
        ]
        
        for scenario in scenarios {
            DispatchQueue.main.asyncAfter(deadline: .now() + scenario.time) {
                simulator.simulateNetwork(profile: scenario.profile)
                
                // 코덱이 자동 조정되는지 확인
                let currentBitrate = voipCodec.getCurrentBitrate()
                print("Time \(scenario.time)s: Bitrate = \(currentBitrate) bps")
                
                // 품질 측정
                let mos = self.measureCallQuality()
                XCTAssertGreaterThan(mos, 3.0) // 최소 품질 유지
            }
        }
    }
}
```

## 실시간 네트워크 모니터링
```swift
class NetworkMonitor {
    private var timer: Timer?
    private var stats: NetworkStats = NetworkStats()
    
    struct NetworkStats {
        var bandwidth: Double = 0
        var latency: Double = 0
        var packetLoss: Double = 0
        var jitter: Double = 0
        
        var quality: NetworkQuality {
            if packetLoss > 5 || latency > 500 {
                return .poor
            } else if packetLoss > 2 || latency > 200 {
                return .fair
            } else if packetLoss > 0.5 || latency > 100 {
                return .good
            } else {
                return .excellent
            }
        }
    }
    
    func startMonitoring() {
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { _ in
            self.updateNetworkStats()
        }
    }
    
    private func updateNetworkStats() {
        // RTT 측정
        measureRTT { rtt in
            self.stats.latency = rtt
        }
        
        // 패킷 손실 계산
        let sent = getRTPPacketsSent()
        let received = getRTPPacketsReceived()
        self.stats.packetLoss = Double(sent - received) / Double(sent) * 100
        
        // Jitter 계산
        self.stats.jitter = calculateJitter()
        
        // 대역폭 추정
        self.stats.bandwidth = estimateBandwidth()
    }
}
```

## 연관 개념
- [[xctest_voip_testing]]
- [[bandwidth_adaptation]]
- [[jitter_buffer]]
- [[packet_loss_concealment]]

## 태그
#testing #network #simulation #performance #debugging