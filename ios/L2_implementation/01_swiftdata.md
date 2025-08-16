# SwiftData 실전 가이드

## 핵심: Core Data의 복잡함을 버리고 Swift의 간결함을 택하다

SwiftData는 **Swift-first 영속성 프레임워크**입니다. Core Data의 강력함은 유지하면서 Swift의 현대적 문법으로 단순화했습니다.

## 1. 모델 정의

### @Model 매크로 활용

```swift
import SwiftData

// MARK: - 기본 모델
@Model
final class Order {
    // 자동으로 @Attribute 적용
    var id: UUID
    var orderNumber: String
    var createdAt: Date
    var status: OrderStatus
    
    // 관계 설정
    var customer: Customer?
    var items: [OrderItem]
    var delivery: Delivery?
    
    // Computed Property
    var totalAmount: Decimal {
        items.reduce(0) { $0 + $1.subtotal }
    }
    
    // 초기화
    init(
        orderNumber: String,
        customer: Customer,
        items: [OrderItem] = []
    ) {
        self.id = UUID()
        self.orderNumber = orderNumber
        self.createdAt = Date()
        self.status = .pending
        self.customer = customer
        self.items = items
    }
}

// MARK: - Enum 지원
enum OrderStatus: String, Codable {
    case pending = "pending"
    case confirmed = "confirmed"
    case preparing = "preparing"
    case ready = "ready"
    case onTheWay = "on_the_way"
    case delivered = "delivered"
    case cancelled = "cancelled"
}

// MARK: - 관계 모델
@Model
final class Customer {
    var id: UUID
    var name: String
    var email: String
    var phone: String
    
    // Inverse relationship
    @Relationship(deleteRule: .cascade, inverse: \Order.customer)
    var orders: [Order] = []
    
    // 메타데이터
    var joinedAt: Date
    var lastOrderAt: Date?
    var totalSpent: Decimal
    
    init(name: String, email: String, phone: String) {
        self.id = UUID()
        self.name = name
        self.email = email
        self.phone = phone
        self.joinedAt = Date()
        self.totalSpent = 0
    }
}

@Model
final class OrderItem {
    var id: UUID
    var menuItem: MenuItem
    var quantity: Int
    var price: Decimal
    var specialInstructions: String?
    
    // 계산 속성
    var subtotal: Decimal {
        price * Decimal(quantity)
    }
    
    // Inverse relationship
    var order: Order?
    
    init(menuItem: MenuItem, quantity: Int, price: Decimal) {
        self.id = UUID()
        self.menuItem = menuItem
        self.quantity = quantity
        self.price = price
    }
}

// MARK: - 복잡한 타입 저장
@Model
final class Delivery {
    var id: UUID
    var address: Address // Codable struct
    var scheduledAt: Date
    var deliveredAt: Date?
    var driverName: String?
    var trackingURL: URL?
    
    // 위치 추적
    @Attribute(.transformable(by: LocationTransformer.self))
    var route: [CLLocation]?
    
    init(address: Address, scheduledAt: Date) {
        self.id = UUID()
        self.address = address
        self.scheduledAt = scheduledAt
    }
}

// Codable struct for embedded data
struct Address: Codable {
    let street: String
    let city: String
    let postalCode: String
    let coordinate: Coordinate
    
    struct Coordinate: Codable {
        let latitude: Double
        let longitude: Double
    }
}
```

## 2. ModelContainer & ModelContext

### 컨테이너 설정

