# L5: 시스템 통합 - Widget, App Intents, Live Activities

## iOS 생태계: 앱을 넘어선 존재

```
앱은 더 이상 고립된 섬이 아니다.
시스템의 일부가 되어야 한다.
Control Center, visionOS, 모든 곳에 존재해야 한다.
```

## Widget: 홈 화면의 얼굴

### Timeline Provider 아키텍처

```swift
// Widget/FoodDeliveryWidget.swift
import WidgetKit
import SwiftUI

struct FoodDeliveryWidget: Widget {
    let kind: String = "FoodDeliveryWidget"
    
    var body: some WidgetConfiguration {
        AppIntentConfiguration(
            kind: kind,
            intent: ConfigurationAppIntent.self,
            provider: Provider()
        ) { entry in
            FoodDeliveryWidgetView(entry: entry)
        }
        .configurationDisplayName("음식 배달")
        .description("주문 현황과 추천 메뉴")
        .supportedFamilies([
            .systemSmall, .systemMedium, .systemLarge,
            .accessoryCircular, .accessoryRectangular, .accessoryInline
        ])
        .contentMarginsDisabled()  // iOS 17+
    }
}

// Timeline Provider
struct Provider: AppIntentTimelineProvider {
    func placeholder(in context: Context) -> FoodDeliveryEntry {
        FoodDeliveryEntry(
            date: Date(),
            configuration: ConfigurationAppIntent(),
            orderStatus: nil,
            recommendation: MenuItem.placeholder
        )
    }
    
    func snapshot(for configuration: ConfigurationAppIntent, in context: Context) async -> FoodDeliveryEntry {
        // 빠른 스냅샷 생성 (위젯 갤러리용)
        if context.isPreview {
            return FoodDeliveryEntry(
                date: Date(),
                configuration: configuration,
                orderStatus: OrderStatus.mock,
                recommendation: MenuItem.mock
            )
        }
        
        // 실제 데이터 로드
        return await loadCurrentEntry(for: configuration)
    }
    
    func timeline(for configuration: ConfigurationAppIntent, in context: Context) async -> Timeline<FoodDeliveryEntry> {
        var entries: [FoodDeliveryEntry] = []
        let currentDate = Date()
        
        // 현재 주문 상태 확인
        let orderStatus = await loadCurrentOrder()
        
        if let order = orderStatus {
            // 주문이 있으면 자주 업데이트
            for minuteOffset in 0..<60 {
                let entryDate = Calendar.current.date(
                    byAdding: .minute,
                    value: minuteOffset,
                    to: currentDate
                )!
                
                let entry = FoodDeliveryEntry(
                    date: entryDate,
                    configuration: configuration,
                    orderStatus: estimateStatus(order, at: entryDate),
                    recommendation: nil
                )
                entries.append(entry)
            }
            
            // 1시간 후 다시 업데이트
            let nextUpdate = Calendar.current.date(byAdding: .hour, value: 1, to: currentDate)!
            return Timeline(entries: entries, policy: .after(nextUpdate))
        } else {
            // 주문이 없으면 추천 메뉴 표시
            let recommendations = await loadRecommendations()
            
            for (index, recommendation) in recommendations.enumerated() {
                let entryDate = Calendar.current.date(
                    byAdding: .hour,
                    value: index * 3,  // 3시간마다 추천 변경
                    to: currentDate
                )!
                
                let entry = FoodDeliveryEntry(
                    date: entryDate,
                    configuration: configuration,
                    orderStatus: nil,
                    recommendation: recommendation
                )
                entries.append(entry)
            }
            
            // 다음 날 업데이트
            let tomorrow = Calendar.current.date(byAdding: .day, value: 1, to: currentDate)!
            return Timeline(entries: entries, policy: .after(tomorrow))
        }
    }
    
    private func loadCurrentOrder() async -> OrderInfo? {
        // App Groups를 통한 데이터 공유
        guard let sharedDefaults = UserDefaults(suiteName: "group.com.app.fooddelivery") else {
            return nil
        }
        
        guard let data = sharedDefaults.data(forKey: "currentOrder"),
              let order = try? JSONDecoder().decode(OrderInfo.self, from: data) else {
            return nil
        }
        
        return order
    }
    
    private func estimateStatus(_ order: OrderInfo, at date: Date) -> OrderInfo {
        let elapsed = date.timeIntervalSince(order.orderedAt)
        
        var updatedOrder = order
        if elapsed < 600 {  // 10분
            updatedOrder.status = .confirmed
        } else if elapsed < 1200 {  // 20분
            updatedOrder.status = .preparing
        } else if elapsed < 2400 {  // 40분
            updatedOrder.status = .delivering
        } else {
            updatedOrder.status = .delivered
        }
        
        return updatedOrder
    }
}

// Widget Entry
struct FoodDeliveryEntry: TimelineEntry {
    let date: Date
    let configuration: ConfigurationAppIntent
    let orderStatus: OrderInfo?
    let recommendation: MenuItem?
    
    var relevance: TimelineEntryRelevance? {
        if let order = orderStatus {
            // 주문 상태에 따른 관련성 점수
            switch order.status {
            case .delivering:
                return TimelineEntryRelevance(score: 100, duration: 300)  // 5분간 최고 우선순위
            case .preparing:
                return TimelineEntryRelevance(score: 75, duration: 600)
            default:
                return TimelineEntryRelevance(score: 50, duration: 900)
            }
        }
        return nil
    }
}
```

