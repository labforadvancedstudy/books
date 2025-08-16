# 고급 디버깅 전략: 보이지 않는 버그 추적

## Core Insight
디버깅은 가설 수립과 검증의 과학적 과정이며, 고급 도구와 기법의 조합으로 불가능해 보이는 버그도 추적할 수 있다.

디버깅은 프로그래밍에서 가장 어려운 부분 중 하나다. 코드를 작성하는 것보다 버그를 찾고 수정하는 것이 더 복잡하고 창의적인 사고를 요구한다. iOS 개발에서는 멀티스레딩, 메모리 관리, UI 업데이트 등 복잡한 요소들이 얽혀 있어 디버깅이 더욱 까다롭다.

**체계적 디버깅 접근법:**

1. **문제 재현 (Reproduction)**
```swift
// 재현 가능한 테스트 케이스 작성
func testBugReproduction() {
    // Given: 특정 조건 설정
    let viewModel = UserViewModel()
    viewModel.users = createTestUsers(count: 100)
    
    // When: 문제 상황 실행
    viewModel.deleteUser(at: 50)
    
    // Then: 예상 결과와 실제 결과 비교
    XCTAssertEqual(viewModel.users.count, 99)
}
```

2. **가설 수립 (Hypothesis)**
```swift
// 로깅을 통한 가설 검증
enum DebugContext {
    case userDeletion
    case networkRequest
    case stateUpdate
}

func debugLog(_ message: String, context: DebugContext) {
    #if DEBUG
    print("🐛 [\(context)] \(message) - \(Thread.current)")
    #endif
}
```

**LLDB 고급 기법:**

**조건부 브레이크포인트:**
```bash
# 특정 조건에서만 멈춤
(lldb) breakpoint set --name viewDidLoad --condition 'user.id == 123'

# 히트 카운트 기반 브레이크포인트
(lldb) breakpoint modify --ignore-count 10 1

# 메모리 워치포인트
(lldb) watchpoint set variable self.importantProperty
```

**Python 스크립트 활용:**
```python
# ~/.lldbinit 파일에 추가
command script import ~/lldb_scripts/ios_debug.py

# ios_debug.py
import lldb

def print_view_hierarchy(debugger, command, result, internal_dict):
    target = debugger.GetSelectedTarget()
    process = target.GetProcess()
    thread = process.GetSelectedThread()
    frame = thread.GetSelectedFrame()
    
    # 현재 뷰 계층 구조 출력
    view = frame.FindVariable("self.view")
    result.AppendMessage(str(view.GetChildMemberWithName("recursiveDescription").GetSummary()))

def __lldb_init_module(debugger, internal_dict):
    debugger.HandleCommand('command script add -f ios_debug.print_view_hierarchy pv')
```

**Instruments를 활용한 성능 디버깅:**

**Memory Graph Debugger:**
```swift
// 메모리 리크 감지용 마킹
final class UserViewController: UIViewController {
    deinit {
        print("✅ UserViewController deallocated")
    }
}

// 강한 참조 순환 방지
class ViewModel {
    weak var delegate: ViewModelDelegate?
    
    var completion: (() -> Void)? {
        didSet {
            // 클로저에서 self 캡처 시 약한 참조 사용 확인
            if let completion = completion {
                DispatchQueue.main.async { [weak self] in
                    completion()
                }
            }
        }
    }
}
```

**Time Profiler 활용:**
```swift
import os.signpost

class PerformanceProfiler {
    private let log = OSLog(subsystem: "com.example.app", category: "performance")
    
    func profileOperation<T>(_ name: String, operation: () throws -> T) rethrows -> T {
        os_signpost(.begin, log: log, name: StaticString(name.utf8Start))
        defer { os_signpost(.end, log: log, name: StaticString(name.utf8Start)) }
        return try operation()
    }
}

// 사용법
let profiler = PerformanceProfiler()
let result = profiler.profileOperation("Heavy Calculation") {
    return heavyCalculation()
}
```

**Main Thread Checker 활용:**
```swift
// UI 업데이트 메인 스레드 보장
func updateUI() {
    DispatchQueue.main.async {
        // UI 업데이트 코드
        titleLabel.text = "Updated"
    }
}

// 디버그 모드에서 스레드 검증
#if DEBUG
func verifyMainThread() {
    assert(Thread.isMainThread, "This method must be called on the main thread")
}
#endif
```

**Thread Sanitizer:**
```swift
// 데이터 레이스 감지를 위한 어노테이션
import Darwin

class ThreadSafeCounter {
    private var _value: Int = 0
    private let queue = DispatchQueue(label: "counter")
    
    var value: Int {
        get {
            return queue.sync { _value }
        }
        set {
            queue.sync { _value = newValue }
        }
    }
    
    func increment() {
        queue.sync {
            _value += 1
        }
    }
}
```

