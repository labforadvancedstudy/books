# 메모리 관리 엣지 케이스: ARC의 함정과 해결책

## Core Insight
ARC는 대부분의 메모리 관리를 자동화하지만, 강한 참조 순환, 비동기 작업, 그리고 C/Objective-C 브릿징에서는 여전히 수동 개입이 필요하다.

ARC(Automatic Reference Counting)는 iOS 개발을 혁명적으로 단순화했다. 하지만 자동이라는 말에 속아서는 안 된다. ARC가 해결하지 못하는 메모리 문제들이 여전히 존재하고, 이런 엣지 케이스들을 이해하지 못하면 메모리 리크와 크래시에 직면하게 된다.

**강한 참조 순환의 미묘한 패턴들:**

**1. 클로저에서의 강한 참조 순환:**
```swift
class DataManager {
    var data: [String] = []
    var completionHandler: (() -> Void)?
    
    func fetchData() {
        // 🚨 강한 참조 순환 - self가 클로저를 소유하고, 클로저가 self를 캡처
        self.completionHandler = {
            self.processData() // self를 강하게 참조
        }
        
        // ✅ 해결책 1: weak self 사용
        self.completionHandler = { [weak self] in
            self?.processData()
        }
        
        // ✅ 해결책 2: unowned self (self가 nil이 될 수 없다고 확신할 때)
        self.completionHandler = { [unowned self] in
            self.processData()
        }
    }
    
    private func processData() {
        print("Processing \(data.count) items")
    }
}

// 더 복잡한 경우: 중첩된 클로저
class NetworkManager {
    func fetchAndProcess(completion: @escaping (Result<Data, Error>) -> Void) {
        URLSession.shared.dataTask(with: URL(string: "https://api.example.com")!) { [weak self] data, response, error in
            guard let self = self else { return }
            
            if let error = error {
                completion(.failure(error))
                return
            }
            
            // 🚨 중첩된 클로저에서 또 다른 강한 참조
            self.processInBackground(data) { processedData in
                // 여기서도 self를 캡처해야 하는가?
                DispatchQueue.main.async { [weak self] in
                    self?.handleProcessedData(processedData)
                    completion(.success(processedData))
                }
            }
        }.resume()
    }
    
    private func processInBackground(_ data: Data?, completion: @escaping (Data) -> Void) {
        // 백그라운드 처리
    }
    
    private func handleProcessedData(_ data: Data) {
        // 처리된 데이터 핸들링
    }
}
```

**2. Delegate 패턴에서의 메모리 리크:**
```swift
protocol CustomViewDelegate: AnyObject {
    func customViewDidTap(_ view: CustomView)
}

class CustomView: UIView {
    // ✅ delegate는 반드시 weak로 선언
    weak var delegate: CustomViewDelegate?
    
    private var tapGesture: UITapGestureRecognizer?
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupTapGesture()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        setupTapGesture()
    }
    
    private func setupTapGesture() {
        tapGesture = UITapGestureRecognizer(target: self, action: #selector(handleTap))
        addGestureRecognizer(tapGesture!)
    }
    
    @objc private func handleTap() {
        delegate?.customViewDidTap(self)
    }
    
    deinit {
        // 🚨 제스처 인식기 정리 필요
        if let gesture = tapGesture {
            removeGestureRecognizer(gesture)
        }
        print("CustomView deallocated")
    }
}

class ViewController: UIViewController, CustomViewDelegate {
    private var customView: CustomView?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        customView = CustomView(frame: CGRect(x: 0, y: 0, width: 100, height: 100))
        customView?.delegate = self
        view.addSubview(customView!)
    }
    
    func customViewDidTap(_ view: CustomView) {
        print("Custom view tapped")
    }
    
    deinit {
        print("ViewController deallocated")
    }
}
```

**3. NotificationCenter에서의 메모리 리크:**
```swift
class NotificationObserver {
    private var notificationToken: NSObjectProtocol?
    
    init() {
        setupNotifications()
    }
    
    private func setupNotifications() {
        // 🚨 클로저 기반 옵저버는 강한 참조를 만들 수 있음
        notificationToken = NotificationCenter.default.addObserver(
            forName: .UIApplicationDidEnterBackground,
            object: nil,
            queue: .main
        ) { [weak self] notification in
            self?.handleBackgroundTransition()
        }
        
        // ✅ 또는 셀렉터 기반 옵저버 사용
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(handleBackgroundTransition),
            name: .UIApplicationDidEnterBackground,
            object: nil
        )
    }
    
    @objc private func handleBackgroundTransition() {
        print("App entered background")
    }
    
    deinit {
        // ✅ 옵저버 정리 필수
        if let token = notificationToken {
            NotificationCenter.default.removeObserver(token)
        }
        
        NotificationCenter.default.removeObserver(self)
        print("NotificationObserver deallocated")
    }
}
```

