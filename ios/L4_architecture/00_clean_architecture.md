# L4: 시스템 아키텍처 - Clean Architecture in SwiftUI

## 아키텍처의 목적

```
변경의 비용을 최소화하면서
이해의 용이성을 최대화한다
```

## Clean Architecture의 원칙

```
┌─────────────────────────────────────┐
│          Presentation               │ ← SwiftUI Views
│         (UI Framework)              │
├─────────────────────────────────────┤
│            Domain                   │ ← Business Logic
│        (Business Rules)             │   Use Cases
├─────────────────────────────────────┤
│             Data                    │ ← Repository
│        (External World)             │   API, DB
└─────────────────────────────────────┘

의존성 방향: 외부 → 내부 (단방향)
```

## 실제 구현: 음식 배달 앱

### 1. Domain Layer (핵심 비즈니스)

```swift
// Domain/Entities/Restaurant.swift
struct Restaurant: Identifiable {
    let id: UUID
    let name: String
    let cuisine: CuisineType
    let rating: Double
    let deliveryTime: TimeInterval
    let minimumOrder: Decimal
    let deliveryFee: Decimal
    
    var isOpen: Bool {
        // 비즈니스 로직
        let now = Date()
        return openingHours.contains(now)
    }
    
    var estimatedDeliveryTime: String {
        let minutes = Int(deliveryTime / 60)
        return "\(minutes)-\(minutes + 10)분"
    }
}

// Domain/Entities/Order.swift
struct Order: Identifiable {
    let id: UUID
    let restaurant: Restaurant
    let items: [OrderItem]
    let createdAt: Date
    var status: OrderStatus
    
    var subtotal: Decimal {
        items.reduce(0) { $0 + $1.totalPrice }
    }
    
    var total: Decimal {
        subtotal + restaurant.deliveryFee
    }
    
    func canBeCancelled() -> Bool {
        status == .pending || status == .confirmed
    }
}

// Domain/UseCases/PlaceOrderUseCase.swift
protocol PlaceOrderUseCaseProtocol {
    func execute(restaurant: Restaurant, items: [OrderItem]) async throws -> Order
}

class PlaceOrderUseCase: PlaceOrderUseCaseProtocol {
    private let orderRepository: OrderRepositoryProtocol
    private let paymentService: PaymentServiceProtocol
    private let notificationService: NotificationServiceProtocol
    
    init(
        orderRepository: OrderRepositoryProtocol,
        paymentService: PaymentServiceProtocol,
        notificationService: NotificationServiceProtocol
    ) {
        self.orderRepository = orderRepository
        self.paymentService = paymentService
        self.notificationService = notificationService
    }
    
    func execute(restaurant: Restaurant, items: [OrderItem]) async throws -> Order {
        // 1. 비즈니스 규칙 검증
        guard restaurant.isOpen else {
            throw OrderError.restaurantClosed
        }
        
        let subtotal = items.reduce(0) { $0 + $1.totalPrice }
        guard subtotal >= restaurant.minimumOrder else {
            throw OrderError.belowMinimumOrder
        }
        
        // 2. 결제 처리
        let paymentResult = try await paymentService.process(amount: subtotal + restaurant.deliveryFee)
        
        // 3. 주문 생성
        let order = Order(
            id: UUID(),
            restaurant: restaurant,
            items: items,
            createdAt: Date(),
            status: .pending
        )
        
        // 4. 저장
        let savedOrder = try await orderRepository.save(order)
        
        // 5. 알림 발송
        await notificationService.sendOrderConfirmation(savedOrder)
        
        return savedOrder
    }
}

// Domain/UseCases/TrackOrderUseCase.swift
protocol TrackOrderUseCaseProtocol {
    func execute(orderId: UUID) -> AsyncStream<OrderStatus>
}

class TrackOrderUseCase: TrackOrderUseCaseProtocol {
    private let orderRepository: OrderRepositoryProtocol
    
    func execute(orderId: UUID) -> AsyncStream<OrderStatus> {
        AsyncStream { continuation in
            Task {
                do {
                    for try await status in orderRepository.trackOrder(orderId) {
                        continuation.yield(status)
                    }
                    continuation.finish()
                } catch {
                    continuation.finish()
                }
            }
        }
    }
}
```

### 2. Data Layer (외부 세계와의 통신)

