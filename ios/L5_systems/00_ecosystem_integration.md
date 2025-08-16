# L5: 생태계 통합 - 앱이 시스템의 일부가 되다

## 핵심: 앱의 경계를 넘어서

> "The best technology is the one that becomes invisible." - Mark Weiser

진정한 iOS 앱은 시스템과 하나가 된다. 사용자는 앱을 열지 않고도 가치를 얻고, 시리를 통해 대화하며, 위젯으로 정보를 확인한다.

## 1. App Intents - Siri와의 대화

### 기본 Intent 정의

```swift
import AppIntents

// MARK: - 기본 Intent
struct OrderFoodIntent: AppIntent {
    static var title: LocalizedStringResource = "음식 주문하기"
    static var description = IntentDescription("좋아하는 음식을 빠르게 주문합니다")
    static var openAppWhenRun: Bool = false
    
    // 매개변수
    @Parameter(title: "레스토랑", description: "주문할 레스토랑")
    var restaurant: RestaurantEntity?
    
    @Parameter(title: "음식", description: "주문할 음식")
    var menuItem: MenuItemEntity?
    
    @Parameter(title: "수량", description: "주문 수량", default: 1)
    var quantity: Int
    
    static var parameterSummary: some ParameterSummary {
        Summary("\(\.$restaurant)에서 \(\.$menuItem) \(\.$quantity)개 주문하기") {
            \.$restaurant
            \.$menuItem
            \.$quantity
        }
    }
    
    func perform() async throws -> some IntentResult {
        // 1. 레스토랑 검증
        guard let restaurant = restaurant else {
            throw OrderError.noRestaurant
        }
        
        // 2. 메뉴 아이템 검증
        guard let menuItem = menuItem else {
            throw OrderError.noMenuItem
        }
        
        // 3. 실제 주문 처리
        let orderService = DependencyContainer.shared.orderService
        let order = try await orderService.placeOrder(
            restaurant: restaurant.restaurant,
            items: [OrderItem(menuItem: menuItem.menuItem, quantity: quantity)]
        )
        
        // 4. 결과 반환
        return .result(
            value: OrderEntity(order: order),
            dialog: IntentDialog("주문이 완료되었습니다. 예상 배달 시간은 \(order.estimatedDeliveryTime)입니다.")
        )
    }
}

// MARK: - Entity 정의
struct RestaurantEntity: AppEntity {
    static var typeDisplayRepresentation = TypeDisplayRepresentation(name: "레스토랑")
    static var defaultQuery = RestaurantQuery()
    
    let id: UUID
    let name: String
    let cuisine: String
    let rating: Double
    
    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(
            title: "\(name)",
            subtitle: "\(cuisine) • ⭐ \(rating, specifier: "%.1f")"
        )
    }
    
    // 실제 도메인 모델과 연결
    var restaurant: Restaurant {
        Restaurant(
            id: id,
            name: name,
            cuisine: CuisineType(rawValue: cuisine) ?? .other,
            rating: rating
        )
    }
    
    init(restaurant: Restaurant) {
        self.id = restaurant.id
        self.name = restaurant.name
        self.cuisine = restaurant.cuisine.rawValue
        self.rating = restaurant.rating
    }
}

// MARK: - Entity Query
struct RestaurantQuery: EntityQuery {
    func entities(for identifiers: [UUID]) async throws -> [RestaurantEntity] {
        let restaurantService = DependencyContainer.shared.restaurantService
        let restaurants = try await restaurantService.getRestaurants(ids: identifiers)
        return restaurants.map(RestaurantEntity.init)
    }
    
    func suggestedEntities() async throws -> [RestaurantEntity] {
        let restaurantService = DependencyContainer.shared.restaurantService
        let popularRestaurants = try await restaurantService.getPopularRestaurants()
        return popularRestaurants.map(RestaurantEntity.init)
    }
    
    func entities(matching query: String) async throws -> [RestaurantEntity] {
        let restaurantService = DependencyContainer.shared.restaurantService
        let searchResults = try await restaurantService.searchRestaurants(query: query)
        return searchResults.map(RestaurantEntity.init)
    }
}

// MARK: - Shortcuts Provider
struct FoodDeliveryShortcutsProvider: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: OrderFoodIntent(),
            phrases: [
                "음식을 \(.applicationName)으로 주문해줘",
                "\(.applicationName)에서 치킨 주문해줘",
                "배달음식 주문하기"
            ],
            shortTitle: "음식 주문",
            systemImageName: "fork.knife.circle"
        )
    }
}
```