**커스텀 디버깅 도구:**

**Debug View Overlay:**
```swift
#if DEBUG
struct DebugOverlay: View {
    @State private var showFPS = false
    @State private var showMemory = false
    
    var body: some View {
        VStack {
            if showFPS {
                FPSView()
            }
            if showMemory {
                MemoryUsageView()
            }
            Spacer()
        }
        .onShake {
            showDebugMenu()
        }
    }
}

extension UIDevice {
    static let deviceDidShakeNotification = Notification.Name(rawValue: "deviceDidShakeNotification")
}

extension UIWindow {
    override func motionEnded(_ motion: UIEvent.EventSubtype, with event: UIEvent?) {
        if motion == .motionShake {
            NotificationCenter.default.post(name: UIDevice.deviceDidShakeNotification, object: nil)
        }
    }
}
#endif
```

**네트워크 디버깅:**
```swift
// 네트워크 요청 로깅
class NetworkLogger: URLProtocol {
    override func startLoading() {
        print("🌐 Request: \(request.url?.absoluteString ?? "Unknown")")
        print("📊 Headers: \(request.allHTTPHeaderFields ?? [:])")
        
        if let body = request.httpBody {
            print("📝 Body: \(String(data: body, encoding: .utf8) ?? "Binary data")")
        }
        
        // 실제 요청 실행
        super.startLoading()
    }
    
    override func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
        if let error = error {
            print("❌ Network Error: \(error)")
        } else {
            print("✅ Network Success")
        }
    }
}

// 디버그 모드에서 활성화
#if DEBUG
URLProtocol.registerClass(NetworkLogger.self)
#endif
```

**메모리 디버깅 도구:**
```swift
// 메모리 사용량 모니터링
class MemoryMonitor {
    static func getCurrentMemoryUsage() -> UInt64 {
        var info = mach_task_basic_info()
        var count = mach_msg_type_number_t(MemoryLayout<mach_task_basic_info>.size)/4
        
        let kerr: kern_return_t = withUnsafeMutablePointer(to: &info) {
            $0.withMemoryRebound(to: integer_t.self, capacity: 1) {
                task_info(mach_task_self_,
                         task_flavor_t(MACH_TASK_BASIC_INFO),
                         $0,
                         &count)
            }
        }
        
        return kerr == KERN_SUCCESS ? info.resident_size : 0
    }
    
    static func logMemoryUsage(tag: String) {
        let usage = getCurrentMemoryUsage()
        print("💾 Memory [\(tag)]: \(ByteCountFormatter.string(fromByteCount: Int64(usage), countStyle: .memory))")
    }
}
```

**크래시 리포트 분석:**
```swift
// 크래시 전 상태 보존
class CrashReporter {
    static func logUserAction(_ action: String) {
        UserDefaults.standard.set(action, forKey: "lastUserAction")
        UserDefaults.standard.set(Date(), forKey: "lastActionTime")
    }
    
    static func logAppState() {
        let state = [
            "memoryUsage": MemoryMonitor.getCurrentMemoryUsage(),
            "activeViewController": getActiveViewController(),
            "networkStatus": NetworkReachability.current.status
        ]
        
        UserDefaults.standard.set(state, forKey: "appStateBeforeCrash")
    }
}

// 앱 시작 시 이전 크래시 확인
func checkForPreviousCrash() {
    if let lastAction = UserDefaults.standard.string(forKey: "lastUserAction") {
        print("🚨 Potential crash after: \(lastAction)")
        // 크래시 리포트 서비스로 전송
    }
}
```

**SwiftUI 디버깅:**
```swift
// SwiftUI 뷰 계층 구조 디버깅
extension View {
    func debugPrint(_ value: Any) -> some View {
        #if DEBUG
        print("🐛 Debug: \(value)")
        #endif
        return self
    }
    
    func debugModifier<T>(_ modifier: T) -> some View where T: ViewModifier {
        #if DEBUG
        return self.modifier(modifier)
        #else
        return self
        #endif
    }
}

// 뷰 경계 표시
struct DebugBorderModifier: ViewModifier {
    func body(content: Content) -> some View {
        content
            .border(Color.red, width: 1)
            .background(Color.yellow.opacity(0.1))
    }
}
```

디버깅은 예술과 과학의 결합이다. 체계적인 접근법과 강력한 도구들을 조합하면, 가장 복잡한 버그도 결국 그 정체를 드러낸다. 핵심은 포기하지 않는 끈기와 창의적 사고다.

## Connections
→ [[297-app-store-optimization-aso]]
→ [[298-analytics-user-behavior-tracking]]
← [[295-continuous-integration-xcode-cloud]]
← [[294-xcode-productivity-techniques]]

---
Level: L3
Date: 2025-08-16
Tags: #debugging #lldb #instruments #memory-debugging #performance #crash-analysis