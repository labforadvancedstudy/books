# 성능 최적화의 본질: 사용자 시간에 대한 존중

## Core Insight
성능은 기능이 아니라 예의다. 느린 앱은 사용자의 생명을 조금씩 훔치는 것이다. 1초의 지연 × 100만 사용자 = 11.5일의 인류 시간 손실.

## The Physics of Performance

First principles로 분해:
1. **에너지 보존**: 모든 연산은 배터리를 소모한다
2. **시간의 유한성**: 사용자의 인내심은 2초가 한계
3. **열역학 제2법칙**: 무질서(메모리 누수)는 자연적으로 증가한다
4. **관성의 법칙**: 한번 느려진 앱은 계속 느려진다

## The 16ms Rule

화면 주사율 60Hz = 프레임당 16.67ms
ProMotion 120Hz = 프레임당 8.33ms

```swift
// 인간 지각의 한계
struct PerceptionThreshold {
    static let instantaneous = 100.ms  // 즉각적으로 느껴짐
    static let noticeable = 1.second   // 지연을 인지
    static let frustration = 10.seconds // 포기 시작
    
    // 매 프레임이 약속이다
    func renderFrame() async {
        let deadline = DispatchTime.now() + .milliseconds(16)
        
        await doWork()
        
        if DispatchTime.now() > deadline {
            // 약속을 어겼다. 사용자는 버벅임을 느낀다.
            metrics.recordFrameDrop()
        }
    }
}
```

## Battery as Life Force

배터리는 모바일 기기의 생명력이다. 1%의 배터리는 비상시 3분의 통화 시간이다.

```swift
actor BatteryRespect {
    // 에너지 예산
    struct EnergyBudget {
        let critical: [Work] = []      // 반드시 해야 할 일
        let important: [Work] = []     // 중요하지만 연기 가능
        let nice: [Work] = []          // 전원 연결시에만
        
        func schedule() async {
            let batteryLevel = await UIDevice.current.batteryLevel
            let isCharging = await UIDevice.current.batteryState == .charging
            
            switch (batteryLevel, isCharging) {
            case (_, true):
                // 충전 중: 모든 작업 수행
                await performAll()
            case (0.5..., false):
                // 배터리 충분: 중요 작업까지
                await perform(critical + important)
            case (0.2..<0.5, false):
                // 배터리 절약 모드
                await performOnly(critical)
            case (..<0.2, false):
                // 생존 모드
                await suspendAll()
            }
        }
    }
}
```

## Memory: The Hidden Cost

메모리는 무료가 아니다. 
- 할당 = CPU 시간
- 유지 = 전력 소모
- 스왑 = 디스크 I/O = 극심한 배터리 소모

```swift
// 메모리 존중의 아키텍처
class MemoryConsciousCache<T> {
    private var cache: [Key: T] = [:]
    private var costs: [Key: Int] = [:]
    private let maxCost: Int
    
    func set(_ object: T, for key: Key, cost: Int) {
        // 가치 있는 것만 메모리에
        if cost > expectedValue(for: key) {
            return // 보관할 가치가 없다
        }
        
        // 메모리 압박시 즉각 해제
        NotificationCenter.default.addObserver(
            forName: UIApplication.didReceiveMemoryWarningNotification
        ) { _ in
            self.evictLeastValuable()
        }
    }
}
```

## The Lazy Principle

지연 로딩은 예의가 아니라 필수다.

```swift
// 잘못된 접근: 모든 것을 미리 로드
class EagerViewController {
    init() {
        loadAllImages()      // 10MB 메모리
        prefetchAllData()    // 5초 지연
        initializeEverything() // 배터리 2% 소모
    }
}

// 올바른 접근: Just-In-Time
class LazyViewController {
    // 보이는 것만 로드
    private lazy var visibleContent = loadVisible()
    
    // 스크롤 예측
    func scrollViewWillEndDragging(_ scrollView: UIScrollView, 
                                  targetContentOffset: CGPoint) {
        let predictedIndexPath = calculateDestination(targetContentOffset)
        Task {
            await prefetchContent(at: predictedIndexPath)
        }
    }
}
```