```swift
import SwiftUI
import SwiftData

@main
struct DeliveryApp: App {
    let container: ModelContainer
    
    init() {
        do {
            // 스키마 정의
            let schema = Schema([
                Order.self,
                Customer.self,
                OrderItem.self,
                MenuItem.self,
                Delivery.self
            ])
            
            // 설정
            let configuration = ModelConfiguration(
                schema: schema,
                isStoredInMemoryOnly: false,
                allowsSave: true,
                groupContainer: .automatic,
                cloudKitDatabase: .automatic // CloudKit 동기화
            )
            
            container = try ModelContainer(
                for: schema,
                configurations: [configuration]
            )
            
            // 마이그레이션 처리
            container.mainContext.autosaveEnabled = true
            
        } catch {
            fatalError("Failed to create ModelContainer: \(error)")
        }
    }
    
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(container)
    }
}

// MARK: - 고급 설정
class DataController: ObservableObject {
    let container: ModelContainer
    
    @MainActor
    var mainContext: ModelContext {
        container.mainContext
    }
    
    init(inMemory: Bool = false) {
        let schema = Schema([
            Order.self,
            Customer.self,
            OrderItem.self
        ])
        
        let configuration = ModelConfiguration(
            schema: schema,
            isStoredInMemoryOnly: inMemory,
            allowsSave: !inMemory,
            groupContainer: .identifier("group.com.example.delivery"),
            cloudKitDatabase: inMemory ? .none : .automatic
        )
        
        do {
            container = try ModelContainer(
                for: schema,
                configurations: [configuration]
            )
            
            // 백그라운드 컨텍스트 설정
            setupBackgroundContext()
            
            // 개발 환경에서 샘플 데이터
            #if DEBUG
            if inMemory {
                Task { @MainActor in
                    createSampleData()
                }
            }
            #endif
            
        } catch {
            fatalError("Failed to create container: \(error)")
        }
    }
    
    private func setupBackgroundContext() {
        let backgroundContext = ModelContext(container)
        backgroundContext.autosaveEnabled = false
        
        // 백그라운드 작업용
        Task.detached {
            // 대량 데이터 처리
        }
    }
    
    @MainActor
    private func createSampleData() {
        let customer = Customer(
            name: "테스트 사용자",
            email: "test@example.com",
            phone: "010-1234-5678"
        )
        
        mainContext.insert(customer)
        
        // 샘플 주문 생성
        for i in 1...10 {
            let order = Order(
                orderNumber: "ORD-\(String(format: "%04d", i))",
                customer: customer
            )
            mainContext.insert(order)
        }
    }
}
```

## 3. Query와 데이터 페칭

### @Query 매크로 활용

```swift
import SwiftUI
import SwiftData

struct OrderListView: View {
    // 기본 쿼리
    @Query private var allOrders: [Order]
    
    // 정렬된 쿼리
    @Query(sort: \Order.createdAt, order: .reverse)
    private var recentOrders: [Order]
    
    // 필터링된 쿼리
    @Query(filter: #Predicate<Order> { order in
        order.status == .pending || order.status == .confirmed
    })
    private var activeOrders: [Order]
    
    // 복잡한 쿼리
    @Query(
        filter: #Predicate<Order> { order in
            order.createdAt > Date.now.addingTimeInterval(-86400) &&
            order.totalAmount > 50000
        },
        sort: [
            SortDescriptor(\Order.totalAmount, order: .reverse),
            SortDescriptor(\Order.createdAt)
        ]
    )
    private var todayLargeOrders: [Order]
    
    var body: some View {
        List {
            Section("활성 주문") {
                ForEach(activeOrders) { order in
                    OrderRow(order: order)
                }
            }
            
            Section("오늘 대형 주문") {
                ForEach(todayLargeOrders) { order in
                    LargeOrderRow(order: order)
                }
            }
        }
    }
}

// MARK: - 동적 쿼리
struct FilteredOrderView: View {
    let statusFilter: OrderStatus?
    let customerID: UUID?
    
    // 동적 Predicate
    private var predicate: Predicate<Order> {
        if let status = statusFilter, let customerID = customerID {
            return #Predicate { order in
                order.status == status &&
                order.customer?.id == customerID
            }
        } else if let status = statusFilter {
            return #Predicate { order in
                order.status == status
            }
        } else if let customerID = customerID {
            return #Predicate { order in
                order.customer?.id == customerID
            }
        } else {
            return #Predicate { _ in true }
        }
    }
    
    var body: some View {
        OrderListWithPredicate(predicate: predicate)
    }
}

struct OrderListWithPredicate: View {
    @Query private var orders: [Order]
    
    init(predicate: Predicate<Order>) {
        _orders = Query(
            filter: predicate,
            sort: \Order.createdAt,
            order: .reverse
        )
    }
    
    var body: some View {
        List(orders) { order in
            OrderRow(order: order)
        }
    }
}
```