**4. Timer와 메모리 리크:**
```swift
class TimerManager {
    private var timer: Timer?
    private weak var target: AnyObject?
    
    // 🚨 Timer는 강한 참조를 만듦
    func startBadTimer() {
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] _ in
            self?.timerTick()
        }
    }
    
    // ✅ 더 안전한 방법: weak target 사용
    func startSafeTimer(target: AnyObject) {
        self.target = target
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] _ in
            guard let _ = self?.target else {
                self?.stopTimer()
                return
            }
            self?.timerTick()
        }
    }
    
    // ✅ 가장 안전한 방법: CADisplayLink 사용 (UI 업데이트용)
    private var displayLink: CADisplayLink?
    
    func startDisplayLink() {
        displayLink = CADisplayLink(target: self, selector: #selector(displayLinkTick))
        displayLink?.add(to: .main, forMode: .default)
    }
    
    @objc private func displayLinkTick() {
        // 60fps 업데이트
    }
    
    private func timerTick() {
        print("Timer tick")
    }
    
    func stopTimer() {
        timer?.invalidate()
        timer = nil
        
        displayLink?.invalidate()
        displayLink = nil
    }
    
    deinit {
        stopTimer()
        print("TimerManager deallocated")
    }
}
```

**5. 비동기 작업과 메모리 관리:**
```swift
class AsyncOperationManager {
    private var operations: [Operation] = []
    private let operationQueue = OperationQueue()
    
    func performAsyncOperation() {
        let operation = BlockOperation { [weak self] in
            // 🚨 긴 작업 중에 self가 해제될 수 있음
            guard let self = self else { return }
            
            for i in 0..<1000000 {
                // 주기적으로 취소 확인
                if operation.isCancelled {
                    return
                }
                
                self.processItem(i)
                
                // 100개마다 잠시 대기
                if i % 100 == 0 {
                    Thread.sleep(forTimeInterval: 0.001)
                }
            }
        }
        
        operations.append(operation)
        operationQueue.addOperation(operation)
    }
    
    // ✅ 현대적 async/await 방식
    func performModernAsyncOperation() async {
        await withTaskGroup(of: Void.self) { group in
            for i in 0..<10 {
                group.addTask { [weak self] in
                    await self?.processItemAsync(i)
                }
            }
        }
    }
    
    private func processItem(_ index: Int) {
        // 무거운 작업
    }
    
    private func processItemAsync(_ index: Int) async {
        // 비동기 무거운 작업
        await Task.sleep(nanoseconds: 1_000_000) // 1ms
    }
    
    func cancelAllOperations() {
        operationQueue.cancelAllOperations()
        operations.removeAll()
    }
    
    deinit {
        cancelAllOperations()
        print("AsyncOperationManager deallocated")
    }
}
```

**6. Core Data와 메모리 관리:**
```swift
import CoreData

class CoreDataMemoryManager {
    lazy var persistentContainer: NSPersistentContainer = {
        let container = NSPersistentContainer(name: "DataModel")
        container.loadPersistentStores { _, error in
            if let error = error {
                fatalError("Core Data error: \(error)")
            }
        }
        return container
    }()
    
    // 🚨 메모리 문제를 일으킬 수 있는 패턴
    func badFetchExample() -> [NSManagedObject] {
        let context = persistentContainer.viewContext
        let request: NSFetchRequest<NSManagedObject> = NSFetchRequest(entityName: "User")
        
        // 모든 객체를 메모리에 로드 - 위험!
        do {
            return try context.fetch(request)
        } catch {
            return []
        }
    }
    
    // ✅ 메모리 효율적인 패턴
    func efficientFetchExample() {
        let context = persistentContainer.newBackgroundContext()
        let request: NSFetchRequest<NSManagedObject> = NSFetchRequest(entityName: "User")
        
        // 배치 크기 제한
        request.fetchBatchSize = 50
        
        // 필요한 속성만 가져오기
        request.propertiesToFetch = ["name", "email"]
        request.returnsObjectsAsFaults = false
        
        do {
            let results = try context.fetch(request)
            
            // 처리 후 즉시 정리
            for object in results {
                context.refresh(object, mergeChanges: false)
            }
            
        } catch {
            print("Fetch error: \(error)")
        }
    }
    
    // ✅ 대용량 데이터 처리를 위한 NSFetchedResultsController
    func setupFetchedResultsController() -> NSFetchedResultsController<NSManagedObject> {
        let context = persistentContainer.viewContext
        let request: NSFetchRequest<NSManagedObject> = NSFetchRequest(entityName: "User")
        
        request.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
        request.fetchBatchSize = 20
        
        let controller = NSFetchedResultsController(
            fetchRequest: request,
            managedObjectContext: context,
            sectionNameKeyPath: nil,
            cacheName: "UserCache"
        )
        
        return controller
    }
}
```

