# L7: 최적화와 성능

## Core Insight
VoIP 앱의 성능은 배터리 수명과 통화 품질 사이의 절묘한 균형이다. 너무 최적화하면 품질이 떨어지고, 품질에만 집중하면 배터리가 순식간에 닳는다.

---

## 성능 병목 지점 찾기

최적화의 첫 걸음은 측정이다:

```swift
import os.signpost

class PerformanceProfiler {
    private let log = OSLog(subsystem: "com.app.voip", category: "performance")
    private let signpostID = OSSignpostID(log: .default)
    
    // Signpost로 구간 측정
    func measureAudioProcessing() {
        os_signpost(.begin, log: log, name: "Audio Processing", signpostID: signpostID)
        
        // 오디오 처리 코드
        processAudio()
        
        os_signpost(.end, log: log, name: "Audio Processing", signpostID: signpostID)
    }
    
    // 더 자세한 측정
    func profileCallFlow() {
        let metrics = CallMetrics()
        
        // 1. 연결 시간
        let connectStart = CFAbsoluteTimeGetCurrent()
        establishConnection()
        metrics.connectionTime = CFAbsoluteTimeGetCurrent() - connectStart
        
        // 2. 첫 패킷까지 시간
        let firstPacketStart = CFAbsoluteTimeGetCurrent()
        waitForFirstPacket()
        metrics.timeToFirstPacket = CFAbsoluteTimeGetCurrent() - firstPacketStart
        
        // 3. 메모리 사용량
        metrics.memoryUsage = getMemoryUsage()
        
        // 4. CPU 사용률
        metrics.cpuUsage = getCPUUsage()
        
        print("Performance Metrics: \(metrics)")
    }
    
    private func getMemoryUsage() -> Float {
        var info = mach_task_basic_info()
        var count = mach_msg_type_number_t(MemoryLayout<mach_task_basic_info>.size) / 4
        
        let result = withUnsafeMutablePointer(to: &info) {
            $0.withMemoryRebound(to: integer_t.self, capacity: 1) {
                task_info(mach_task_self_,
                         task_flavor_t(MACH_TASK_BASIC_INFO),
                         $0,
                         &count)
            }
        }
        
        return result == KERN_SUCCESS ? Float(info.resident_size) / 1024 / 1024 : 0
    }
}
```

## 배터리 최적화

VoIP 앱은 배터리 킬러가 될 수 있다:

### 1. 적응형 코덱 설정

```swift
class AdaptivePowerManager {
    private var currentPowerMode: PowerMode = .balanced
    
    enum PowerMode {
        case performance  // 최고 품질, 배터리 소모 大
        case balanced     // 균형
        case powerSaving  // 배터리 우선
    }
    
    func adjustForBatteryLevel() {
        let batteryLevel = UIDevice.current.batteryLevel
        let batteryState = UIDevice.current.batteryState
        
        // 배터리 상태에 따른 조정
        if batteryState == .unplugged {
            if batteryLevel < 0.2 {
                currentPowerMode = .powerSaving
                applyPowerSavingSettings()
            } else if batteryLevel < 0.5 {
                currentPowerMode = .balanced
                applyBalancedSettings()
            } else {
                currentPowerMode = .performance
                applyPerformanceSettings()
            }
        } else {
            // 충전 중
            currentPowerMode = .performance
            applyPerformanceSettings()
        }
    }
    
    private func applyPowerSavingSettings() {
        // 낮은 비트레이트
        opus.setBitrate(32000)  // 32 kbps
        
        // 긴 패킷 간격
        opus.setPacketDuration(60)  // 60ms packets
        
        // 비디오 비활성화
        disableVideo()
        
        // CPU 사용량 감소
        echoCanceller.setComplexity(.low)
        noiseReducer.disable()
        
        // 네트워크 최적화
        webrtc.setDataSaverMode(true)
    }
    
    private func applyBalancedSettings() {
        opus.setBitrate(64000)  // 64 kbps
        opus.setPacketDuration(40)  // 40ms
        echoCanceller.setComplexity(.medium)
        noiseReducer.setLevel(.moderate)
    }
    
    private func applyPerformanceSettings() {
        opus.setBitrate(128000)  // 128 kbps
        opus.setPacketDuration(20)  // 20ms
        echoCanceller.setComplexity(.high)
        noiseReducer.setLevel(.maximum)
    }
}
```