### 수동 페칭

```swift
extension ModelContext {
    // 페이징 지원
    func fetchOrders(
        page: Int,
        pageSize: Int = 20,
        status: OrderStatus? = nil
    ) throws -> [Order] {
        var descriptor = FetchDescriptor<Order>(
            sortBy: [SortDescriptor(\Order.createdAt, order: .reverse)]
        )
        
        // 필터
        if let status = status {
            descriptor.predicate = #Predicate { order in
                order.status == status
            }
        }
        
        // 페이징
        descriptor.fetchLimit = pageSize
        descriptor.fetchOffset = page * pageSize
        
        return try fetch(descriptor)
    }
    
    // 집계 쿼리
    func calculateDailyRevenue(for date: Date) throws -> Decimal {
        let calendar = Calendar.current
        let startOfDay = calendar.startOfDay(for: date)
        let endOfDay = calendar.date(byAdding: .day, value: 1, to: startOfDay)!
        
        let descriptor = FetchDescriptor<Order>(
            predicate: #Predicate { order in
                order.createdAt >= startOfDay &&
                order.createdAt < endOfDay &&
                order.status == .delivered
            }
        )
        
        let orders = try fetch(descriptor)
        return orders.reduce(0) { $0 + $1.totalAmount }
    }
    
    // 관계 프리페칭
    func fetchOrdersWithRelationships() throws -> [Order] {
        var descriptor = FetchDescriptor<Order>()
        descriptor.relationshipKeyPathsForPrefetching = [
            \Order.customer,
            \Order.items,
            \Order.delivery
        ]
        
        return try fetch(descriptor)
    }
}
```

## 4. Core Data 마이그레이션

### 기존 Core Data에서 SwiftData로