```swift
// Data/Repositories/OrderRepository.swift
protocol OrderRepositoryProtocol {
    func save(_ order: Order) async throws -> Order
    func get(id: UUID) async throws -> Order
    func getAll() async throws -> [Order]
    func update(_ order: Order) async throws -> Order
    func trackOrder(_ orderId: UUID) -> AsyncThrowingStream<OrderStatus, Error>
}

class OrderRepository: OrderRepositoryProtocol {
    private let remoteDataSource: OrderRemoteDataSourceProtocol
    private let localDataSource: OrderLocalDataSourceProtocol
    private let mapper: OrderMapper
    
    init(
        remoteDataSource: OrderRemoteDataSourceProtocol,
        localDataSource: OrderLocalDataSourceProtocol,
        mapper: OrderMapper = OrderMapper()
    ) {
        self.remoteDataSource = remoteDataSource
        self.localDataSource = localDataSource
        self.mapper = mapper
    }
    
    func save(_ order: Order) async throws -> Order {
        // 1. Remote에 저장
        let dto = mapper.toDTO(order)
        let savedDTO = try await remoteDataSource.createOrder(dto)
        
        // 2. Local에 캐싱
        try await localDataSource.save(savedDTO)
        
        // 3. Domain 모델로 변환
        return mapper.toDomain(savedDTO)
    }
    
    func get(id: UUID) async throws -> Order {
        // 1. Local 확인
        if let localDTO = try? await localDataSource.get(id: id) {
            return mapper.toDomain(localDTO)
        }
        
        // 2. Remote에서 가져오기
        let remoteDTO = try await remoteDataSource.getOrder(id: id)
        
        // 3. Local에 캐싱
        try? await localDataSource.save(remoteDTO)
        
        return mapper.toDomain(remoteDTO)
    }
    
    func trackOrder(_ orderId: UUID) -> AsyncThrowingStream<OrderStatus, Error> {
        remoteDataSource.subscribeToOrderUpdates(orderId: orderId)
            .map { dto in
                OrderStatus(rawValue: dto.status) ?? .unknown
            }
            .eraseToThrowingStream()
    }
}

// Data/DataSources/Remote/OrderRemoteDataSource.swift
protocol OrderRemoteDataSourceProtocol {
    func createOrder(_ order: OrderDTO) async throws -> OrderDTO
    func getOrder(id: UUID) async throws -> OrderDTO
    func subscribeToOrderUpdates(orderId: UUID) -> AsyncThrowingStream<OrderDTO, Error>
}

class OrderRemoteDataSource: OrderRemoteDataSourceProtocol {
    private let apiClient: APIClient
    private let websocket: WebSocketClient
    
    func createOrder(_ order: OrderDTO) async throws -> OrderDTO {
        let response = try await apiClient.post(
            "/orders",
            body: order
        )
        return response
    }
    
    func getOrder(id: UUID) async throws -> OrderDTO {
        try await apiClient.get("/orders/\(id)")
    }
    
    func subscribeToOrderUpdates(orderId: UUID) -> AsyncThrowingStream<OrderDTO, Error> {
        websocket.subscribe(to: "/orders/\(orderId)/updates")
    }
}

// Data/DataSources/Local/OrderLocalDataSource.swift
protocol OrderLocalDataSourceProtocol {
    func save(_ order: OrderDTO) async throws
    func get(id: UUID) async throws -> OrderDTO?
    func getAll() async throws -> [OrderDTO]
    func delete(id: UUID) async throws
}

@ModelActor
actor OrderLocalDataSource: OrderLocalDataSourceProtocol {
    func save(_ orderDTO: OrderDTO) async throws {
        let context = modelContext
        
        // SwiftData 모델로 변환
        let orderEntity = OrderEntity(from: orderDTO)
        context.insert(orderEntity)
        
        try context.save()
    }
    
    func get(id: UUID) async throws -> OrderDTO? {
        let descriptor = FetchDescriptor<OrderEntity>(
            predicate: #Predicate { $0.id == id }
        )
        
        let results = try modelContext.fetch(descriptor)
        return results.first?.toDTO()
    }
}

// Data/DTOs/OrderDTO.swift
struct OrderDTO: Codable {
    let id: String
    let restaurantId: String
    let items: [OrderItemDTO]
    let status: String
    let createdAt: String
    let totalAmount: Double
}

// Data/Mappers/OrderMapper.swift
class OrderMapper {
    func toDomain(_ dto: OrderDTO) -> Order {
        Order(
            id: UUID(uuidString: dto.id) ?? UUID(),
            restaurant: restaurantMapper.toDomain(dto.restaurant),
            items: dto.items.map { itemMapper.toDomain($0) },
            createdAt: ISO8601DateFormatter().date(from: dto.createdAt) ?? Date(),
            status: OrderStatus(rawValue: dto.status) ?? .unknown
        )
    }
    
    func toDTO(_ domain: Order) -> OrderDTO {
        OrderDTO(
            id: domain.id.uuidString,
            restaurantId: domain.restaurant.id.uuidString,
            items: domain.items.map { itemMapper.toDTO($0) },
            status: domain.status.rawValue,
            createdAt: ISO8601DateFormatter().string(from: domain.createdAt),
            totalAmount: NSDecimalNumber(decimal: domain.total).doubleValue
        )
    }
}
```