### 2. 백그라운드 작업 최적화

```swift
class BackgroundOptimizer {
    private var backgroundTask: UIBackgroundTaskIdentifier = .invalid
    
    func optimizeForBackground() {
        // 백그라운드 진입 시
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(didEnterBackground),
            name: UIApplication.didEnterBackgroundNotification,
            object: nil
        )
    }
    
    @objc private func didEnterBackground() {
        // 통화 중이 아니면 최소 모드
        if !CallManager.shared.hasActiveCalls {
            // 네트워크 연결 유지만
            webrtc.enterKeepAliveMode()
            
            // 불필요한 타이머 정지
            stopNonEssentialTimers()
            
            // 메모리 정리
            clearCaches()
        } else {
            // 통화 중이면 백그라운드 작업 시작
            startBackgroundTask()
            
            // 비디오는 정지
            pauseVideo()
            
            // 오디오 품질 낮춤
            reduceAudioQuality()
        }
    }
    
    private func startBackgroundTask() {
        backgroundTask = UIApplication.shared.beginBackgroundTask {
            self.endBackgroundTask()
        }
    }
}
```

## 메모리 최적화

### 1. 오디오 버퍼 관리

```swift
class AudioBufferManager {
    private let maxBufferSize = 10 * 1024 * 1024  // 10MB
    private var currentBufferSize = 0
    private let bufferQueue = DispatchQueue(label: "audio.buffer")
    
    // 링 버퍼로 메모리 재사용
    class RingBuffer {
        private var buffer: UnsafeMutablePointer<Float>
        private let capacity: Int
        private var writeIndex = 0
        private var readIndex = 0
        
        init(capacity: Int) {
            self.capacity = capacity
            self.buffer = UnsafeMutablePointer<Float>.allocate(capacity: capacity)
        }
        
        func write(_ data: [Float]) {
            let writeCount = min(data.count, capacity - writeIndex)
            
            data.withUnsafeBufferPointer { src in
                buffer.advanced(by: writeIndex).update(from: src.baseAddress!, 
                                                       count: writeCount)
            }
            
            writeIndex = (writeIndex + writeCount) % capacity
        }
        
        func read(count: Int) -> [Float] {
            let readCount = min(count, availableToRead())
            var result = [Float](repeating: 0, count: readCount)
            
            result.withUnsafeMutableBufferPointer { dst in
                buffer.advanced(by: readIndex).copyBytes(to: dst.baseAddress!, 
                                                         count: readCount)
            }
            
            readIndex = (readIndex + readCount) % capacity
            return result
        }
        
        private func availableToRead() -> Int {
            if writeIndex >= readIndex {
                return writeIndex - readIndex
            } else {
                return capacity - readIndex + writeIndex
            }
        }
        
        deinit {
            buffer.deallocate()
        }
    }
}
```

### 2. 이미지/비디오 메모리 관리