### 복잡한 대화형 Intent

```swift
// MARK: - 다단계 주문 Intent
struct DetailedOrderIntent: AppIntent {
    static var title: LocalizedStringResource = "상세 주문하기"
    static var description = IntentDescription("단계별로 음식을 주문합니다")
    
    @Parameter(title: "음식 종류")
    var cuisineType: CuisineTypeEntity?
    
    @Parameter(title: "배달 주소")
    var deliveryAddress: String?
    
    @Parameter(title: "특별 요청사항")
    var specialInstructions: String?
    
    func perform() async throws -> some IntentResult & ProvidesDialog {
        // 1. 요리 종류 선택
        let selectedCuisine: CuisineTypeEntity
        if let cuisine = cuisineType {
            selectedCuisine = cuisine
        } else {
            let availableCuisines = try await getCuisineTypes()
            selectedCuisine = try await $cuisineType.requestValue(
                dialog: IntentDialog("어떤 종류의 음식을 주문하시겠습니까?"),
                options: .init(availableCuisines)
            )
        }
        
        // 2. 레스토랑 선택
        let restaurants = try await getRestaurants(for: selectedCuisine)
        guard !restaurants.isEmpty else {
            throw OrderError.noRestaurantsAvailable
        }
        
        let selectedRestaurant = try await requestRestaurant(from: restaurants)
        
        // 3. 메뉴 선택
        let menuItems = try await getMenuItems(for: selectedRestaurant)
        let selectedItems = try await requestMenuItems(from: menuItems)
        
        // 4. 배달 주소 확인
        let address = try await confirmDeliveryAddress()
        
        // 5. 주문 확인
        let orderSummary = createOrderSummary(
            restaurant: selectedRestaurant,
            items: selectedItems,
            address: address
        )
        
        let confirmation = try await requestConfirmation(for: orderSummary)
        guard confirmation else {
            return .result(dialog: IntentDialog("주문이 취소되었습니다."))
        }
        
        // 6. 실제 주문 처리
        let order = try await placeOrder(
            restaurant: selectedRestaurant,
            items: selectedItems,
            address: address,
            specialInstructions: specialInstructions
        )
        
        return .result(
            dialog: IntentDialog("""
            주문이 완료되었습니다!
            주문 번호: \(order.orderNumber)
            예상 배달 시간: \(order.estimatedDeliveryTime)
            """)
        )
    }
    
    // MARK: - Helper Methods
    private func requestRestaurant(from restaurants: [RestaurantEntity]) async throws -> RestaurantEntity {
        if restaurants.count == 1 {
            return restaurants[0]
        }
        
        return try await IntentDialog("어느 레스토랑에서 주문하시겠습니까?")
            .requestValue(from: restaurants)
    }
    
    private func requestMenuItems(from menuItems: [MenuItemEntity]) async throws -> [MenuItemEntity] {
        return try await IntentDialog("어떤 음식을 주문하시겠습니까?")
            .requestValue(from: menuItems, allowsMultipleSelection: true)
    }
    
    private func confirmDeliveryAddress() async throws -> String {
        if let address = deliveryAddress {
            let confirmation = try await IntentDialog("배달 주소가 '\(address)'가 맞습니까?")
                .requestConfirmation()
            
            if confirmation {
                return address
            }
        }
        
        return try await IntentDialog("배달 주소를 알려주세요")
            .requestValue(String.self)
    }
    
    private func requestConfirmation(for summary: OrderSummary) async throws -> Bool {
        return try await IntentDialog("""
        주문 내용을 확인해주세요:
        \(summary.description)
        
        총 금액: \(summary.totalAmount)원
        주문하시겠습니까?
        """).requestConfirmation()
    }
}
```

## 2. 위젯 시스템 - 정보의 창

### TimelineProvider 구현