## Network: The Expensive Reality

네트워크 요청 한 번 = 
- Radio 활성화: 2초
- 유지 전력: 1.5W
- Tail time: 10초 (추가 전력 소모)

```swift
// 배치 처리의 지혜
actor NetworkOptimizer {
    private var pendingRequests: [Request] = []
    
    func request(_ req: Request) async {
        if req.priority == .immediate {
            await performNow(req)
        } else {
            pendingRequests.append(req)
            await batchIfReady()
        }
    }
    
    private func batchIfReady() async {
        // 모아서 한 번에
        if pendingRequests.count >= 5 || 
           timeSinceLastBatch > 30.seconds {
            await performBatch(pendingRequests)
            pendingRequests.removeAll()
        }
    }
}
```

## Animations: The Performance Theater

애니메이션은 성능을 숨기는 마법이다.

```swift
// 지각된 성능 > 실제 성능
extension UIView {
    func loadWithDelight() {
        // 즉각적인 피드백
        self.showSkeleton()
        
        // 점진적 로드
        Task {
            let content = await fetchContent()
            await self.revealGracefully(content)
        }
    }
    
    func revealGracefully(_ content: Content) async {
        // 스켈레톤에서 실제 컨텐츠로 부드럽게 전환
        await UIView.animate(withDuration: 0.3) {
            self.skeleton.alpha = 0
            self.content.alpha = 1
        }
    }
}
```

## Profiling: The Truth Serum

측정하지 않으면 최적화할 수 없다.

```swift
// 성능 진실의 순간
class PerformanceReality {
    func measureTruth() {
        // Time Profiler: CPU의 진실
        os_signpost(.begin, log: .performance, name: "CriticalPath")
        defer { os_signpost(.end, log: .performance, name: "CriticalPath") }
        
        // Allocations: 메모리의 진실
        let before = memory_usage()
        performOperation()
        let after = memory_usage()
        
        // Energy Log: 배터리의 진실
        let impact = ProcessInfo.processInfo.thermalState
    }
}
```

## Background Tasks: The Invisible Courtesy

백그라운드 작업은 사용자가 자는 동안 요정이 집안일을 하는 것과 같다.

```swift
// 배려심 있는 백그라운드 작업
class BackgroundCourtesy {
    func scheduleConsiderately() {
        BGTaskScheduler.shared.register(
            forTaskWithIdentifier: "cleanup",
            using: nil
        ) { task in
            // 충전 중이고 WiFi 연결시에만
            task.requiresNetworkConnectivity = true
            task.requiresExternalPower = true
            
            await self.performHousekeeping()
        }
    }
}
```

## The Perception Hack

사용자는 절대 시간이 아닌 체감 시간을 경험한다.

```swift
struct PerceptionOptimizer {
    // Progressive Loading
    func loadProgressively(_ content: Content) async {
        // 1. 즉각: 캐시된 내용 표시 (0ms)
        showCached()
        
        // 2. 빠름: 필수 요소 로드 (100ms)
        await loadCritical()
        
        // 3. 보통: 보조 요소 로드 (500ms)
        await loadSecondary()
        
        // 4. 느림: 선택적 요소 로드 (2s)
        await loadOptional()
    }
}
```

## The Ultimate Performance Test

앱을 iPhone 6S에서 Low Power Mode로 실행해보라.
그래도 부드럽다면 진짜 최적화된 것이다.

## Connections
→ [[086-instruments-deep-dive]]
→ [[087-memory-management-patterns]]
→ [[088-energy-efficiency-guide]]
→ [[064-performance-as-feature]]
← [[001-ios-app-essence]]

---
Level: L8
Date: 2025-08-15
Tags: #ios #performance #optimization #battery #memory #efficiency