```swift
class VideoMemoryOptimizer {
    private let pixelBufferPool: CVPixelBufferPool
    
    init() {
        // 픽셀 버퍼 풀 생성
        let attributes: [String: Any] = [
            kCVPixelBufferPixelFormatTypeKey as String: kCVPixelFormatType_420YpCbCr8BiPlanarVideoRange,
            kCVPixelBufferWidthKey as String: 1280,
            kCVPixelBufferHeightKey as String: 720,
            kCVPixelBufferIOSurfacePropertiesKey as String: [:]
        ]
        
        var pool: CVPixelBufferPool?
        CVPixelBufferPoolCreate(nil, nil, attributes as CFDictionary, &pool)
        self.pixelBufferPool = pool!
    }
    
    func getReusablePixelBuffer() -> CVPixelBuffer? {
        var pixelBuffer: CVPixelBuffer?
        CVPixelBufferPoolCreatePixelBuffer(nil, pixelBufferPool, &pixelBuffer)
        return pixelBuffer
    }
    
    // 텍스처 캐시로 GPU 메모리 효율화
    func createTextureCache() -> CVMetalTextureCache? {
        var textureCache: CVMetalTextureCache?
        CVMetalTextureCacheCreate(nil, nil, MTLCreateSystemDefaultDevice()!, 
                                 nil, &textureCache)
        return textureCache
    }
}
```

## 네트워크 최적화

### 1. 적응형 비트레이트 (ABR)

```swift
class AdaptiveBitrateController {
    private var currentBitrate = 64000
    private let minBitrate = 16000
    private let maxBitrate = 256000
    
    // REMB (Receiver Estimated Maximum Bitrate)
    func adjustBitrateBasedOnNetwork(_ stats: NetworkStats) {
        let packetLoss = stats.packetLossRate
        let rtt = stats.roundTripTime
        let jitter = stats.jitter
        
        // 네트워크 점수 계산
        let networkScore = calculateNetworkScore(packetLoss: packetLoss,
                                                 rtt: rtt,
                                                 jitter: jitter)
        
        // 비트레이트 조정
        if networkScore < 3 {
            // 나쁜 네트워크
            decreaseBitrate(by: 0.5)
        } else if networkScore > 4 {
            // 좋은 네트워크
            increaseBitrate(by: 1.2)
        }
        
        // OPUS 코덱에 적용
        opus.encoder.setBitrate(currentBitrate)
    }
    
    private func calculateNetworkScore(packetLoss: Float, 
                                      rtt: Float, 
                                      jitter: Float) -> Float {
        var score: Float = 5.0
        
        // 패킷 손실 페널티
        if packetLoss > 0.01 { score -= 1 }
        if packetLoss > 0.03 { score -= 1 }
        if packetLoss > 0.05 { score -= 1 }
        
        // RTT 페널티
        if rtt > 150 { score -= 0.5 }
        if rtt > 300 { score -= 1 }
        
        // 지터 페널티
        if jitter > 30 { score -= 0.5 }
        if jitter > 50 { score -= 1 }
        
        return max(1, min(5, score))
    }
}
```

### 2. 네트워크 전환 처리

```swift
class NetworkHandover {
    private let reachability = NetworkReachability()
    
    func handleNetworkChange() {
        reachability.onNetworkChange = { [weak self] networkType in
            switch networkType {
            case .wifi:
                self?.switchToHighQuality()
                
            case .cellular5G:
                self?.switchToMediumQuality()
                
            case .cellular4G:
                self?.switchToMediumQuality()
                
            case .cellular3G:
                self?.switchToLowQuality()
                
            case .none:
                self?.handleNoNetwork()
            }
        }
    }
    
    // ICE Restart로 연결 유지
    func performICERestart() {
        guard let pc = peerConnection else { return }
        
        // 새 ICE 후보 수집
        let config = pc.configuration
        config.iceServers = updateICEServers()
        pc.setConfiguration(config)
        
        // Offer with ICE restart
        let constraints = RTCMediaConstraints(
            mandatoryConstraints: ["IceRestart": "true"],
            optionalConstraints: nil
        )
        
        pc.offer(for: constraints) { sdp, error in
            if let sdp = sdp {
                self.handleICERestartOffer(sdp)
            }
        }
    }
}
```

## CPU 최적화

### 1. SIMD와 벡터 연산