### Widget View 구현

```swift
// Widget View
struct FoodDeliveryWidgetView: View {
    var entry: Provider.Entry
    @Environment(\.widgetFamily) var family
    @Environment(\.showsWidgetContainerBackground) var showsBackground  // iOS 17+
    
    var body: some View {
        switch family {
        case .systemSmall:
            SmallWidgetView(entry: entry)
        case .systemMedium:
            MediumWidgetView(entry: entry)
        case .systemLarge:
            LargeWidgetView(entry: entry)
        case .accessoryCircular:
            CircularWidgetView(entry: entry)
        default:
            EmptyView()
        }
    }
}

struct SmallWidgetView: View {
    let entry: FoodDeliveryEntry
    
    var body: some View {
        if let order = entry.orderStatus {
            // 주문 상태 표시
            VStack(spacing: 8) {
                Image(systemName: order.status.icon)
                    .font(.largeTitle)
                    .foregroundStyle(order.status.color)
                
                Text(order.status.title)
                    .font(.caption.bold())
                
                Text(order.estimatedTime)
                    .font(.caption2)
                    .foregroundStyle(.secondary)
            }
            .containerBackground(for: .widget) {
                Color.systemBackground
            }
            .widgetURL(URL(string: "fooddelivery://order/\(order.id)"))
        } else if let recommendation = entry.recommendation {
            // 추천 메뉴 표시
            VStack(alignment: .leading, spacing: 4) {
                Text("오늘의 추천")
                    .font(.caption)
                    .foregroundStyle(.secondary)
                
                Text(recommendation.emoji)
                    .font(.largeTitle)
                
                Text(recommendation.name)
                    .font(.footnote.bold())
                    .minimumScaleFactor(0.7)
                
                Text(recommendation.formattedPrice)
                    .font(.caption)
                    .foregroundStyle(.blue)
            }
            .frame(maxWidth: .infinity, maxHeight: .infinity, alignment: .topLeading)
            .padding()
            .containerBackground(for: .widget) {
                recommendation.gradientBackground
            }
            .widgetURL(URL(string: "fooddelivery://menu/\(recommendation.id)"))
        }
    }
}

// Interactive Widget (iOS 17+)
struct MediumWidgetView: View {
    let entry: FoodDeliveryEntry
    
    var body: some View {
        HStack(spacing: 12) {
            // 왼쪽: 상태/추천
            VStack(alignment: .leading, spacing: 8) {
                if let order = entry.orderStatus {
                    OrderStatusCard(order: order)
                } else {
                    RecommendationCard(item: entry.recommendation)
                }
            }
            .frame(maxWidth: .infinity)
            
            // 오른쪽: 빠른 액션
            VStack(spacing: 8) {
                Button(intent: QuickOrderIntent(itemId: "favorite")) {
                    Label("즐겨찾기", systemImage: "star.fill")
                        .font(.caption)
                }
                .buttonStyle(.bordered)
                
                Button(intent: ReorderLastIntent()) {
                    Label("재주문", systemImage: "arrow.clockwise")
                        .font(.caption)
                }
                .buttonStyle(.bordered)
            }
        }
        .padding()
        .containerBackground(for: .widget) {
            Color.systemBackground
        }
    }
}
```