```swift
// MARK: - 마이그레이션 매니저
class MigrationManager {
    static func migrateFromCoreData() async throws {
        // 1. Core Data 스택 로드
        let coreDataStack = CoreDataStack()
        let coreDataContext = coreDataStack.viewContext
        
        // 2. SwiftData 컨테이너 생성
        let swiftDataContainer = try ModelContainer(
            for: Order.self, Customer.self
        )
        let swiftDataContext = swiftDataContainer.mainContext
        
        // 3. 데이터 마이그레이션
        let fetchRequest = NSFetchRequest<NSManagedObject>(
            entityName: "CDOrder"
        )
        let coreDataOrders = try coreDataContext.fetch(fetchRequest)
        
        for cdOrder in coreDataOrders {
            // Core Data 객체를 SwiftData로 변환
            let order = Order(
                orderNumber: cdOrder.value(forKey: "orderNumber") as! String,
                customer: Customer(
                    name: cdOrder.value(forKey: "customerName") as! String,
                    email: cdOrder.value(forKey: "customerEmail") as! String,
                    phone: cdOrder.value(forKey: "customerPhone") as! String
                )
            )
            
            order.createdAt = cdOrder.value(forKey: "createdAt") as! Date
            order.status = OrderStatus(
                rawValue: cdOrder.value(forKey: "status") as! String
            )!
            
            swiftDataContext.insert(order)
        }
        
        // 4. 저장
        try swiftDataContext.save()
        
        // 5. Core Data 파일 백업
        backupCoreDataFiles()
    }
    
    private static func backupCoreDataFiles() {
        let documentsDirectory = FileManager.default.urls(
            for: .documentDirectory,
            in: .userDomainMask
        ).first!
        
        let coreDataURL = documentsDirectory.appendingPathComponent("DataModel.sqlite")
        let backupURL = documentsDirectory.appendingPathComponent("DataModel_backup.sqlite")
        
        try? FileManager.default.copyItem(at: coreDataURL, to: backupURL)
    }
}

// MARK: - 스키마 마이그레이션
enum OrderSchemaV1: VersionedSchema {
    static var versionIdentifier = Schema.Version(1, 0, 0)
    static var models: [any PersistentModel.Type] {
        [OrderV1.self]
    }
    
    @Model
    final class OrderV1 {
        var id: UUID
        var orderNumber: String
        var createdAt: Date
        
        init(orderNumber: String) {
            self.id = UUID()
            self.orderNumber = orderNumber
            self.createdAt = Date()
        }
    }
}

enum OrderSchemaV2: VersionedSchema {
    static var versionIdentifier = Schema.Version(2, 0, 0)
    static var models: [any PersistentModel.Type] {
        [OrderV2.self]
    }
    
    @Model
    final class OrderV2 {
        var id: UUID
        var orderNumber: String
        var createdAt: Date
        var status: String // 새 필드 추가
        var totalAmount: Decimal // 새 필드 추가
        
        init(orderNumber: String) {
            self.id = UUID()
            self.orderNumber = orderNumber
            self.createdAt = Date()
            self.status = "pending"
            self.totalAmount = 0
        }
    }
}

// 마이그레이션 플랜
enum OrderMigrationPlan: SchemaMigrationPlan {
    static var schemas: [any VersionedSchema.Type] {
        [OrderSchemaV1.self, OrderSchemaV2.self]
    }
    
    static var stages: [MigrationStage] {
        [migrateV1toV2]
    }
    
    static let migrateV1toV2 = MigrationStage.custom(
        fromVersion: OrderSchemaV1.self,
        toVersion: OrderSchemaV2.self,
        willMigrate: { context in
            // 마이그레이션 전 준비
        },
        didMigrate: { context in
            // 마이그레이션 후 처리
            let orders = try context.fetch(FetchDescriptor<OrderSchemaV2.OrderV2>())
            for order in orders {
                if order.status.isEmpty {
                    order.status = "pending"
                }
                if order.totalAmount == 0 {
                    // 계산 로직
                    order.totalAmount = calculateTotal(for: order)
                }
            }
            try context.save()
        }
    )
}
```

## 5. CloudKit 동기화

### CloudKit 통합 설정

```swift
// MARK: - CloudKit 동기화 설정
class CloudKitSyncManager {
    private let container: ModelContainer
    
    init() throws {
        let schema = Schema([
            Order.self,
            Customer.self,
            OrderItem.self
        ])
        
        // CloudKit 컨테이너 설정
        let configuration = ModelConfiguration(
            schema: schema,
            isStoredInMemoryOnly: false,
            allowsSave: true,
            groupContainer: .identifier("group.com.example.delivery"),
            cloudKitDatabase: .automatic(
                CloudKitDatabase.Configuration(
                    containerIdentifier: "iCloud.com.example.delivery",
                    databaseScope: .private
                )
            )
        )
        
        container = try ModelContainer(
            for: schema,
            configurations: [configuration]
        )
        
        setupSyncMonitoring()
    }
    
    private func setupSyncMonitoring() {
        // 동기화 상태 모니터링
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(syncDidStart),
            name: .NSPersistentStoreRemoteChange,
            object: nil
        )
    }
    
    @objc private func syncDidStart(_ notification: Notification) {
        Task { @MainActor in
            // UI 업데이트
            print("CloudKit sync started")
        }
    }
    
    // 수동 동기화 트리거
    func triggerSync() async throws {
        let context = ModelContext(container)
        
        // 변경사항 강제 저장
        if context.hasChanges {
            try context.save()
        }
        
        // CloudKit 동기화 요청
        // SwiftData는 자동으로 처리하지만 필요시 수동 트리거
    }
    
    // 충돌 해결
    func resolveConflicts() {
        container.mainContext.mergePolicy = .mergeByPropertyObjectTrump
        // 또는 커스텀 정책
    }
}

// MARK: - 공유 데이터
@Model
final class SharedOrder {
    var id: UUID
    var orderNumber: String
    
    // CloudKit 공유 설정
    @Attribute(.externalStorage)
    var shareRecord: CKShare?
    
    func share(with email: String) async throws {
        // CKShare 생성 및 설정
        let share = CKShare(rootRecord: self.cloudKitRecord)
        share.publicPermission = .readOnly
        
        // 참여자 추가
        let participant = CKShare.Participant()
        participant.userIdentity = try await lookupUser(email: email)
        participant.permission = .readWrite
        share.addParticipant(participant)
        
        self.shareRecord = share
    }
}
```