### 3. Presentation Layer (UI)

```swift
// Presentation/ViewModels/RestaurantListViewModel.swift
@Observable
class RestaurantListViewModel {
    private let getRestaurantsUseCase: GetRestaurantsUseCaseProtocol
    private let coordinator: AppCoordinator
    
    var restaurants: [Restaurant] = []
    var isLoading = false
    var error: Error?
    var selectedCuisine: CuisineType?
    
    var filteredRestaurants: [Restaurant] {
        guard let cuisine = selectedCuisine else {
            return restaurants
        }
        return restaurants.filter { $0.cuisine == cuisine }
    }
    
    init(
        getRestaurantsUseCase: GetRestaurantsUseCaseProtocol,
        coordinator: AppCoordinator
    ) {
        self.getRestaurantsUseCase = getRestaurantsUseCase
        self.coordinator = coordinator
    }
    
    func loadRestaurants() async {
        isLoading = true
        error = nil
        
        do {
            restaurants = try await getRestaurantsUseCase.execute()
        } catch {
            self.error = error
        }
        
        isLoading = false
    }
    
    func selectRestaurant(_ restaurant: Restaurant) {
        coordinator.showRestaurantDetail(restaurant)
    }
}

// Presentation/Views/RestaurantListView.swift
struct RestaurantListView: View {
    @State private var viewModel: RestaurantListViewModel
    
    init(viewModel: RestaurantListViewModel) {
        self._viewModel = State(wrappedValue: viewModel)
    }
    
    var body: some View {
        ScrollView {
            if viewModel.isLoading {
                ProgressView()
                    .frame(maxWidth: .infinity, maxHeight: .infinity)
            } else if let error = viewModel.error {
                ErrorView(error: error) {
                    Task {
                        await viewModel.loadRestaurants()
                    }
                }
            } else {
                LazyVStack(spacing: 16) {
                    ForEach(viewModel.filteredRestaurants) { restaurant in
                        RestaurantCard(restaurant: restaurant) {
                            viewModel.selectRestaurant(restaurant)
                        }
                    }
                }
                .padding()
            }
        }
        .task {
            await viewModel.loadRestaurants()
        }
        .refreshable {
            await viewModel.loadRestaurants()
        }
    }
}

// Presentation/Coordinator/AppCoordinator.swift
@Observable
class AppCoordinator {
    var path = NavigationPath()
    private let container: DIContainer
    
    init(container: DIContainer) {
        self.container = container
    }
    
    func showRestaurantDetail(_ restaurant: Restaurant) {
        path.append(NavigationDestination.restaurantDetail(restaurant))
    }
    
    func showOrderTracking(_ order: Order) {
        path.append(NavigationDestination.orderTracking(order))
    }
    
    func showCheckout() {
        path.append(NavigationDestination.checkout)
    }
    
    @ViewBuilder
    func view(for destination: NavigationDestination) -> some View {
        switch destination {
        case .restaurantDetail(let restaurant):
            let viewModel = container.makeRestaurantDetailViewModel(restaurant: restaurant)
            RestaurantDetailView(viewModel: viewModel)
            
        case .orderTracking(let order):
            let viewModel = container.makeOrderTrackingViewModel(order: order)
            OrderTrackingView(viewModel: viewModel)
            
        case .checkout:
            let viewModel = container.makeCheckoutViewModel()
            CheckoutView(viewModel: viewModel)
        }
    }
}
```