## App Intents: Siri와 Shortcuts 통합

```swift
// AppIntents/QuickOrderIntent.swift
import AppIntents

struct QuickOrderIntent: AppIntent {
    static var title: LocalizedStringResource = "빠른 주문"
    static var description = IntentDescription("자주 먹는 메뉴를 빠르게 주문합니다")
    
    @Parameter(title: "메뉴", description: "주문할 메뉴")
    var itemId: String?
    
    @Parameter(title: "수량", description: "주문 수량", default: 1)
    var quantity: Int
    
    @MainActor
    func perform() async throws -> some IntentResult & ReturnsValue<OrderConfirmation> {
        // 1. 메뉴 확인
        guard let itemId = itemId else {
            throw IntentError.missingParameter
        }
        
        // 2. 주문 처리
        let orderService = OrderService.shared
        let order = try await orderService.quickOrder(itemId: itemId, quantity: quantity)
        
        // 3. 결과 반환
        return .result(
            value: OrderConfirmation(
                orderId: order.id,
                estimatedTime: order.estimatedTime,
                totalPrice: order.totalPrice
            ),
            dialog: IntentDialog("주문이 완료되었습니다. 예상 시간은 \(order.estimatedTime)분입니다.")
        )
    }
    
    // 파라미터 자동 완성
    static var parameterSummary: some ParameterSummary {
        Summary("주문할 메뉴: \(\.$itemId), 수량: \(\.$quantity)개")
    }
}

// Entity for Siri
struct MenuItem: AppEntity {
    let id: String
    let name: String
    let emoji: String
    let price: Int
    
    static var typeDisplayRepresentation = TypeDisplayRepresentation(name: "메뉴")
    
    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(
            title: "\(emoji) \(name)",
            subtitle: "\(price)원"
        )
    }
    
    static var defaultQuery = MenuItemQuery()
}

struct MenuItemQuery: EntityQuery {
    func entities(for identifiers: [String]) async throws -> [MenuItem] {
        // ID로 메뉴 조회
        return try await MenuService.shared.getMenuItems(ids: identifiers)
    }
    
    func suggestedEntities() async throws -> [MenuItem] {
        // 자주 주문하는 메뉴 제안
        return try await MenuService.shared.getFavoriteItems()
    }
    
    func entities(matching string: String) async throws -> [MenuItem] {
        // 검색
        return try await MenuService.shared.searchItems(query: string)
    }
}

// Shortcuts 제공
struct FoodDeliveryShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: QuickOrderIntent(),
            phrases: [
                "배달 주문",
                "음식 주문",
                "\(.applicationName)에서 주문"
            ],
            shortTitle: "빠른 주문",
            systemImageName: "bag"
        )
        
        AppShortcut(
            intent: TrackOrderIntent(),
            phrases: [
                "배달 추적",
                "주문 상태",
                "\(.applicationName)에서 배달 확인"
            ],
            shortTitle: "주문 추적",
            systemImageName: "location"
        )
        
        AppShortcut(
            intent: ReorderLastIntent(),
            phrases: [
                "다시 주문",
                "재주문",
                "\(.applicationName)에서 같은 거"
            ],
            shortTitle: "재주문",
            systemImageName: "arrow.clockwise"
        )
    }
}
```