```swift
import Accelerate

class OptimizedDSP {
    // 일반 버전
    func dotProductSlow(_ a: [Float], _ b: [Float]) -> Float {
        var result: Float = 0
        for i in 0..<a.count {
            result += a[i] * b[i]
        }
        return result
    }
    
    // SIMD 최적화 버전
    func dotProductFast(_ a: [Float], _ b: [Float]) -> Float {
        var result: Float = 0
        vDSP_dotpr(a, 1, b, 1, &result, vDSP_Length(a.count))
        return result
    }
    
    // FFT 최적화
    func performFFT(_ input: [Float]) -> [Float] {
        let log2n = vDSP_Length(log2(Float(input.count)))
        guard let fftSetup = vDSP_create_fftsetup(log2n, FFTRadix(kFFTRadix2)) else {
            return []
        }
        defer { vDSP_destroy_fftsetup(fftSetup) }
        
        var real = [Float](input)
        var imag = [Float](repeating: 0, count: input.count)
        var splitComplex = DSPSplitComplex(realp: &real, imagp: &imag)
        
        vDSP_fft_zip(fftSetup, &splitComplex, 1, log2n, FFTDirection(FFT_FORWARD))
        
        var magnitude = [Float](repeating: 0, count: input.count/2)
        vDSP_zvmags(&splitComplex, 1, &magnitude, 1, vDSP_Length(input.count/2))
        
        return magnitude
    }
}
```

### 2. GCD로 병렬 처리

```swift
class ParallelProcessor {
    private let processingQueue = DispatchQueue(
        label: "audio.processing",
        qos: .userInteractive,
        attributes: .concurrent
    )
    
    func processAudioFrames(_ frames: [[Float]]) -> [[Float]] {
        let group = DispatchGroup()
        var results = [[Float]](repeating: [], count: frames.count)
        let lock = NSLock()
        
        for (index, frame) in frames.enumerated() {
            group.enter()
            processingQueue.async {
                // 각 프레임을 병렬로 처리
                let processed = self.processFrame(frame)
                
                lock.lock()
                results[index] = processed
                lock.unlock()
                
                group.leave()
            }
        }
        
        group.wait()
        return results
    }
    
    // Metal로 GPU 활용
    func processWithMetal(_ data: [Float]) -> [Float] {
        guard let device = MTLCreateSystemDefaultDevice(),
              let commandQueue = device.makeCommandQueue(),
              let library = device.makeDefaultLibrary(),
              let function = library.makeFunction(name: "audioProcessing"),
              let pipeline = try? device.makeComputePipelineState(function: function) else {
            return data
        }
        
        // GPU 버퍼 생성
        let inputBuffer = device.makeBuffer(bytes: data, 
                                           length: data.count * MemoryLayout<Float>.size,
                                           options: .storageModeShared)
        
        let outputBuffer = device.makeBuffer(length: data.count * MemoryLayout<Float>.size,
                                            options: .storageModeShared)
        
        // 커맨드 인코딩
        guard let commandBuffer = commandQueue.makeCommandBuffer(),
              let encoder = commandBuffer.makeComputeCommandEncoder() else {
            return data
        }
        
        encoder.setComputePipelineState(pipeline)
        encoder.setBuffer(inputBuffer, offset: 0, index: 0)
        encoder.setBuffer(outputBuffer, offset: 0, index: 1)
        
        let threadsPerGrid = MTLSize(width: data.count, height: 1, depth: 1)
        let threadsPerThreadgroup = MTLSize(width: 256, height: 1, depth: 1)
        
        encoder.dispatchThreads(threadsPerGrid, 
                               threadsPerThreadgroup: threadsPerThreadgroup)
        encoder.endEncoding()
        
        commandBuffer.commit()
        commandBuffer.waitUntilCompleted()
        
        // 결과 읽기
        let result = outputBuffer!.contents().bindMemory(to: Float.self, 
                                                        capacity: data.count)
        return Array(UnsafeBufferPointer(start: result, count: data.count))
    }
}
```