### 4. Dependency Injection Container

```swift
// DI/DIContainer.swift
class DIContainer {
    // MARK: - Singleton Services
    private lazy var apiClient = APIClient()
    private lazy var websocketClient = WebSocketClient()
    private lazy var databaseManager = DatabaseManager()
    
    // MARK: - Data Sources
    private lazy var orderRemoteDataSource = OrderRemoteDataSource(
        apiClient: apiClient,
        websocket: websocketClient
    )
    
    private lazy var orderLocalDataSource = OrderLocalDataSource(
        database: databaseManager
    )
    
    // MARK: - Repositories
    private lazy var orderRepository: OrderRepositoryProtocol = OrderRepository(
        remoteDataSource: orderRemoteDataSource,
        localDataSource: orderLocalDataSource
    )
    
    private lazy var restaurantRepository: RestaurantRepositoryProtocol = RestaurantRepository(
        remoteDataSource: restaurantRemoteDataSource,
        localDataSource: restaurantLocalDataSource
    )
    
    // MARK: - Use Cases
    func makePlaceOrderUseCase() -> PlaceOrderUseCaseProtocol {
        PlaceOrderUseCase(
            orderRepository: orderRepository,
            paymentService: paymentService,
            notificationService: notificationService
        )
    }
    
    func makeTrackOrderUseCase() -> TrackOrderUseCaseProtocol {
        TrackOrderUseCase(orderRepository: orderRepository)
    }
    
    func makeGetRestaurantsUseCase() -> GetRestaurantsUseCaseProtocol {
        GetRestaurantsUseCase(repository: restaurantRepository)
    }
    
    // MARK: - ViewModels
    func makeRestaurantListViewModel(coordinator: AppCoordinator) -> RestaurantListViewModel {
        RestaurantListViewModel(
            getRestaurantsUseCase: makeGetRestaurantsUseCase(),
            coordinator: coordinator
        )
    }
    
    func makeRestaurantDetailViewModel(
        restaurant: Restaurant,
        coordinator: AppCoordinator
    ) -> RestaurantDetailViewModel {
        RestaurantDetailViewModel(
            restaurant: restaurant,
            getMenuUseCase: makeGetMenuUseCase(),
            addToCartUseCase: makeAddToCartUseCase(),
            coordinator: coordinator
        )
    }
    
    func makeCheckoutViewModel(coordinator: AppCoordinator) -> CheckoutViewModel {
        CheckoutViewModel(
            placeOrderUseCase: makePlaceOrderUseCase(),
            getCartUseCase: makeGetCartUseCase(),
            coordinator: coordinator
        )
    }
}

// DI/AppAssembly.swift
class AppAssembly {
    static func assemble() -> some View {
        let container = DIContainer()
        let coordinator = AppCoordinator(container: container)
        
        return AppRootView(coordinator: coordinator)
            .environment(container)
            .environment(coordinator)
    }
}
```

### 5. 테스트