```swift
import WidgetKit
import SwiftUI

// MARK: - Widget Entry
struct OrderStatusEntry: TimelineEntry {
    let date: Date
    let order: Order?
    let relevance: TimelineEntryRelevance?
    
    var isPlaceholder: Bool { order == nil }
}

// MARK: - Timeline Provider
struct OrderStatusProvider: TimelineProvider {
    func placeholder(in context: Context) -> OrderStatusEntry {
        OrderStatusEntry(
            date: Date(),
            order: Order.placeholder,
            relevance: nil
        )
    }
    
    func getSnapshot(in context: Context, completion: @escaping (OrderStatusEntry) -> Void) {
        let entry: OrderStatusEntry
        
        if context.isPreview {
            entry = OrderStatusEntry(
                date: Date(),
                order: Order.sample,
                relevance: TimelineEntryRelevance(score: 1.0)
            )
        } else {
            // 실제 데이터 로드
            entry = OrderStatusEntry(
                date: Date(),
                order: getCurrentOrder(),
                relevance: calculateRelevance()
            )
        }
        
        completion(entry)
    }
    
    func getTimeline(in context: Context, completion: @escaping (Timeline<OrderStatusEntry>) -> Void) {
        Task {
            let entries = await generateEntries()
            let timeline = Timeline(
                entries: entries,
                policy: .after(Date().addingTimeInterval(60)) // 1분 후 다시 업데이트
            )
            completion(timeline)
        }
    }
    
    private func generateEntries() async -> [OrderStatusEntry] {
        let now = Date()
        var entries: [OrderStatusEntry] = []
        
        // 현재 주문 확인
        guard let currentOrder = await getCurrentActiveOrder() else {
            // 주문이 없으면 빈 엔트리
            let entry = OrderStatusEntry(
                date: now,
                order: nil,
                relevance: nil
            )
            return [entry]
        }
        
        // 주문 상태에 따라 타임라인 생성
        switch currentOrder.status {
        case .pending:
            entries.append(contentsOf: await generatePendingTimeline(for: currentOrder, from: now))
        case .confirmed:
            entries.append(contentsOf: await generateConfirmedTimeline(for: currentOrder, from: now))
        case .preparing:
            entries.append(contentsOf: await generatePreparingTimeline(for: currentOrder, from: now))
        case .onTheWay:
            entries.append(contentsOf: await generateDeliveryTimeline(for: currentOrder, from: now))
        case .delivered:
            entries.append(OrderStatusEntry(
                date: now,
                order: currentOrder,
                relevance: TimelineEntryRelevance(score: 0.1) // 낮은 관련성
            ))
        default:
            break
        }
        
        return entries
    }
    
    private func generateDeliveryTimeline(for order: Order, from startDate: Date) async -> [OrderStatusEntry] {
        var entries: [OrderStatusEntry] = []
        let deliveryTime = order.estimatedDeliveryTime
        let interval: TimeInterval = 30 // 30초마다 업데이트
        
        for i in 0..<Int(deliveryTime / interval) {
            let date = startDate.addingTimeInterval(interval * Double(i))
            let remainingTime = deliveryTime - interval * Double(i)
            
            var updatedOrder = order
            updatedOrder.estimatedRemainingTime = remainingTime
            
            // 배달이 가까워질수록 높은 관련성
            let relevanceScore = max(0.1, 1.0 - (remainingTime / deliveryTime))
            
            let entry = OrderStatusEntry(
                date: date,
                order: updatedOrder,
                relevance: TimelineEntryRelevance(score: relevanceScore)
            )
            entries.append(entry)
        }
        
        return entries
    }
    
    private func calculateRelevance() -> TimelineEntryRelevance? {
        guard let order = getCurrentOrder() else { return nil }
        
        switch order.status {
        case .onTheWay:
            return TimelineEntryRelevance(score: 1.0, duration: order.estimatedRemainingTime)
        case .preparing:
            return TimelineEntryRelevance(score: 0.8, duration: order.estimatedRemainingTime)
        case .confirmed:
            return TimelineEntryRelevance(score: 0.6)
        case .delivered:
            return TimelineEntryRelevance(score: 0.1, duration: 300) // 5분간 낮은 관련성
        default:
            return nil
        }
    }
}

// MARK: - Widget View
struct OrderStatusWidgetView: View {
    let entry: OrderStatusEntry
    
    @Environment(\.widgetFamily) var family
    @Environment(\.colorScheme) var colorScheme
    
    var body: some View {
        if let order = entry.order {
            switch family {
            case .systemSmall:
                SmallOrderStatusView(order: order)
            case .systemMedium:
                MediumOrderStatusView(order: order)
            case .systemLarge:
                LargeOrderStatusView(order: order)
            case .accessoryInline:
                InlineOrderStatusView(order: order)
            case .accessoryRectangular:
                RectangularOrderStatusView(order: order)
            case .accessoryCircular:
                CircularOrderStatusView(order: order)
            default:
                SmallOrderStatusView(order: order)
            }
        } else {
            NoOrderView()
        }
    }
}

struct SmallOrderStatusView: View {
    let order: Order
    
    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            Text(order.restaurant.name)
                .font(.caption)
                .fontWeight(.medium)
                .lineLimit(1)
            
            Text(order.status.displayName)
                .font(.caption2)
                .foregroundColor(.secondary)
            
            Spacer()
            
            HStack {
                Image(systemName: order.status.iconName)
                    .foregroundColor(order.status.color)
                
                if let remainingTime = order.estimatedRemainingTime {
                    Text(formatRemainingTime(remainingTime))
                        .font(.caption)
                        .fontWeight(.semibold)
                }
            }
        }
        .padding(8)
        .background(backgroundGradient)
        .clipShape(RoundedRectangle(cornerRadius: 8))
        .widgetURL(order.deepLinkURL)
    }
    
    private var backgroundGradient: LinearGradient {
        LinearGradient(
            colors: [order.status.color.opacity(0.1), Color.clear],
            startPoint: .topLeading,
            endPoint: .bottomTrailing
        )
    }
}

struct MediumOrderStatusView: View {
    let order: Order
    
    var body: some View {
        HStack(spacing: 12) {
            VStack(alignment: .leading, spacing: 6) {
                Text(order.restaurant.name)
                    .font(.headline)
                    .fontWeight(.bold)
                
                Text(order.items.map(\.name).joined(separator: ", "))
                    .font(.caption)
                    .foregroundColor(.secondary)
                    .lineLimit(2)
                
                Label(order.status.displayName, systemImage: order.status.iconName)
                    .font(.caption)
                    .foregroundColor(order.status.color)
            }
            
            Spacer()
            
            VStack(alignment: .trailing, spacing: 6) {
                if let remainingTime = order.estimatedRemainingTime {
                    Text(formatRemainingTime(remainingTime))
                        .font(.title2)
                        .fontWeight(.bold)
                        .foregroundColor(order.status.color)
                    
                    Text("남음")
                        .font(.caption2)
                        .foregroundColor(.secondary)
                }
                
                if order.status == .onTheWay {
                    Button(intent: TrackOrderIntent(orderId: order.id)) {
                        Text("추적")
                            .font(.caption)
                            .padding(.horizontal, 8)
                            .padding(.vertical, 4)
                            .background(Color.accentColor)
                            .foregroundColor(.white)
                            .clipShape(Capsule())
                    }
                    .buttonStyle(.plain)
                }
            }
        }
        .padding()
        .background(backgroundGradient)
        .clipShape(RoundedRectangle(cornerRadius: 12))
        .widgetURL(order.deepLinkURL)
    }
}
```