## Live Activities: Dynamic Island 활용

```swift
// LiveActivity/DeliveryActivity.swift
import ActivityKit

struct DeliveryActivityAttributes: ActivityAttributes {
    public struct ContentState: Codable, Hashable {
        var status: OrderStatus
        var driverLocation: Location?
        var estimatedArrival: Date
        var lastUpdate: Date
    }
    
    var orderNumber: String
    var restaurantName: String
    var totalItems: Int
}

// Live Activity View
struct DeliveryActivityView: View {
    let context: ActivityViewContext<DeliveryActivityAttributes>
    
    var body: some View {
        HStack {
            // 왼쪽: 상태 정보
            VStack(alignment: .leading, spacing: 4) {
                HStack(spacing: 4) {
                    Image(systemName: context.state.status.icon)
                        .foregroundStyle(context.state.status.color)
                    Text(context.state.status.title)
                        .font(.caption.bold())
                }
                
                Text(context.attributes.restaurantName)
                    .font(.caption2)
                    .foregroundStyle(.secondary)
                
                Text("\(context.attributes.totalItems)개 항목")
                    .font(.caption2)
                    .foregroundStyle(.secondary)
            }
            
            Spacer()
            
            // 오른쪽: 시간 정보
            VStack(alignment: .trailing, spacing: 4) {
                Text(context.state.estimatedArrival, style: .timer)
                    .font(.headline.monospacedDigit())
                    .foregroundStyle(.blue)
                
                Text("도착 예정")
                    .font(.caption2)
                    .foregroundStyle(.secondary)
            }
        }
        .padding()
    }
}

// Dynamic Island
struct DeliveryDynamicIsland: DynamicIsland {
    var expanded: some View {
        DynamicIslandExpandedView(context: context)
    }
    
    var compactLeading: some View {
        Image(systemName: context.state.status.icon)
            .foregroundStyle(context.state.status.color)
    }
    
    var compactTrailing: some View {
        Text(context.state.estimatedArrival, style: .timer)
            .font(.caption2.monospacedDigit())
    }
    
    var minimal: some View {
        Image(systemName: "bicycle")
            .foregroundStyle(.blue)
    }
}

// Dynamic Island Expanded View
struct DynamicIslandExpandedView: View {
    let context: ActivityViewContext<DeliveryActivityAttributes>
    
    var body: some View {
        VStack(spacing: 12) {
            // 헤더
            HStack {
                VStack(alignment: .leading, spacing: 2) {
                    Text("주문 #\(context.attributes.orderNumber)")
                        .font(.caption.bold())
                    Text(context.attributes.restaurantName)
                        .font(.caption2)
                        .foregroundStyle(.secondary)
                }
                
                Spacer()
                
                Image(systemName: context.state.status.icon)
                    .font(.title2)
                    .foregroundStyle(context.state.status.color)
            }
            
            // 진행 상황 바
            ProgressView(value: context.state.status.progress)
                .tint(.blue)
            
            // 하단 정보
            HStack {
                // 배달원 위치 (있으면)
                if let location = context.state.driverLocation {
                    HStack(spacing: 4) {
                        Image(systemName: "location.fill")
                            .font(.caption)
                        Text(location.description)
                            .font(.caption)
                    }
                    .foregroundStyle(.secondary)
                }
                
                Spacer()
                
                // 예상 도착 시간
                VStack(alignment: .trailing, spacing: 0) {
                    Text(context.state.estimatedArrival, style: .timer)
                        .font(.headline.monospacedDigit())
                    Text("남음")
                        .font(.caption2)
                        .foregroundStyle(.secondary)
                }
            }
        }
        .padding()
    }
}

// Activity 시작/업데이트
class LiveActivityManager {
    private var currentActivity: Activity<DeliveryActivityAttributes>?
    
    func startTracking(order: Order) {
        guard ActivityAuthorizationInfo().areActivitiesEnabled else {
            return
        }
        
        let attributes = DeliveryActivityAttributes(
            orderNumber: order.number,
            restaurantName: order.restaurant.name,
            totalItems: order.items.count
        )
        
        let initialState = DeliveryActivityAttributes.ContentState(
            status: order.status,
            driverLocation: nil,
            estimatedArrival: order.estimatedArrival,
            lastUpdate: Date()
        )
        
        do {
            currentActivity = try Activity.request(
                attributes: attributes,
                content: .init(state: initialState, staleDate: nil),
                pushType: .token  // Push 업데이트 활성화
            )
            
            // Push token 저장
            Task {
                for await token in currentActivity!.pushTokenUpdates {
                    await ServerAPI.updatePushToken(token)
                }
            }
        } catch {
            print("Failed to start activity: \(error)")
        }
    }
    
    func updateTracking(status: OrderStatus, location: Location?, arrival: Date) {
        guard let activity = currentActivity else { return }
        
        Task {
            let updatedState = DeliveryActivityAttributes.ContentState(
                status: status,
                driverLocation: location,
                estimatedArrival: arrival,
                lastUpdate: Date()
            )
            
            await activity.update(
                ActivityContent(
                    state: updatedState,
                    staleDate: Date().addingTimeInterval(60)  // 1분 후 stale
                )
            )
        }
    }
    
    func endTracking() {
        guard let activity = currentActivity else { return }
        
        Task {
            let finalState = DeliveryActivityAttributes.ContentState(
                status: .delivered,
                driverLocation: nil,
                estimatedArrival: Date(),
                lastUpdate: Date()
            )
            
            await activity.end(
                ActivityContent(
                    state: finalState,
                    staleDate: nil
                ),
                dismissalPolicy: .default  // 잠시 후 자동 제거
            )
            
            currentActivity = nil
        }
    }
}
```