```swift
// Tests/Domain/UseCases/PlaceOrderUseCaseTests.swift
class PlaceOrderUseCaseTests: XCTestCase {
    var sut: PlaceOrderUseCase!
    var mockOrderRepository: MockOrderRepository!
    var mockPaymentService: MockPaymentService!
    var mockNotificationService: MockNotificationService!
    
    override func setUp() {
        super.setUp()
        mockOrderRepository = MockOrderRepository()
        mockPaymentService = MockPaymentService()
        mockNotificationService = MockNotificationService()
        
        sut = PlaceOrderUseCase(
            orderRepository: mockOrderRepository,
            paymentService: mockPaymentService,
            notificationService: mockNotificationService
        )
    }
    
    func testPlaceOrder_WhenRestaurantClosed_ThrowsError() async {
        // Given
        let restaurant = Restaurant.mock(isOpen: false)
        let items = [OrderItem.mock()]
        
        // When/Then
        do {
            _ = try await sut.execute(restaurant: restaurant, items: items)
            XCTFail("Should throw error")
        } catch {
            XCTAssertEqual(error as? OrderError, .restaurantClosed)
        }
    }
    
    func testPlaceOrder_WhenBelowMinimum_ThrowsError() async {
        // Given
        let restaurant = Restaurant.mock(minimumOrder: 100)
        let items = [OrderItem.mock(price: 50)]
        
        // When/Then
        await assertThrowsError {
            _ = try await sut.execute(restaurant: restaurant, items: items)
        } errorHandler: { error in
            XCTAssertEqual(error as? OrderError, .belowMinimumOrder)
        }
    }
    
    func testPlaceOrder_Success() async throws {
        // Given
        let restaurant = Restaurant.mock()
        let items = [OrderItem.mock()]
        mockPaymentService.processResult = .success(PaymentResult.mock())
        mockOrderRepository.saveResult = .success(Order.mock())
        
        // When
        let order = try await sut.execute(restaurant: restaurant, items: items)
        
        // Then
        XCTAssertNotNil(order)
        XCTAssertTrue(mockPaymentService.processCalled)
        XCTAssertTrue(mockOrderRepository.saveCalled)
        XCTAssertTrue(mockNotificationService.sendOrderConfirmationCalled)
    }
}

// Tests/Presentation/ViewModels/RestaurantListViewModelTests.swift
@MainActor
class RestaurantListViewModelTests: XCTestCase {
    var sut: RestaurantListViewModel!
    var mockUseCase: MockGetRestaurantsUseCase!
    var mockCoordinator: MockAppCoordinator!
    
    override func setUp() {
        super.setUp()
        mockUseCase = MockGetRestaurantsUseCase()
        mockCoordinator = MockAppCoordinator()
        sut = RestaurantListViewModel(
            getRestaurantsUseCase: mockUseCase,
            coordinator: mockCoordinator
        )
    }
    
    func testLoadRestaurants_Success() async {
        // Given
        let expectedRestaurants = [Restaurant.mock()]
        mockUseCase.executeResult = .success(expectedRestaurants)
        
        // When
        await sut.loadRestaurants()
        
        // Then
        XCTAssertEqual(sut.restaurants, expectedRestaurants)
        XCTAssertFalse(sut.isLoading)
        XCTAssertNil(sut.error)
    }
    
    func testFilterByCuisine() {
        // Given
        sut.restaurants = [
            Restaurant.mock(cuisine: .italian),
            Restaurant.mock(cuisine: .japanese),
            Restaurant.mock(cuisine: .italian)
        ]
        
        // When
        sut.selectedCuisine = .italian
        
        // Then
        XCTAssertEqual(sut.filteredRestaurants.count, 2)
        XCTAssertTrue(sut.filteredRestaurants.allSatisfy { $0.cuisine == .italian })
    }
}
```

## 모듈화 전략

```swift
// Package.swift (Swift Package Manager)
let package = Package(
    name: "FoodDeliveryApp",
    platforms: [.iOS(.v17)],
    products: [
        .library(name: "Domain", targets: ["Domain"]),
        .library(name: "Data", targets: ["Data"]),
        .library(name: "Presentation", targets: ["Presentation"]),
        .library(name: "Core", targets: ["Core"])
    ],
    dependencies: [
        .package(url: "https://github.com/apple/swift-async-algorithms", from: "1.0.0")
    ],
    targets: [
        // Domain Layer - 외부 의존성 없음
        .target(
            name: "Domain",
            dependencies: []
        ),
        
        // Data Layer - Domain에만 의존
        .target(
            name: "Data",
            dependencies: ["Domain", "Core"]
        ),
        
        // Presentation Layer - Domain에만 의존
        .target(
            name: "Presentation",
            dependencies: ["Domain", "Core"]
        ),
        
        // Core - 공통 유틸리티
        .target(
            name: "Core",
            dependencies: [
                .product(name: "AsyncAlgorithms", package: "swift-async-algorithms")
            ]
        ),
        
        // Tests
        .testTarget(
            name: "DomainTests",
            dependencies: ["Domain"]
        ),
        .testTarget(
            name: "DataTests",
            dependencies: ["Data"]
        ),
        .testTarget(
            name: "PresentationTests",
            dependencies: ["Presentation"]
        )
    ]
)
```

## 에러 처리 전략