## 3. Live Activities - 실시간 상태 표시

### Activity Attributes

```swift
import ActivityKit

// MARK: - Live Activity 속성 정의
struct OrderTrackingAttributes: ActivityAttributes {
    public struct ContentState: Codable, Hashable {
        var status: OrderStatus
        var estimatedDeliveryTime: Date
        var driverLocation: CLLocationCoordinate2D?
        var driverName: String?
        var lastUpdate: Date
        
        var isDriverAssigned: Bool {
            driverName != nil
        }
        
        var timeRemaining: TimeInterval {
            estimatedDeliveryTime.timeIntervalSinceNow
        }
    }
    
    // 변경되지 않는 정보
    let orderNumber: String
    let restaurantName: String
    let totalAmount: Decimal
    let items: [String]
    let customerAddress: String
}

// MARK: - Live Activity 관리자
@Observable
class LiveActivityManager {
    private var currentActivity: Activity<OrderTrackingAttributes>?
    
    func startOrderTracking(for order: Order) async {
        guard ActivityAuthorizationInfo().areActivitiesEnabled else {
            print("Live Activities are not enabled")
            return
        }
        
        let attributes = OrderTrackingAttributes(
            orderNumber: order.orderNumber,
            restaurantName: order.restaurant.name,
            totalAmount: order.total,
            items: order.items.map(\.name),
            customerAddress: order.deliveryAddress
        )
        
        let initialState = OrderTrackingAttributes.ContentState(
            status: order.status,
            estimatedDeliveryTime: order.estimatedDeliveryTime,
            driverLocation: nil,
            driverName: nil,
            lastUpdate: Date()
        )
        
        do {
            currentActivity = try Activity.request(
                attributes: attributes,
                content: .init(state: initialState, staleDate: nil),
                pushType: .token
            )
            
            print("Live Activity started: \(currentActivity?.id ?? "unknown")")
            
            // Push token 등록
            if let activity = currentActivity {
                await registerPushToken(for: activity)
            }
            
        } catch {
            print("Failed to start Live Activity: \(error)")
        }
    }
    
    func updateOrderStatus(
        status: OrderStatus,
        driverName: String? = nil,
        driverLocation: CLLocationCoordinate2D? = nil,
        estimatedDeliveryTime: Date? = nil
    ) async {
        guard let activity = currentActivity else { return }
        
        let newState = OrderTrackingAttributes.ContentState(
            status: status,
            estimatedDeliveryTime: estimatedDeliveryTime ?? activity.content.state.estimatedDeliveryTime,
            driverLocation: driverLocation,
            driverName: driverName,
            lastUpdate: Date()
        )
        
        await activity.update(.init(state: newState, staleDate: nil))
        
        // 배달 완료 시 종료
        if status == .delivered {
            await endOrderTracking()
        }
    }
    
    func endOrderTracking() async {
        guard let activity = currentActivity else { return }
        
        let finalState = OrderTrackingAttributes.ContentState(
            status: .delivered,
            estimatedDeliveryTime: Date(),
            driverLocation: nil,
            driverName: activity.content.state.driverName,
            lastUpdate: Date()
        )
        
        await activity.end(.init(state: finalState, staleDate: Date()), dismissalPolicy: .immediate)
        currentActivity = nil
    }
    
    private func registerPushToken(for activity: Activity<OrderTrackingAttributes>) async {
        for await data in activity.pushTokenUpdates {
            let token = data.map { String(format: "%02x", $0) }.joined()
            
            // 서버에 토큰 등록
            try? await APIClient.shared.registerActivityPushToken(
                activityId: activity.id,
                token: token,
                orderNumber: activity.attributes.orderNumber
            )
        }
    }
}

// MARK: - Live Activity UI
struct OrderTrackingLiveActivity: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: OrderTrackingAttributes.self) { context in
            // 잠금 화면 표시
            LockScreenLiveActivityView(context: context)
        } dynamicIsland: { context in
            // Dynamic Island 표시
            DynamicIsland {
                // Expanded UI
                DynamicIslandExpandedRegion(.leading) {
                    VStack(alignment: .leading) {
                        Text(context.attributes.restaurantName)
                            .font(.caption)
                            .fontWeight(.semibold)
                        
                        Text("주문 #\(context.attributes.orderNumber)")
                            .font(.caption2)
                            .foregroundColor(.secondary)
                    }
                }
                
                DynamicIslandExpandedRegion(.trailing) {
                    VStack(alignment: .trailing) {
                        Text(formatTimeRemaining(context.state.timeRemaining))
                            .font(.caption)
                            .fontWeight(.bold)
                            .foregroundColor(statusColor(context.state.status))
                        
                        Text(context.state.status.displayName)
                            .font(.caption2)
                            .foregroundColor(.secondary)
                    }
                }
                
                DynamicIslandExpandedRegion(.bottom) {
                    if let driverName = context.state.driverName {
                        HStack {
                            Image(systemName: "car.fill")
                                .foregroundColor(.accentColor)
                            
                            Text("배달원: \(driverName)")
                                .font(.caption)
                            
                            Spacer()
                            
                            Button(intent: CallDriverIntent()) {
                                Image(systemName: "phone.fill")
                                    .foregroundColor(.green)
                            }
                            .buttonStyle(.plain)
                        }
                        .padding(.horizontal)
                    } else {
                        ProgressView()
                            .scaleEffect(0.8)
                    }
                }
            } compactLeading: {
                Image(systemName: context.state.status.iconName)
                    .foregroundColor(statusColor(context.state.status))
            } compactTrailing: {
                Text(formatTimeRemaining(context.state.timeRemaining))
                    .font(.caption2)
                    .fontWeight(.semibold)
            } minimal: {
                Image(systemName: context.state.status.iconName)
                    .foregroundColor(statusColor(context.state.status))
            }
        }
    }
}

struct LockScreenLiveActivityView: View {
    let context: ActivityViewContext<OrderTrackingAttributes>
    
    var body: some View {
        VStack(spacing: 12) {
            // 헤더
            HStack {
                VStack(alignment: .leading) {
                    Text(context.attributes.restaurantName)
                        .font(.headline)
                        .fontWeight(.semibold)
                    
                    Text("주문 #\(context.attributes.orderNumber)")
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
                
                Spacer()
                
                VStack(alignment: .trailing) {
                    Text(formatTimeRemaining(context.state.timeRemaining))
                        .font(.title3)
                        .fontWeight(.bold)
                        .foregroundColor(statusColor(context.state.status))
                    
                    Text("예상 도착")
                        .font(.caption2)
                        .foregroundColor(.secondary)
                }
            }
            
            // 상태 바
            HStack(spacing: 8) {
                ForEach(OrderStatus.trackingSteps, id: \.self) { step in
                    StatusDot(
                        status: step,
                        isActive: step.rawValue <= context.state.status.rawValue,
                        isCurrent: step == context.state.status
                    )
                    
                    if step != OrderStatus.trackingSteps.last {
                        Rectangle()
                            .fill(step.rawValue < context.state.status.rawValue ? 
                                  Color.accentColor : Color.secondary.opacity(0.3))
                            .frame(height: 2)
                    }
                }
            }
            
            // 배달원 정보 (있는 경우)
            if let driverName = context.state.driverName {
                HStack {
                    Image(systemName: "car.fill")
                        .foregroundColor(.accentColor)
                    
                    Text("배달원: \(driverName)")
                        .font(.caption)
                    
                    Spacer()
                    
                    Button(intent: TrackLocationIntent()) {
                        HStack(spacing: 4) {
                            Image(systemName: "location.fill")
                            Text("위치")
                        }
                        .font(.caption)
                        .padding(.horizontal, 8)
                        .padding(.vertical, 4)
                        .background(Color.accentColor.opacity(0.2))
                        .foregroundColor(.accentColor)
                        .clipShape(Capsule())
                    }
                    .buttonStyle(.plain)
                }
            }
        }
        .padding()
        .activityBackgroundTint(Color.black.opacity(0.1))
        .activitySystemActionForegroundColor(.accentColor)
    }
}

struct StatusDot: View {
    let status: OrderStatus
    let isActive: Bool
    let isCurrent: Bool
    
    var body: some View {
        ZStack {
            Circle()
                .fill(isActive ? status.color : Color.secondary.opacity(0.3))
                .frame(width: 12, height: 12)
            
            if isCurrent {
                Circle()
                    .fill(Color.white)
                    .frame(width: 6, height: 6)
            }
        }
    }
}
```