## Control Center Controls (iOS 18+)

```swift
// Controls/QuickOrderControl.swift
import WidgetKit
import AppIntents

struct QuickOrderControl: ControlWidget {
    var body: some ControlWidgetConfiguration {
        AppIntentControlConfiguration(
            kind: "com.app.quickorder",
            provider: Provider()
        ) { value in
            ControlWidgetButton(action: QuickOrderIntent()) {
                Label("빠른 주문", systemImage: "bag.fill")
            }
        }
        .displayName("빠른 주문")
        .description("자주 먹는 메뉴 바로 주문")
    }
}

// Toggle Control
struct DeliveryNotificationControl: ControlWidget {
    var body: some ControlWidgetConfiguration {
        AppIntentControlConfiguration(
            kind: "com.app.notifications",
            provider: NotificationProvider()
        ) { isEnabled in
            ControlWidgetToggle(
                "배달 알림",
                systemImage: isEnabled ? "bell.fill" : "bell.slash",
                isOn: isEnabled,
                action: ToggleNotificationIntent()
            ) { isOn in
                // 상태 변경 시 액션
            }
        }
    }
}
```

## App Clips: 즉석 경험

```swift
// AppClip/AppClipApp.swift
import SwiftUI

@main
struct FoodDeliveryAppClip: App {
    @StateObject private var appClipManager = AppClipManager()
    
    var body: some Scene {
        WindowGroup {
            AppClipContentView()
                .onContinueUserActivity(NSUserActivityTypeBrowsingWeb) { activity in
                    guard let url = activity.webpageURL else { return }
                    appClipManager.handleURL(url)
                }
                .environmentObject(appClipManager)
        }
    }
}

// App Clip Manager
class AppClipManager: ObservableObject {
    @Published var restaurant: Restaurant?
    @Published var tableNumber: String?
    @Published var specialOffer: SpecialOffer?
    
    func handleURL(_ url: URL) {
        // URL 파싱 (예: https://app.com/restaurant/123/table/5)
        let components = URLComponents(url: url, resolvingAgainstBaseURL: true)
        
        if let restaurantId = components?.queryItems?.first(where: { $0.name == "restaurant" })?.value {
            loadRestaurant(id: restaurantId)
        }
        
        if let table = components?.queryItems?.first(where: { $0.name == "table" })?.value {
            tableNumber = table
        }
    }
    
    func completeExperience() {
        // App Clip 경험 완료 후 전체 앱 다운로드 유도
        SKOverlay.AppClipConfiguration(position: .bottom)
            .present(in: UIApplication.shared.windows.first!)
    }
}

// App Clip Location Confirmation
struct LocationConfirmationView: View {
    @StateObject private var locationManager = LocationManager()
    let expectedLocation: CLLocation
    
    var body: some View {
        VStack(spacing: 20) {
            if locationManager.isNearExpectedLocation(expectedLocation) {
                // 위치 확인됨 - 주문 진행
                QuickOrderView()
            } else {
                // 위치 불일치
                Text("매장 근처에서만 주문 가능합니다")
                    .font(.headline)
                
                Text("매장에 도착 후 다시 시도해주세요")
                    .foregroundStyle(.secondary)
            }
        }
        .task {
            await locationManager.requestLocation()
        }
    }
}
```