**7. 메모리 디버깅 도구들:**
```swift
class MemoryDebugger {
    static func enableMemoryDebugging() {
        #if DEBUG
        // 메모리 사용량 모니터링
        startMemoryMonitoring()
        
        // 좀비 객체 감지
        enableZombieDetection()
        
        // 메모리 경고 시뮬레이션
        simulateMemoryWarning()
        #endif
    }
    
    private static func startMemoryMonitoring() {
        Timer.scheduledTimer(withTimeInterval: 5.0, repeats: true) { _ in
            let usage = getMemoryUsage()
            print("Memory usage: \(usage.used / 1024 / 1024) MB / \(usage.total / 1024 / 1024) MB")
            
            if usage.used > usage.total * 0.8 {
                print("⚠️ High memory usage detected!")
            }
        }
    }
    
    private static func getMemoryUsage() -> (used: UInt64, total: UInt64) {
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
        
        if kerr == KERN_SUCCESS {
            return (used: info.resident_size, total: ProcessInfo.processInfo.physicalMemory)
        } else {
            return (used: 0, total: 0)
        }
    }
    
    private static func enableZombieDetection() {
        // NSZombie 환경 변수를 통한 좀비 객체 감지
        // Product -> Scheme -> Edit Scheme -> Run -> Arguments -> Environment Variables
        // NSZombieEnabled = YES
        print("Enable NSZombieEnabled in scheme for zombie detection")
    }
    
    private static func simulateMemoryWarning() {
        // 메모리 경고 시뮬레이션
        DispatchQueue.main.asyncAfter(deadline: .now() + 10) {
            let app = UIApplication.shared
            if app.responds(to: #selector(UIApplication._performMemoryWarning)) {
                app.perform(#selector(UIApplication._performMemoryWarning))
            }
        }
    }
    
    // 메모리 리크 감지를 위한 객체 추적
    private static var trackedObjects: [String: WeakObjectContainer] = [:]
    
    static func trackObject(_ object: AnyObject, name: String) {
        trackedObjects[name] = WeakObjectContainer(object: object)
    }
    
    static func checkForLeaks() {
        for (name, container) in trackedObjects {
            if container.object != nil {
                print("Potential leak detected: \(name)")
            }
        }
        
        // 해제된 객체 정리
        trackedObjects = trackedObjects.filter { $0.value.object != nil }
    }
}

class WeakObjectContainer {
    weak var object: AnyObject?
    
    init(object: AnyObject) {
        self.object = object
    }
}

// 사용법
class LeakProneClass {
    init() {
        MemoryDebugger.trackObject(self, name: "LeakProneClass-\(ObjectIdentifier(self))")
    }
    
    deinit {
        print("LeakProneClass deallocated")
    }
}
```

메모리 관리는 iOS 개발에서 절대 간과해서는 안 되는 영역이다. ARC가 대부분을 처리해주지만, 복잡한 객체 관계와 비동기 작업에서는 여전히 세심한 주의가 필요하다. 특히 프로덕션 환경에서 메모리 리크는 사용자 경험을 치명적으로 해칠 수 있으므로, 개발 단계에서부터 체계적으로 관리해야 한다.

## Connections
→ [[307-offline-experiences-network-failures]]
→ [[308-device-specific-optimizations]]
← [[305-internationalization-advanced]]
← [[190-arc-automatic-memory-management]]

---
Level: L3
Date: 2025-08-16
Tags: #memory-management #arc #retain-cycles #memory-leaks #debugging #performance