## 4. Control Center 통합

### iOS 18 Control Widget

```swift
import WidgetKit
import SwiftUI

// MARK: - Control Widget
struct QuickOrderControl: ControlWidget {
    static let kind: String = "QuickOrderControl"
    
    var body: some ControlWidgetConfiguration {
        AppIntentControlConfiguration(
            kind: Self.kind,
            intent: QuickOrderControlIntent.self
        ) { configuration in
            ControlWidgetButton(
                action: QuickOrderControlIntent()
            ) {
                Label("빠른 주문", systemImage: "fork.knife.circle.fill")
                    .foregroundStyle(.white)
            }
            .controlWidgetActionHint("자주 주문하는 음식을 빠르게 주문합니다")
        }
        .displayName("빠른 주문")
        .description("자주 주문하는 음식을 한 번에 주문할 수 있습니다")
    }
}

// MARK: - Control Intent
struct QuickOrderControlIntent: AppIntent {
    static var title: LocalizedStringResource = "빠른 주문"
    static var description = IntentDescription("자주 주문하는 음식을 빠르게 주문합니다")
    static var openAppWhenRun: Bool = false
    
    func perform() async throws -> some IntentResult {
        // 1. 최근 주문 기록 확인
        let orderHistory = try await OrderHistoryService.shared.getRecentOrders()
        guard let lastOrder = orderHistory.first else {
            // 주문 기록이 없으면 앱으로 이동
            return .result(opensIntent: SelectRestaurantIntent())
        }
        
        // 2. 같은 주문 재주문
        let orderService = DependencyContainer.shared.orderService
        let newOrder = try await orderService.reorder(lastOrder)
        
        return .result(
            dialog: IntentDialog("""
            \(lastOrder.restaurant.name)에서 재주문이 완료되었습니다.
            주문 번호: \(newOrder.orderNumber)
            예상 배달 시간: \(newOrder.estimatedDeliveryTime)
            """)
        )
    }
}

// MARK: - Toggle Control (배달 알림)
struct DeliveryNotificationToggle: ControlWidget {
    static let kind: String = "DeliveryNotificationToggle"
    
    var body: some ControlWidgetConfiguration {
        AppIntentControlConfiguration(
            kind: Self.kind,
            intent: ToggleDeliveryNotificationIntent.self
        ) { configuration in
            ControlWidgetToggle(
                isOn: configuration.isEnabled
            ) { isOn in
                ToggleDeliveryNotificationIntent(isEnabled: isOn)
            } label: {
                Label("배달 알림", systemImage: "bell.circle.fill")
            }
            .controlWidgetActionHint("배달 상태 알림을 켜거나 끕니다")
        }
        .displayName("배달 알림")
        .description("주문 상태 변경 알림을 제어합니다")
    }
}

struct ToggleDeliveryNotificationIntent: AppIntent {
    static var title: LocalizedStringResource = "배달 알림 설정"
    static var openAppWhenRun: Bool = false
    
    @Parameter(title: "알림 활성화")
    var isEnabled: Bool
    
    init() {
        // 현재 설정 로드
        self.isEnabled = UserDefaults.standard.bool(forKey: "deliveryNotificationsEnabled")
    }
    
    init(isEnabled: Bool) {
        self.isEnabled = isEnabled
    }
    
    func perform() async throws -> some IntentResult {
        // 설정 저장
        UserDefaults.standard.set(isEnabled, forKey: "deliveryNotificationsEnabled")
        
        // 알림 권한 확인 및 요청
        if isEnabled {
            let notificationService = NotificationService.shared
            try await notificationService.requestPermission()
        }
        
        return .result(
            value: isEnabled,
            dialog: IntentDialog(isEnabled ? "배달 알림이 활성화되었습니다" : "배달 알림이 비활성화되었습니다")
        )
    }
}
```