## 6. 성능 최적화

### 쿼리 최적화

```swift
// MARK: - 배치 처리
class BatchProcessor {
    private let context: ModelContext
    
    init(context: ModelContext) {
        self.context = context
    }
    
    // 대량 삽입
    func batchInsertOrders(_ ordersData: [[String: Any]]) async throws {
        // 백그라운드 컨텍스트 사용
        let backgroundContext = ModelContext(context.container)
        backgroundContext.autosaveEnabled = false
        
        // 배치 단위로 처리
        let batchSize = 1000
        for batch in ordersData.chunked(into: batchSize) {
            autoreleasepool {
                for orderData in batch {
                    let order = Order(
                        orderNumber: orderData["orderNumber"] as! String,
                        customer: findOrCreateCustomer(
                            data: orderData["customer"] as! [String: Any],
                            in: backgroundContext
                        )
                    )
                    backgroundContext.insert(order)
                }
            }
            
            // 배치마다 저장
            try backgroundContext.save()
            
            // 메모리 정리
            backgroundContext.reset()
        }
    }
    
    // 대량 업데이트
    func batchUpdateOrderStatus(
        from oldStatus: OrderStatus,
        to newStatus: OrderStatus
    ) async throws {
        let descriptor = FetchDescriptor<Order>(
            predicate: #Predicate { order in
                order.status == oldStatus
            }
        )
        
        let orders = try context.fetch(descriptor)
        
        // 배치 업데이트
        for order in orders {
            order.status = newStatus
        }
        
        // 한번에 저장
        try context.save()
    }
    
    // 대량 삭제
    func batchDeleteOldOrders(olderThan date: Date) throws {
        try context.delete(
            model: Order.self,
            where: #Predicate { order in
                order.createdAt < date
            }
        )
    }
}

// MARK: - 메모리 관리
extension ModelContext {
    func performMemoryEfficientFetch<T: PersistentModel>(
        _ type: T.Type,
        batchSize: Int = 100,
        process: (T) throws -> Void
    ) throws {
        var descriptor = FetchDescriptor<T>()
        descriptor.fetchLimit = batchSize
        
        var offset = 0
        var hasMore = true
        
        while hasMore {
            descriptor.fetchOffset = offset
            let batch = try fetch(descriptor)
            
            if batch.isEmpty {
                hasMore = false
            } else {
                for item in batch {
                    try process(item)
                }
                offset += batchSize
                
                // 메모리 정리
                reset()
            }
        }
    }
}
```

### 인덱싱과 쿼리 계획

```swift
// MARK: - 인덱스 최적화
@Model
final class OptimizedOrder {
    @Attribute(.unique) var id: UUID
    @Attribute(.index) var orderNumber: String
    @Attribute(.index) var createdAt: Date
    @Attribute(.index) var status: OrderStatus
    
    // 복합 인덱스를 위한 계산 속성
    @Attribute(.index) 
    var statusDateKey: String {
        "\(status.rawValue)_\(createdAt.timeIntervalSince1970)"
    }
    
    // 관계 프리페칭 힌트
    @Relationship(deleteRule: .nullify, minimumModelCount: 0, maximumModelCount: 1)
    var customer: Customer?
    
    @Relationship(deleteRule: .cascade, minimumModelCount: 1)
    var items: [OrderItem]
}

// 쿼리 성능 측정
class QueryPerformanceMonitor {
    func measureQueryTime<T>(
        _ query: () throws -> T
    ) rethrows -> (result: T, time: TimeInterval) {
        let start = CFAbsoluteTimeGetCurrent()
        let result = try query()
        let time = CFAbsoluteTimeGetCurrent() - start
        
        #if DEBUG
        print("Query executed in \(String(format: "%.3f", time))s")
        #endif
        
        return (result, time)
    }
}
```

