# 메모리 누수 추적

## 개념
VoIP 앱에서 장시간 통화 시 발생할 수 있는 메모리 누수를 Instruments와 코드 분석으로 검출.

## Instruments를 이용한 누수 검출
```swift
// 메모리 누수 테스트 코드
class LeakDetectionTest {
    func detectLeaksInCallSession() {
        // Instruments Leaks 템플릿으로 프로파일링
        autoreleasepool {
            // 통화 세션 생성
            let session = VoIPCallSession()
            session.startCall(to: "+1234567890")
            
            // 30초 통화
            Thread.sleep(forTimeInterval: 30)
            
            session.endCall()
            // session이 해제되어야 함
        }
        
        // 메모리 스냅샷 비교
        let memoryBefore = getMemoryUsage()
        performCallCycle()
        let memoryAfter = getMemoryUsage()
        
        XCTAssertLessThan(memoryAfter - memoryBefore, 1024 * 1024) // 1MB 이하
    }
}
```

## 순환 참조 검출
```swift
class RetainCycleDetector {
    // Weak 참조 패턴
    class CallDelegate {
        weak var callManager: CallManager?
        var strongCallback: (() -> Void)?
        
        deinit {
            print("CallDelegate deallocated") // 디버그용
        }
    }
    
    // 순환 참조 방지
    class CallManager {
        var delegates: [WeakBox<CallDelegate>] = []
        
        func addDelegate(_ delegate: CallDelegate) {
            delegates.append(WeakBox(delegate))
            
            // 클로저 내 self 캡처 방지
            delegate.strongCallback = { [weak self] in
                self?.handleCallback()
            }
        }
    }
    
    // Weak 래퍼
    class WeakBox<T: AnyObject> {
        weak var value: T?
        init(_ value: T) {
            self.value = value
        }
    }
}
```

## 오디오 버퍼 누수 추적
```swift
class AudioBufferLeakDetector {
    private var allocatedBuffers: Set<UnsafeMutableRawPointer> = []
    
    func allocateAudioBuffer(size: Int) -> UnsafeMutableRawPointer {
        let buffer = UnsafeMutableRawPointer.allocate(
            byteCount: size,
            alignment: MemoryLayout<Float>.alignment
        )
        
        // 할당된 버퍼 추적
        allocatedBuffers.insert(buffer)
        
        #if DEBUG
        print("Allocated buffer: \(buffer), total: \(allocatedBuffers.count)")
        #endif
        
        return buffer
    }
    
    func deallocateAudioBuffer(_ buffer: UnsafeMutableRawPointer) {
        buffer.deallocate()
        allocatedBuffers.remove(buffer)
        
        #if DEBUG
        print("Deallocated buffer: \(buffer), remaining: \(allocatedBuffers.count)")
        #endif
    }
    
    func checkForLeaks() {
        if !allocatedBuffers.isEmpty {
            print("WARNING: \(allocatedBuffers.count) audio buffers not deallocated!")
            
            // Debug 빌드에서 crash
            #if DEBUG
            fatalError("Audio buffer leak detected")
            #endif
        }
    }
    
    deinit {
        checkForLeaks()
    }
}
```

## 메모리 사용량 모니터링
```swift
class MemoryMonitor {
    private var timer: Timer?
    private var baseline: Float = 0
    
    func startMonitoring() {
        baseline = getCurrentMemoryUsage()
        
        timer = Timer.scheduledTimer(withTimeInterval: 5.0, repeats: true) { _ in
            let current = self.getCurrentMemoryUsage()
            let delta = current - self.baseline
            
            if delta > 10 { // 10MB 초과
                self.reportPotentialLeak(delta: delta)
            }
            
            // 특정 임계치 초과 시 경고
            if current > 100 { // 100MB
                self.sendMemoryWarning()
            }
        }
    }
    
    private func getCurrentMemoryUsage() -> Float {
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

## WebRTC 메모리 최적화
```swift
class WebRTCMemoryOptimizer {
    func optimizeVideoMemory() {
        // 비디오 비활성화 시 메모리 해제
        RTCCleanupVideoMemory()
        
        // 비디오 렇더 풀 크기 제한
        RTCSetVideoDecoderPoolSize(2)
        
        // 오디오 버퍼 풀 크기
        RTCSetAudioBufferPoolSize(10)
    }
    
    func releaseUnusedResources() {
        // 사용하지 않는 코덱 해제
        RTCDefaultVideoDecoderFactory().releaseUnusedDecoders()
        
        // 캐시 정리
        URLCache.shared.removeAllCachedResponses()
        
        // 이미지 캐시 정리
        SDImageCache.shared.clearMemory()
    }
}
```

## 연관 개념
- [[memory_management_arc]]
- [[cpu_optimization]]
- [[crash_reporting]]

## 태그
#memory #leak #debugging #instruments #profiling