## 시작 시간 최적화

### 1. 지연 로딩

```swift
class LazyInitializer {
    // 무거운 객체들을 lazy로
    private lazy var webrtcClient: WebRTCClient = {
        return WebRTCClient()
    }()
    
    private lazy var audioProcessor: AudioProcessor = {
        return AudioProcessor()
    }()
    
    // 필요할 때만 초기화
    func startCall() {
        // 이제야 WebRTC 초기화
        _ = webrtcClient
        webrtcClient.connect()
    }
}
```

### 2. 프리페칭과 캐싱

```swift
class ResourcePreloader {
    func preloadCriticalResources() {
        DispatchQueue.global(qos: .background).async {
            // STUN 서버 미리 연결
            self.prefetchSTUNServers()
            
            // 자주 쓰는 사운드 미리 로드
            self.preloadSoundEffects()
            
            // 인증서 캐싱
            self.cacheCertificates()
        }
    }
    
    private func prefetchSTUNServers() {
        let stunServers = ["stun.l.google.com:19302",
                          "stun1.l.google.com:19302"]
        
        for server in stunServers {
            // DNS 미리 조회
            performDNSPrefetch(for: server)
        }
    }
}
```

## 프로파일링과 모니터링

```swift
class PerformanceMonitor: ObservableObject {
    @Published var fps: Double = 0
    @Published var cpuUsage: Double = 0
    @Published var memoryUsage: Double = 0
    @Published var networkLatency: Double = 0
    
    private var displayLink: CADisplayLink?
    private var lastTimestamp: CFTimeInterval = 0
    
    func startMonitoring() {
        // FPS 모니터링
        displayLink = CADisplayLink(target: self, 
                                   selector: #selector(updateFPS))
        displayLink?.add(to: .main, forMode: .common)
        
        // 시스템 메트릭 모니터링
        Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { _ in
            self.updateSystemMetrics()
        }
    }
    
    @objc private func updateFPS(_ displayLink: CADisplayLink) {
        if lastTimestamp != 0 {
            let delta = displayLink.timestamp - lastTimestamp
            fps = 1.0 / delta
        }
        lastTimestamp = displayLink.timestamp
    }
    
    private func updateSystemMetrics() {
        cpuUsage = getCurrentCPUUsage()
        memoryUsage = getCurrentMemoryUsage()
        networkLatency = measureNetworkLatency()
    }
}
```

## 최적화 체크리스트

```swift
struct OptimizationChecklist {
    let items = [
        "✓ 프로파일링으로 병목 지점 찾기",
        "✓ 배터리 수준에 따른 적응형 설정",
        "✓ 메모리 풀과 재사용",
        "✓ 네트워크 상태에 따른 품질 조정",
        "✓ SIMD/Metal로 연산 가속",
        "✓ GCD로 병렬 처리",
        "✓ 지연 로딩과 프리페칭",
        "✓ 백그라운드 작업 최소화",
        "✓ 실시간 모니터링",
        "✓ A/B 테스트로 검증"
    ]
}
```

## 마치며

최적화는 끝이 없는 여정이다.
새로운 iOS 버전, 새로운 하드웨어, 새로운 네트워크 기술...

하지만 기억하자:
- 조기 최적화는 모든 악의 근원
- 측정 없는 최적화는 추측
- 사용자가 체감하는 것만이 진짜

다음 레벨에서는 이 모든 것을 앱스토어에 배포하고
실제 사용자들에게 전달하는 방법을 다룬다.

---

## Connections

→ [[L8_deployment]] - 배포와 운영
→ [[L9_philosophy]] - 성능과 경험의 철학

← [[L6_security]] - 보안으로 돌아가기
← [[index]] - 목차

---

Level: L7
Date: 2025-08-15
Tags: #optimization #performance #battery #memory #cpu