## 시스템 통합 최적화

### 1. App Groups로 데이터 공유

```swift
// Shared/DataContainer.swift
class SharedDataContainer {
    static let suiteName = "group.com.app.fooddelivery"
    
    private var userDefaults: UserDefaults? {
        UserDefaults(suiteName: Self.suiteName)
    }
    
    private var sharedDirectory: URL? {
        FileManager.default.containerURL(
            forSecurityApplicationGroupIdentifier: Self.suiteName
        )
    }
    
    func saveCurrentOrder(_ order: Order) {
        guard let data = try? JSONEncoder().encode(order) else { return }
        userDefaults?.set(data, forKey: "currentOrder")
        
        // Widget 업데이트 트리거
        WidgetCenter.shared.reloadAllTimelines()
    }
    
    func getCurrentOrder() -> Order? {
        guard let data = userDefaults?.data(forKey: "currentOrder"),
              let order = try? JSONDecoder().decode(Order.self, from: data) else {
            return nil
        }
        return order
    }
}
```

### 2. Background Tasks

```swift
// Background/BackgroundTaskManager.swift
import BackgroundTasks

class BackgroundTaskManager {
    static let orderUpdateTaskIdentifier = "com.app.orderupdate"
    
    func scheduleOrderUpdate(for order: Order) {
        let request = BGAppRefreshTaskRequest(identifier: Self.orderUpdateTaskIdentifier)
        request.earliestBeginDate = Date(timeIntervalSinceNow: 60)  // 1분 후
        
        do {
            try BGTaskScheduler.shared.submit(request)
        } catch {
            print("Failed to schedule background task: \(error)")
        }
    }
    
    func handleOrderUpdate(task: BGAppRefreshTask) {
        task.expirationHandler = {
            // 시간 초과 시 정리
            task.setTaskCompleted(success: false)
        }
        
        Task {
            do {
                let order = try await OrderAPI.checkStatus()
                
                // Live Activity 업데이트
                LiveActivityManager.shared.updateTracking(
                    status: order.status,
                    location: order.driverLocation,
                    arrival: order.estimatedArrival
                )
                
                // Widget 업데이트
                SharedDataContainer().saveCurrentOrder(order)
                
                task.setTaskCompleted(success: true)
                
                // 다음 업데이트 스케줄
                if order.status != .delivered {
                    scheduleOrderUpdate(for: order)
                }
            } catch {
                task.setTaskCompleted(success: false)
            }
        }
    }
}
```

### 3. Push Notification Extensions