## 5. Spotlight 검색 통합

### Core Spotlight 인덱싱

```swift
import CoreSpotlight
import MobileCoreServices

// MARK: - Spotlight 관리자
class SpotlightManager {
    static let shared = SpotlightManager()
    private init() {}
    
    func indexRestaurants(_ restaurants: [Restaurant]) async {
        let searchableItems = restaurants.map { restaurant in
            let attributeSet = CSSearchableItemAttributeSet(itemContentType: UTType.data.identifier)
            
            // 기본 정보
            attributeSet.title = restaurant.name
            attributeSet.contentDescription = "\(restaurant.cuisine.displayName) • ⭐ \(restaurant.rating)"
            attributeSet.keywords = [
                restaurant.name,
                restaurant.cuisine.displayName,
                "음식점", "배달", "주문"
            ]
            
            // 위치 정보
            if let coordinate = restaurant.coordinate {
                attributeSet.latitude = NSNumber(value: coordinate.latitude)
                attributeSet.longitude = NSNumber(value: coordinate.longitude)
            }
            
            // 평점
            attributeSet.rating = NSNumber(value: restaurant.rating)
            
            // 썸네일
            if let imageData = restaurant.thumbnailImageData {
                attributeSet.thumbnailData = imageData
            }
            
            // 커스텀 속성
            attributeSet.setValue(restaurant.cuisine.rawValue, forKey: "cuisine")
            attributeSet.setValue(restaurant.deliveryTime, forKey: "deliveryTime")
            attributeSet.setValue(restaurant.minimumOrder, forKey: "minimumOrder")
            
            return CSSearchableItem(
                uniqueIdentifier: "restaurant_\(restaurant.id)",
                domainIdentifier: "restaurants",
                attributeSet: attributeSet
            )
        }
        
        do {
            try await CSSearchableIndex.default().indexSearchableItems(searchableItems)
            print("Indexed \(searchableItems.count) restaurants")
        } catch {
            print("Failed to index restaurants: \(error)")
        }
    }
    
    func indexMenuItems(_ items: [MenuItem], restaurantId: UUID) async {
        let searchableItems = items.map { item in
            let attributeSet = CSSearchableItemAttributeSet(itemContentType: UTType.data.identifier)
            
            attributeSet.title = item.name
            attributeSet.contentDescription = "\(item.description) • \(item.price)원"
            attributeSet.keywords = [
                item.name,
                item.category.displayName,
                "음식", "메뉴", "주문"
            ] + item.ingredients
            
            // 가격
            attributeSet.setValue(item.price, forKey: "price")
            attributeSet.setValue(item.category.rawValue, forKey: "category")
            attributeSet.setValue(restaurantId.uuidString, forKey: "restaurantId")
            
            // 이미지
            if let imageData = item.imageData {
                attributeSet.thumbnailData = imageData
            }
            
            return CSSearchableItem(
                uniqueIdentifier: "menuitem_\(item.id)",
                domainIdentifier: "menuitems",
                attributeSet: attributeSet
            )
        }
        
        try? await CSSearchableIndex.default().indexSearchableItems(searchableItems)
    }
    
    func indexOrders(_ orders: [Order]) async {
        let searchableItems = orders.map { order in
            let attributeSet = CSSearchableItemAttributeSet(itemContentType: UTType.data.identifier)
            
            attributeSet.title = "주문 #\(order.orderNumber)"
            attributeSet.contentDescription = "\(order.restaurant.name) • \(order.status.displayName)"
            attributeSet.contentCreationDate = order.createdAt
            attributeSet.keywords = [
                order.restaurant.name,
                order.orderNumber,
                "주문", "배달"
            ] + order.items.map(\.name)
            
            // 주문 상태에 따른 관련성
            switch order.status {
            case .onTheWay, .preparing:
                attributeSet.rankingHint = NSNumber(value: 1.0) // 높은 우선순위
            case .delivered:
                let daysSinceDelivery = Calendar.current.dateComponents([.day], from: order.deliveredAt ?? Date(), to: Date()).day ?? 0
                attributeSet.rankingHint = NSNumber(value: max(0.1, 1.0 - Double(daysSinceDelivery) / 30.0))
            default:
                attributeSet.rankingHint = NSNumber(value: 0.5)
            }
            
            return CSSearchableItem(
                uniqueIdentifier: "order_\(order.id)",
                domainIdentifier: "orders",
                attributeSet: attributeSet
            )
        }
        
        try? await CSSearchableIndex.default().indexSearchableItems(searchableItems)
    }
    
    func deleteAllItems() async {
        try? await CSSearchableIndex.default().deleteAllSearchableItems()
    }
    
    func deleteItems(withDomainIdentifier domainIdentifier: String) async {
        try? await CSSearchableIndex.default().deleteSearchableItems(withDomainIdentifiers: [domainIdentifier])
    }
}

// MARK: - 앱에서 Spotlight 결과 처리
extension SceneDelegate {
    func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
        if userActivity.activityType == CSSearchableItemActionType {
            handleSpotlightResult(userActivity)
        }
    }
    
    private func handleSpotlightResult(_ userActivity: NSUserActivity) {
        guard let uniqueIdentifier = userActivity.userInfo?[CSSearchableItemActivityIdentifier] as? String else {
            return
        }
        
        let components = uniqueIdentifier.split(separator: "_")
        guard components.count == 2 else { return }
        
        let itemType = String(components[0])
        let itemId = String(components[1])
        
        switch itemType {
        case "restaurant":
            if let uuid = UUID(uuidString: itemId) {
                AppCoordinator.shared.showRestaurant(id: uuid)
            }
        case "menuitem":
            if let uuid = UUID(uuidString: itemId) {
                AppCoordinator.shared.showMenuItem(id: uuid)
            }
        case "order":
            if let uuid = UUID(uuidString: itemId) {
                AppCoordinator.shared.showOrderTracking(id: uuid)
            }
        default:
            break
        }
    }
}
```

## 정리: 생태계 통합의 핵심 원칙

1. **사용자 의도 우선**: 앱을 열지 않고도 목적을 달성할 수 있게
2. **문맥 인식**: 시간, 위치, 상황에 맞는 정보 제공  
3. **점진적 공개**: 필요한 만큼의 정보를 적절한 형태로
4. **일관된 경험**: 어떤 진입점이든 동일한 품질의 서비스
5. **시스템과의 조화**: iOS의 디자인 언어와 자연스럽게 통합

앱이 시스템의 일부가 될 때, 진정한 사용자 경험이 시작됩니다.

→ [L6: 선언적 UI 철학 - SwiftUI의 본질](../L6_philosophy/00_declarative_philosophy.md)

---

*"The best technology is the one that becomes invisible, seamlessly integrated into the fabric of everyday life." - Mark Weiser*