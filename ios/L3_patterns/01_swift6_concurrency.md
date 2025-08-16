# Swift 6 동시성 완벽 가이드

## 핵심 개념: Data Race 완전 차단

Swift 6의 목표는 단순합니다: **컴파일 타임에 모든 data race를 차단**하는 것입니다.

## 1. Sendable: 동시성의 기본 단위

### Sendable이란?
스레드 간에 안전하게 전달할 수 있는 타입을 의미합니다.

```swift
// 자동으로 Sendable
struct ImmutableData: Sendable {
    let id: Int
    let name: String
}

// 명시적 Sendable 채택
final class ThreadSafeCounter: @unchecked Sendable {
    private let lock = NSLock()
    private var _count = 0
    
    var count: Int {
        lock.withLock { _count }
    }
    
    func increment() {
        lock.withLock { _count += 1 }
    }
}

// Sendable 클로저
let operation: @Sendable () async -> Void = {
    await performWork()
}
```

### 실전 예제: API Client

```swift
// MARK: - Sendable한 Request/Response 모델
struct APIRequest: Sendable {
    let endpoint: String
    let method: HTTPMethod
    let body: Data?
}

struct APIResponse<T: Sendable>: Sendable {
    let data: T
    let statusCode: Int
}

// MARK: - Actor 기반 네트워크 매니저
actor NetworkManager {
    private let session: URLSession
    private var activeTasks: [UUID: Task<Data, Error>] = [:]
    
    init(configuration: URLSessionConfiguration = .default) {
        self.session = URLSession(configuration: configuration)
    }
    
    // nonisolated: Actor 외부에서 동기적 접근 가능
    nonisolated func endpoint(for path: String) -> URL {
        URL(string: "https://api.example.com/\(path)")!
    }
    
    func request<T: Decodable & Sendable>(
        _ request: APIRequest,
        expecting: T.Type
    ) async throws -> APIResponse<T> {
        let url = endpoint(for: request.endpoint)
        var urlRequest = URLRequest(url: url)
        urlRequest.httpMethod = request.method.rawValue
        urlRequest.httpBody = request.body
        
        // 중복 요청 방지
        let taskID = UUID()
        let task = Task {
            try await session.data(for: urlRequest).0
        }
        
        activeTasks[taskID] = task
        defer { activeTasks[taskID] = nil }
        
        let data = try await task.value
        let decoded = try JSONDecoder().decode(T.self, from: data)
        
        return APIResponse(data: decoded, statusCode: 200)
    }
}
```

## 2. Actor Isolation 실전

### MainActor와 UI 업데이트

```swift
@MainActor
class DeliveryViewModel: ObservableObject {
    @Published private(set) var orders: [Order] = []
    @Published private(set) var isLoading = false
    @Published private(set) var error: Error?
    
    private let networkManager = NetworkManager()
    private var loadTask: Task<Void, Never>?
    
    // MainActor에서 실행
    func loadOrders() {
        // 이전 작업 취소
        loadTask?.cancel()
        
        loadTask = Task {
            isLoading = true
            error = nil
            
            do {
                // Actor 경계를 넘어 데이터 가져오기
                let response = try await networkManager.request(
                    APIRequest(
                        endpoint: "orders",
                        method: .get,
                        body: nil
                    ),
                    expecting: [Order].self
                )
                
                // Task 취소 체크
                guard !Task.isCancelled else { return }
                
                // MainActor에서 UI 업데이트
                self.orders = response.data
            } catch {
                guard !Task.isCancelled else { return }
                self.error = error
            }
            
            isLoading = false
        }
    }
    
    // nonisolated: 백그라운드에서 실행 가능
    nonisolated func formatOrderDate(_ date: Date) -> String {
        DateFormatter.localizedString(
            from: date,
            dateStyle: .medium,
            timeStyle: .short
        )
    }
}
```

### Custom Actor로 리소스 관리