```swift
// NotificationService/NotificationService.swift
import UserNotifications

class NotificationService: UNNotificationServiceExtension {
    var contentHandler: ((UNNotificationContent) -> Void)?
    var bestAttemptContent: UNMutableNotificationContent?
    
    override func didReceive(
        _ request: UNNotificationRequest,
        withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void
    ) {
        self.contentHandler = contentHandler
        bestAttemptContent = (request.content.mutableCopy() as? UNMutableNotificationContent)
        
        guard let bestAttemptContent = bestAttemptContent else {
            return
        }
        
        // Live Activity 업데이트
        if let orderUpdate = extractOrderUpdate(from: request.content.userInfo) {
            Task {
                await LiveActivityManager.shared.updateFromPush(orderUpdate)
            }
        }
        
        // 이미지 다운로드 및 첨부
        if let imageURL = request.content.userInfo["image_url"] as? String {
            downloadAndAttachImage(imageURL) { attachment in
                if let attachment = attachment {
                    bestAttemptContent.attachments = [attachment]
                }
                contentHandler(bestAttemptContent)
            }
        } else {
            contentHandler(bestAttemptContent)
        }
    }
}
```

## visionOS 지원 (Apple Vision Pro)

### Spatial Computing으로 확장

```swift
// visionOS/SpatialDeliveryApp.swift
import SwiftUI
import RealityKit

#if os(visionOS)
struct SpatialDeliveryView: View {
    @State private var showImmersiveSpace = false
    @State private var immersiveSpaceIsShown = false
    
    @Environment(\.openImmersiveSpace) var openImmersiveSpace
    @Environment(\.dismissImmersiveSpace) var dismissImmersiveSpace
    
    var body: some View {
        NavigationSplitView {
            // 사이드바: 카테고리
            CategoryListView()
        } content: {
            // 메인: 메뉴 그리드
            MenuGridView()
                .navigationTitle("메뉴")
        } detail: {
            // 디테일: 3D 음식 미리보기
            if let selectedItem = selectedMenuItem {
                Food3DPreview(item: selectedItem)
                    .toolbar {
                        ToolbarItem {
                            Toggle("3D 공간", isOn: $showImmersiveSpace)
                                .onChange(of: showImmersiveSpace) { _, newValue in
                                    Task {
                                        if newValue {
                                            await openImmersiveSpace(id: "FoodSpace")
                                        } else {
                                            await dismissImmersiveSpace()
                                        }
                                    }
                                }
                        }
                    }
            }
        }
    }
}

// 3D 음식 미리보기
struct Food3DPreview: View {
    let item: MenuItem
    @State private var rotation = Angle3D.zero
    
    var body: some View {
        RealityView { content, attachments in
            // 3D 모델 로드
            if let foodEntity = try? await ModelEntity(named: item.modelName) {
                foodEntity.scale = [0.5, 0.5, 0.5]
                foodEntity.position = [0, 0, -0.5]
                
                // 조명 추가
                let light = PointLight()
                light.light.intensity = 1000
                light.position = [0, 0.5, 0.5]
                
                content.add(foodEntity)
                content.add(light)
                
                // 정보 패널 첨부
                if let infoPanel = attachments.entity(for: "info") {
                    infoPanel.position = [0.3, 0, 0]
                    foodEntity.addChild(infoPanel)
                }
            }
        } attachments: {
            Attachment(id: "info") {
                FoodInfoPanel(item: item)
                    .frame(width: 200, height: 300)
                    .glassBackgroundEffect()
            }
        }
        .gesture(
            RotateGesture3D()
                .onChanged { value in
                    rotation = value.rotation
                }
        )
    }
}

// Immersive Space
struct FoodImmersiveSpace: ImmersiveSpace {
    var body: some ImmersiveSpaceContent {
        RealityView { content in
            // 레스토랑 환경 생성
            let restaurantEnvironment = try? await Entity(named: "Restaurant")
            content.add(restaurantEnvironment)
            
            // 테이블 위에 음식 배치
            for (index, item) in orderedItems.enumerated() {
                if let food = try? await ModelEntity(named: item.modelName) {
                    let angle = Float(index) * .pi * 2 / Float(orderedItems.count)
                    food.position = [
                        sin(angle) * 0.3,
                        0.8, // 테이블 높이
                        cos(angle) * 0.3
                    ]
                    content.add(food)
                }
            }
        }
    }
}

// Window Group 설정
@main
struct DeliveryVisionApp: App {
    var body: some Scene {
        WindowGroup {
            SpatialDeliveryView()
        }
        .windowStyle(.volumetric)
        .defaultSize(width: 1, height: 1, depth: 1, in: .meters)
        
        ImmersiveSpace(id: "FoodSpace") {
            FoodImmersiveSpace()
        }
        .immersionStyle(selection: .constant(.mixed), in: .mixed, .progressive, .full)
    }
}
#endif

// 공유 코드: iOS와 visionOS 모두 지원
struct AdaptiveMenuView: View {
    @Environment(\.horizontalSizeClass) var sizeClass
    
    var body: some View {
        #if os(visionOS)
        // Vision Pro: 3D 그리드
        LazyVGrid(columns: [
            GridItem(.adaptive(minimum: 200, maximum: 300))
        ], spacing: 30) {
            ForEach(menuItems) { item in
                MenuItem3D(item: item)
                    .frame(depth: 50)
            }
        }
        #else
        // iOS/iPadOS: 일반 그리드
        LazyVGrid(columns: [
            GridItem(.adaptive(minimum: 150, maximum: 200))
        ], spacing: 20) {
            ForEach(menuItems) { item in
                MenuItemCard(item: item)
            }
        }
        #endif
    }
}
```