## 7. 테스팅

### SwiftData 테스트

```swift
import XCTest
import SwiftData

class OrderTests: XCTestCase {
    var container: ModelContainer!
    var context: ModelContext!
    
    override func setUp() async throws {
        // 인메모리 컨테이너
        let schema = Schema([
            Order.self,
            Customer.self,
            OrderItem.self
        ])
        
        let configuration = ModelConfiguration(
            schema: schema,
            isStoredInMemoryOnly: true
        )
        
        container = try ModelContainer(
            for: schema,
            configurations: [configuration]
        )
        
        context = ModelContext(container)
    }
    
    override func tearDown() {
        container = nil
        context = nil
    }
    
    func testOrderCreation() throws {
        // Given
        let customer = Customer(
            name: "Test User",
            email: "test@example.com",
            phone: "010-1234-5678"
        )
        context.insert(customer)
        
        // When
        let order = Order(
            orderNumber: "TEST-001",
            customer: customer
        )
        context.insert(order)
        try context.save()
        
        // Then
        let fetchedOrders = try context.fetch(
            FetchDescriptor<Order>()
        )
        
        XCTAssertEqual(fetchedOrders.count, 1)
        XCTAssertEqual(fetchedOrders.first?.orderNumber, "TEST-001")
        XCTAssertEqual(fetchedOrders.first?.customer?.name, "Test User")
    }
    
    func testConcurrentAccess() async throws {
        // 동시성 테스트
        await withTaskGroup(of: Void.self) { group in
            for i in 1...10 {
                group.addTask {
                    let context = ModelContext(self.container)
                    let customer = Customer(
                        name: "Customer \(i)",
                        email: "customer\(i)@example.com",
                        phone: "010-0000-\(String(format: "%04d", i))"
                    )
                    context.insert(customer)
                    try? context.save()
                }
            }
        }
        
        let customers = try context.fetch(
            FetchDescriptor<Customer>()
        )
        XCTAssertEqual(customers.count, 10)
    }
}
```

## 8. 실전 팁

### 모범 사례

```swift
// MARK: - SwiftData 모범 사례
struct SwiftDataBestPractices {
    // 1. 모델은 final class로
    // 2. ID는 UUID 사용
    // 3. 관계는 옵셔널로
    // 4. Codable 타입 활용
    // 5. @Transient for 계산 속성
    
    // 컨텍스트 관리
    let contextManagement = """
    - MainActor: UI 업데이트용
    - Background: 대량 작업용
    - autosaveEnabled: 적절히 설정
    - 수동 save() 호출 시점 고려
    """
    
    // 성능 최적화
    let performanceTips = """
    - 필요한 필드만 페치
    - 관계 프리페칭 활용
    - 배치 처리 활용
    - 인덱스 적절히 설정
    """
    
    // 디버깅
    let debugging = """
    - SQL 로깅 활성화:
      -com.apple.CoreData.SQLDebug 1
    - 동시성 디버깅:
      -com.apple.CoreData.ConcurrencyDebug 1
    """
}
```

## 결론

SwiftData는 **Core Data의 파워를 Swift의 우아함으로 포장**했습니다. @Model과 @Query만으로 대부분의 영속성 요구사항을 해결할 수 있습니다.

핵심은:
1. **모델을 Swift답게 작성**
2. **Query는 선언적으로**
3. **CloudKit으로 자동 동기화**
4. **성능은 측정하며 최적화**

"Make it work, make it right, make it fast" - SwiftData는 이 순서를 자연스럽게 따르게 합니다.