```swift
// Core/Errors/AppError.swift
enum AppError: LocalizedError {
    case network(NetworkError)
    case business(BusinessError)
    case validation(ValidationError)
    case unknown(Error)
    
    var errorDescription: String? {
        switch self {
        case .network(let error):
            return error.userMessage
        case .business(let error):
            return error.userMessage
        case .validation(let error):
            return error.userMessage
        case .unknown:
            return "알 수 없는 오류가 발생했습니다"
        }
    }
    
    var recoverySuggestion: String? {
        switch self {
        case .network:
            return "네트워크 연결을 확인해주세요"
        case .business(let error):
            return error.recoverySuggestion
        case .validation:
            return "입력 값을 확인해주세요"
        case .unknown:
            return "잠시 후 다시 시도해주세요"
        }
    }
}

// Presentation/Views/ErrorView.swift
struct ErrorView: View {
    let error: Error
    let retry: (() async -> Void)?
    
    var body: some View {
        VStack(spacing: 20) {
            Image(systemName: "exclamationmark.triangle")
                .font(.largeTitle)
                .foregroundColor(.orange)
            
            Text(error.localizedDescription)
                .font(.headline)
                .multilineTextAlignment(.center)
            
            if let suggestion = (error as? AppError)?.recoverySuggestion {
                Text(suggestion)
                    .font(.subheadline)
                    .foregroundColor(.secondary)
                    .multilineTextAlignment(.center)
            }
            
            if let retry = retry {
                Button("다시 시도") {
                    Task {
                        await retry()
                    }
                }
                .buttonStyle(.borderedProminent)
            }
        }
        .padding()
    }
}
```

## 성능 최적화

```swift
// 1. 메모리 관리
class ImageCache {
    private let cache = NSCache<NSString, UIImage>()
    private let diskCache: DiskCache
    
    func image(for url: URL) async -> UIImage? {
        let key = url.absoluteString as NSString
        
        // Memory cache
        if let cached = cache.object(forKey: key) {
            return cached
        }
        
        // Disk cache
        if let diskCached = await diskCache.image(for: url) {
            cache.setObject(diskCached, forKey: key)
            return diskCached
        }
        
        // Download
        guard let image = await downloadImage(from: url) else {
            return nil
        }
        
        cache.setObject(image, forKey: key)
        await diskCache.store(image, for: url)
        
        return image
    }
}

// 2. 데이터 페이징
class PaginatedDataSource<T> {
    private var items: [T] = []
    private var currentPage = 0
    private var hasMorePages = true
    private let pageSize: Int
    private let fetcher: (Int, Int) async throws -> [T]
    
    func loadNextPage() async throws -> [T] {
        guard hasMorePages else { return [] }
        
        let newItems = try await fetcher(currentPage, pageSize)
        
        if newItems.count < pageSize {
            hasMorePages = false
        }
        
        items.append(contentsOf: newItems)
        currentPage += 1
        
        return newItems
    }
}

// 3. 뷰 재사용
struct LazyView<Content: View>: View {
    let build: () -> Content
    
    var body: Content {
        build()
    }
}

extension NavigationLink {
    func lazy<V: View>(_ destination: @autoclosure @escaping () -> V) -> some View {
        self.destination(LazyView(build: destination))
    }
}
```

## 정리: Clean Architecture의 이점

### 1. 테스트 가능성
- 각 레이어를 독립적으로 테스트
- Mock 객체로 쉽게 대체
- 비즈니스 로직 단위 테스트

### 2. 유지보수성
- 변경의 영향 범위 최소화
- 명확한 책임 분리
- 코드 이해 용이

### 3. 확장성
- 새 기능 추가 용이
- 기존 코드 수정 최소화
- 모듈화로 팀 협업 개선

### 4. 플랫폼 독립성
- Domain 레이어는 플랫폼 독립적
- UI 프레임워크 변경 가능
- 다른 플랫폼으로 이식 가능

## First Principles: 왜 이런 구조인가?

1. **변경의 이유 분리**: 각 레이어는 다른 이유로 변경된다
2. **의존성 역전**: 추상화에 의존, 구체화에 의존하지 않음
3. **단일 책임**: 각 클래스는 하나의 책임만
4. **개방-폐쇄**: 확장에는 열려있고 수정에는 닫혀있음
5. **인터페이스 분리**: 필요한 인터페이스만 의존

## 다음 레벨로

아키텍처를 넘어 시스템 통합으로 나아갈 시간이다.

→ [L5: 시스템 통합 - Widget, App Intents, Live Activities](../L5_systems/00_system_integration.md)

---

*"Architecture is about the important stuff. Whatever that is." - Ralph Johnson*