### Hand Tracking과 제스처

```swift
#if os(visionOS)
struct HandGestureOrderView: View {
    @State private var selectedItems: Set<MenuItem> = []
    @State private var handAnchor: HandAnchor?
    
    var body: some View {
        RealityView { content in
            // 손 추적 시작
            let handTracking = HandTrackingProvider()
            
            for await hand in handTracking.updates {
                if let indexTip = hand.joint(.indexFingerTip) {
                    // 검지 끝 위치로 포인팅
                    checkMenuItemCollision(at: indexTip.position)
                }
                
                // 엄지-검지 핀치로 선택
                if hand.isPinching {
                    confirmSelection()
                }
            }
        }
        .gesture(
            SpatialTapGesture()
                .onEnded { location in
                    selectMenuItem(at: location)
                }
        )
    }
}
#endif
```

## 정리: 시스템 통합의 핵심

### 1. Widget
- **Timeline 기반 업데이트**: 효율적인 배터리 사용
- **Interactive Widget**: 앱 열지 않고 액션 수행
- **Smart Stack 최적화**: Relevance score 활용
- **Control Center Widget**: iOS 18+ 빠른 접근

### 2. App Intents
- **Siri 통합**: 자연어 명령 지원
- **Shortcuts 자동화**: 사용자 루틴에 통합
- **Focus Filter**: 집중 모드와 연동

### 3. Live Activities
- **실시간 업데이트**: Push로 즉시 반영
- **Dynamic Island**: 시스템 UI의 일부로
- **효율적 관리**: 적절한 시작/종료

### 4. App Clips
- **즉석 경험**: 설치 없이 기능 제공
- **위치 기반**: 물리적 공간과 연결
- **전환 유도**: 전체 앱으로 자연스럽게

### 5. visionOS (Spatial Computing)
- **3D 인터페이스**: 공간에서 음식 미리보기
- **Immersive Space**: 레스토랑 환경 체험
- **Hand Tracking**: 자연스러운 상호작용
- **Shared Code**: iOS와 코드 공유

## First Principles: 왜 시스템 통합인가?

1. **접근성**: 사용자가 있는 곳으로 찾아가기
2. **편의성**: 최소한의 마찰로 목적 달성
3. **지속성**: 앱을 열지 않아도 존재감 유지
4. **자연스러움**: iOS 경험의 일부가 되기
5. **효율성**: 시스템 리소스 최적 활용

## 다음 레벨로

이제 철학적 깊이로 들어갈 시간이다.

→ [L6: 선언적 UI의 철학 - SwiftUI가 바꾼 사고방식](../L6_philosophy/00_declarative_philosophy.md)

---

*"The best app is the one you don't have to open." - iOS Design Philosophy*