```swift
// 위치 추적을 관리하는 Actor
actor LocationTracker {
    private var locations: [CLLocation] = []
    private var observers: [UUID: @Sendable (CLLocation) -> Void] = [:]
    private let maxLocations = 100
    
    func add(_ location: CLLocation) {
        locations.append(location)
        
        // 메모리 관리: 오래된 위치 제거
        if locations.count > maxLocations {
            locations.removeFirst(locations.count - maxLocations)
        }
        
        // 모든 옵저버에 알림
        for observer in observers.values {
            Task { observer(location) }
        }
    }
    
    func subscribe(
        id: UUID = UUID(),
        handler: @escaping @Sendable (CLLocation) -> Void
    ) {
        observers[id] = handler
    }
    
    func unsubscribe(id: UUID) {
        observers[id] = nil
    }
    
    // 최근 경로 계산
    func recentPath(last: Int = 10) -> [CLLocation] {
        Array(locations.suffix(last))
    }
    
    // 총 이동 거리 계산
    func totalDistance() -> CLLocationDistance {
        guard locations.count > 1 else { return 0 }
        
        return zip(locations, locations.dropFirst())
            .reduce(0) { total, pair in
                total + pair.0.distance(from: pair.1)
            }
    }
}
```

## 3. Strict Concurrency 마이그레이션

### @preconcurrency로 레거시 코드 대응

```swift
// 레거시 SDK 래핑
@preconcurrency import LegacyPaymentSDK

actor PaymentProcessor {
    // @preconcurrency: 컴파일러 경고 억제
    @preconcurrency
    private let legacyProcessor = LegacyPaymentProcessor()
    
    func processPayment(
        amount: Decimal,
        card: CreditCard
    ) async throws -> PaymentResult {
        // Actor 내부에서 안전하게 사용
        return try await withCheckedThrowingContinuation { continuation in
            legacyProcessor.process(
                amount: amount,
                card: card
            ) { result, error in
                if let error {
                    continuation.resume(throwing: error)
                } else if let result {
                    continuation.resume(returning: result)
                } else {
                    continuation.resume(
                        throwing: PaymentError.unknown
                    )
                }
            }
        }
    }
}
```

### Global Actor 활용

```swift
// 데이터베이스 접근을 위한 전역 Actor
@globalActor
actor DatabaseActor {
    static let shared = DatabaseActor()
}

// 데이터베이스 매니저
@DatabaseActor
final class DatabaseManager {
    private let container: NSPersistentContainer
    
    init() {
        container = NSPersistentContainer(name: "DataModel")
        container.loadPersistentStores { _, error in
            if let error {
                fatalError("Database failed: \(error)")
            }
        }
    }
    
    func save(_ order: Order) async throws {
        let context = container.viewContext
        
        let entity = OrderEntity(context: context)
        entity.id = order.id
        entity.status = order.status.rawValue
        entity.createdAt = order.createdAt
        
        try context.save()
    }
    
    func fetchOrders() async throws -> [Order] {
        let context = container.viewContext
        let request = OrderEntity.fetchRequest()
        request.sortDescriptors = [
            NSSortDescriptor(key: "createdAt", ascending: false)
        ]
        
        let entities = try context.fetch(request)
        return entities.compactMap { entity in
            guard let id = entity.id,
                  let statusRaw = entity.status,
                  let status = OrderStatus(rawValue: statusRaw),
                  let createdAt = entity.createdAt else {
                return nil
            }
            
            return Order(
                id: id,
                status: status,
                createdAt: createdAt
            )
        }
    }
}
```

## 4. TaskGroup과 병렬 처리

### 효율적인 병렬 다운로드

```swift
actor ImageCache {
    private var cache: [URL: UIImage] = [:]
    private var inProgress: [URL: Task<UIImage, Error>] = [:]
    
    func image(for url: URL) async throws -> UIImage {
        // 캐시 확인
        if let cached = cache[url] {
            return cached
        }
        
        // 진행 중인 다운로드 확인
        if let task = inProgress[url] {
            return try await task.value
        }
        
        // 새 다운로드 시작
        let task = Task {
            try await downloadImage(from: url)
        }
        
        inProgress[url] = task
        
        do {
            let image = try await task.value
            cache[url] = image
            inProgress[url] = nil
            return image
        } catch {
            inProgress[url] = nil
            throw error
        }
    }
    
    // 여러 이미지 병렬 다운로드
    func preloadImages(urls: [URL]) async {
        await withTaskGroup(of: Void.self) { group in
            for url in urls {
                group.addTask {
                    _ = try? await self.image(for: url)
                }
            }
        }
    }
    
    private func downloadImage(from url: URL) async throws -> UIImage {
        let (data, response) = try await URLSession.shared.data(from: url)
        
        guard let httpResponse = response as? HTTPURLResponse,
              httpResponse.statusCode == 200,
              let image = UIImage(data: data) else {
            throw ImageError.invalidData
        }
        
        return image
    }
}
```

## 5. 성능 최적화 패턴

### AsyncStream으로 실시간 데이터 처리

```swift
actor OrderTracker {
    private var continuation: AsyncStream<OrderUpdate>.Continuation?
    
    func updates() -> AsyncStream<OrderUpdate> {
        AsyncStream { continuation in
            self.continuation = continuation
            
            continuation.onTermination = { _ in
                Task { await self.stopTracking() }
            }
        }
    }
    
    func trackOrder(_ orderID: String) async {
        // WebSocket이나 Server-Sent Events 연결
        let eventSource = EventSource(
            url: URL(string: "https://api.example.com/orders/\(orderID)/track")!
        )
        
        for await event in eventSource.events() {
            switch event {
            case .statusChanged(let status):
                continuation?.yield(
                    OrderUpdate(
                        orderID: orderID,
                        status: status,
                        timestamp: Date()
                    )
                )
                
            case .locationUpdated(let location):
                continuation?.yield(
                    OrderUpdate(
                        orderID: orderID,
                        location: location,
                        timestamp: Date()
                    )
                )
                
            case .completed:
                continuation?.finish()
                
            case .error(let error):
                continuation?.finish()
                print("Tracking error: \(error)")
            }
        }
    }
    
    private func stopTracking() {
        continuation?.finish()
        continuation = nil
    }
}

// 사용 예제
@MainActor
class OrderTrackingView: View {
    @State private var updates: [OrderUpdate] = []
    private let tracker = OrderTracker()
    
    var body: some View {
        List(updates) { update in
            OrderUpdateRow(update: update)
        }
        .task {
            for await update in tracker.updates() {
                updates.append(update)
                
                // UI 업데이트 제한 (최대 30개)
                if updates.count > 30 {
                    updates.removeFirst()
                }
            }
        }
    }
}
```

## 6. 에러 처리와 취소

### Structured Concurrency 활용

```swift
actor DataSyncManager {
    private var syncTask: Task<Void, Error>?
    
    func startSync() {
        // 이전 동기화 취소
        syncTask?.cancel()
        
        syncTask = Task {
            try await withThrowingTaskGroup(of: Void.self) { group in
                // 1. 주문 동기화
                group.addTask {
                    try await self.syncOrders()
                }
                
                // 2. 사용자 데이터 동기화
                group.addTask {
                    try await self.syncUserData()
                }
                
                // 3. 설정 동기화
                group.addTask {
                    try await self.syncSettings()
                }
                
                // 모든 작업 대기
                try await group.waitForAll()
            }
        }
    }
    
    func stopSync() {
        syncTask?.cancel()
        syncTask = nil
    }
    
    private func syncOrders() async throws {
        for page in 1...10 {
            // 취소 체크
            try Task.checkCancellation()
            
            let orders = try await fetchOrdersPage(page)
            await processOrders(orders)
            
            // CPU 양보
            await Task.yield()
        }
    }
}
```

## 7. 실전 체크리스트

### 마이그레이션 단계

1. **프로젝트 설정**
```swift
// Package.swift
.target(
    name: "MyApp",
    swiftSettings: [
        .enableExperimentalFeature("StrictConcurrency")
    ]
)
```

2. **단계별 적용**
   - Complete checking 활성화
   - 경고를 하나씩 수정
   - @preconcurrency로 임시 대응
   - 최종적으로 모든 경고 제거

3. **성능 모니터링**
   - Actor 경계 crossing 최소화
   - 불필요한 await 제거
   - TaskGroup 활용도 확인

### 주의사항

- **Actor 재진입**: await 이후 상태가 변경될 수 있음
- **Sendable 제약**: 클로저와 제네릭 타입 주의
- **MainActor 부담**: UI 업데이트만 MainActor에서
- **취소 처리**: Task.checkCancellation() 적극 활용

## 결론

Swift 6 동시성은 **안전성과 성능을 동시에** 제공합니다. 컴파일 타임에 data race를 차단하면서도, Actor와 async/await로 깔끔한 코드를 작성할 수 있습니다. 

핵심은 **Sendable을 이해하고, Actor isolation을 적절히 활용**하는 것입니다.