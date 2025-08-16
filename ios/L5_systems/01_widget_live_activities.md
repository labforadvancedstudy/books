# Widget & Live Activity 실전 구현

## 핵심: 한 눈에 보는 가치 전달

Widget과 Live Activity는 **앱을 열지 않고도 핵심 정보를 전달**하는 것이 목표입니다.

## 1. Widget 기본 구조

### Timeline Provider 구현

```swift
import WidgetKit
import SwiftUI

// MARK: - Widget Entry
struct DeliveryEntry: TimelineEntry {
    let date: Date
    let configuration: ConfigurationIntent
    let order: Order?
    let deliveryTime: Date?
    let driverLocation: CLLocationCoordinate2D?
}

// MARK: - Timeline Provider
struct DeliveryProvider: IntentTimelineProvider {
    // App Group을 통한 데이터 공유
    private let sharedDefaults = UserDefaults(
        suiteName: "group.com.example.delivery"
    )!
    
    private let locationManager = SharedLocationManager()
    
    // 플레이스홀더: 로딩 중 표시
    func placeholder(in context: Context) -> DeliveryEntry {
        DeliveryEntry(
            date: Date(),
            configuration: ConfigurationIntent(),
            order: Order.placeholder,
            deliveryTime: Date().addingTimeInterval(1800),
            driverLocation: nil
        )
    }
    
    // 스냅샷: Widget Gallery에서 표시
    func getSnapshot(
        for configuration: ConfigurationIntent,
        in context: Context,
        completion: @escaping (DeliveryEntry) -> Void
    ) {
        // 빠른 응답이 중요 (3초 이내)
        if context.isPreview {
            completion(placeholder(in: context))
        } else {
            let entry = loadCurrentOrder(for: configuration)
            completion(entry)
        }
    }
    
    // Timeline 생성: 실제 데이터 로드
    func getTimeline(
        for configuration: ConfigurationIntent,
        in context: Context,
        completion: @escaping (Timeline<DeliveryEntry>) -> Void
    ) {
        Task {
            do {
                let entries = try await generateTimeline(
                    for: configuration,
                    context: context
                )
                
                // 다음 업데이트 시간 계산
                let nextUpdate = calculateNextUpdate(entries: entries)
                
                let timeline = Timeline(
                    entries: entries,
                    policy: .after(nextUpdate)
                )
                
                completion(timeline)
            } catch {
                // 에러 시 5분 후 재시도
                let errorEntry = DeliveryEntry(
                    date: Date(),
                    configuration: configuration,
                    order: nil,
                    deliveryTime: nil,
                    driverLocation: nil
                )
                
                let timeline = Timeline(
                    entries: [errorEntry],
                    policy: .after(Date().addingTimeInterval(300))
                )
                
                completion(timeline)
            }
        }
    }
    
    // Timeline 생성 로직
    private func generateTimeline(
        for configuration: ConfigurationIntent,
        context: Context
    ) async throws -> [DeliveryEntry] {
        // 1. 현재 주문 정보 로드
        guard let orderData = sharedDefaults.data(forKey: "currentOrder"),
              let order = try? JSONDecoder().decode(Order.self, from: orderData) else {
            return [createEmptyEntry(for: configuration)]
        }
        
        // 2. 배달 상태에 따른 업데이트 주기 결정
        let updateInterval: TimeInterval = switch order.status {
        case .preparing:
            300 // 5분: 준비 중
        case .onTheWay:
            60  // 1분: 배달 중 (위치 업데이트)
        case .nearbyHome:
            30  // 30초: 도착 임박
        default:
            900 // 15분: 기본
        }
        
        // 3. 향후 1시간 Timeline 생성
        var entries: [DeliveryEntry] = []
        let currentDate = Date()
        
        for offset in 0..<min(12, Int(3600 / updateInterval)) {
            let entryDate = currentDate.addingTimeInterval(
                Double(offset) * updateInterval
            )
            
            // 예상 도착 시간 계산
            let estimatedTime = calculateEstimatedTime(
                order: order,
                at: entryDate
            )
            
            // 드라이버 위치 (캐시된 데이터 사용)
            let driverLocation = loadDriverLocation()
            
            let entry = DeliveryEntry(
                date: entryDate,
                configuration: configuration,
                order: order,
                deliveryTime: estimatedTime,
                driverLocation: driverLocation
            )
            
            entries.append(entry)
        }
        
        return entries
    }
    
    // 배터리 최적화: 다음 업데이트 시간 계산
    private func calculateNextUpdate(entries: [DeliveryEntry]) -> Date {
        guard let lastEntry = entries.last else {
            return Date().addingTimeInterval(900) // 15분 후
        }
        
        // 배달 완료 예상 시간 이후에는 업데이트 불필요
        if let deliveryTime = lastEntry.deliveryTime,
           deliveryTime < Date() {
            return Date().addingTimeInterval(3600) // 1시간 후
        }
        
        // 일반적인 다음 업데이트
        return lastEntry.date.addingTimeInterval(60)
    }
}
```

### Widget View 구현

```swift
struct DeliveryWidgetView: View {
    let entry: DeliveryEntry
    @Environment(\.widgetFamily) var family
    
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
        case .accessoryRectangular:
            RectangularWidgetView(entry: entry)
        case .accessoryInline:
            InlineWidgetView(entry: entry)
        default:
            EmptyView()
        }
    }
}

// Small Widget: 핵심 정보만
struct SmallWidgetView: View {
    let entry: DeliveryEntry
    
    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            // 상태 표시
            HStack {
                Image(systemName: statusIcon)
                    .foregroundColor(statusColor)
                Text(statusText)
                    .font(.caption)
                    .fontWeight(.semibold)
            }
            
            Spacer()
            
            // 예상 시간
            if let deliveryTime = entry.deliveryTime {
                VStack(alignment: .leading, spacing: 2) {
                    Text("도착 예정")
                        .font(.caption2)
                        .foregroundColor(.secondary)
                    
                    Text(deliveryTime, style: .time)
                        .font(.title2)
                        .fontWeight(.bold)
                    
                    // 남은 시간
                    Text(deliveryTime, style: .relative)
                        .font(.caption)
                        .foregroundColor(.orange)
                }
            }
        }
        .padding()
        .widgetURL(URL(string: "delivery://order/\(entry.order?.id ?? "")")!)
    }
    
    private var statusIcon: String {
        switch entry.order?.status {
        case .preparing: return "clock"
        case .onTheWay: return "bicycle"
        case .nearbyHome: return "location.fill"
        default: return "questionmark"
        }
    }
    
    private var statusColor: Color {
        switch entry.order?.status {
        case .preparing: return .blue
        case .onTheWay: return .orange
        case .nearbyHome: return .green
        default: return .gray
        }
    }
    
    private var statusText: String {
        switch entry.order?.status {
        case .preparing: return "준비 중"
        case .onTheWay: return "배달 중"
        case .nearbyHome: return "곧 도착"
        default: return "주문 없음"
        }
    }
}

// Medium Widget: 지도 포함
struct MediumWidgetView: View {
    let entry: DeliveryEntry
    
    var body: some View {
        HStack(spacing: 12) {
            // 왼쪽: 정보
            VStack(alignment: .leading, spacing: 8) {
                SmallWidgetView(entry: entry)
                
                if let items = entry.order?.items {
                    VStack(alignment: .leading, spacing: 2) {
                        ForEach(items.prefix(2)) { item in
                            Text("• \(item.name)")
                                .font(.caption2)
                                .lineLimit(1)
                        }
                        
                        if items.count > 2 {
                            Text("외 \(items.count - 2)개")
                                .font(.caption2)
                                .foregroundColor(.secondary)
                        }
                    }
                }
            }
            
            // 오른쪽: 지도
            if let location = entry.driverLocation {
                MapSnapshotView(
                    driverLocation: location,
                    homeLocation: entry.order?.deliveryAddress.coordinate
                )
                .frame(width: 120, height: 120)
                .cornerRadius(12)
            }
        }
        .padding()
    }
}
```

### Interactive Widget (iOS 17+)

```swift
struct DeliveryWidget: Widget {
    var body: some WidgetConfiguration {
        AppIntentConfiguration(
            kind: "DeliveryWidget",
            provider: DeliveryProvider()
        ) { entry in
            DeliveryWidgetView(entry: entry)
        }
        .configurationDisplayName("배달 추적")
        .description("현재 주문 상태를 확인하세요")
        .supportedFamilies([
            .systemSmall,
            .systemMedium,
            .systemLarge,
            .accessoryCircular,
            .accessoryRectangular,
            .accessoryInline
        ])
        // iOS 17: Interactive
        .supplementalActivityFamilies([.small, .medium])
    }
}

// App Intent for Interaction
struct RefreshOrderIntent: AppIntent {
    static var title: LocalizedStringResource = "주문 새로고침"
    
    func perform() async throws -> some IntentResult {
        // 주문 정보 업데이트
        let manager = OrderManager.shared
        try await manager.refreshCurrentOrder()
        
        // Widget 업데이트 요청
        WidgetCenter.shared.reloadTimelines(ofKind: "DeliveryWidget")
        
        return .result()
    }
}

// Interactive Button
struct RefreshButton: View {
    var body: some View {
        Button(intent: RefreshOrderIntent()) {
            Label("새로고침", systemImage: "arrow.clockwise")
        }
        .buttonStyle(.borderedProminent)
    }
}
```

## 2. Live Activity 구현

### Activity Configuration

```swift
import ActivityKit

// MARK: - Activity Attributes
struct DeliveryAttributes: ActivityAttributes {
    public struct ContentState: Codable, Hashable {
        // 동적으로 변경되는 데이터
        let status: OrderStatus
        let driverName: String
        let driverPhone: String
        let estimatedTime: Date
        let currentLocation: Location?
        let completionCode: String?
    }
    
    // 정적 데이터 (Activity 생성 시 고정)
    let orderID: String
    let restaurantName: String
    let totalAmount: Int
    let itemCount: Int
}

// MARK: - Live Activity Manager
@MainActor
class LiveActivityManager: ObservableObject {
    @Published private(set) var currentActivity: Activity<DeliveryAttributes>?
    
    private let pushTokenManager = PushTokenManager()
    
    // Live Activity 시작
    func startDeliveryTracking(for order: Order) async throws {
        // 기존 Activity 종료
        await endCurrentActivity()
        
        // ActivityKit 권한 확인
        guard ActivityAuthorizationInfo().areActivitiesEnabled else {
            throw LiveActivityError.disabled
        }
        
        // Attributes 생성
        let attributes = DeliveryAttributes(
            orderID: order.id,
            restaurantName: order.restaurant.name,
            totalAmount: order.totalAmount,
            itemCount: order.items.count
        )
        
        // 초기 상태
        let initialState = DeliveryAttributes.ContentState(
            status: order.status,
            driverName: order.driver?.name ?? "배정 중",
            driverPhone: order.driver?.phone ?? "",
            estimatedTime: order.estimatedDeliveryTime,
            currentLocation: nil,
            completionCode: nil
        )
        
        // Activity 생성
        let activity = try Activity.request(
            attributes: attributes,
            content: .init(state: initialState, staleDate: nil),
            pushType: .token // Push 업데이트 활성화
        )
        
        self.currentActivity = activity
        
        // Push Token 등록
        if let token = activity.pushToken {
            await registerPushToken(token, for: order.id)
        }
        
        // Token 업데이트 관찰
        observePushTokenUpdates()
    }
    
    // 상태 업데이트
    func updateDeliveryStatus(_ update: OrderUpdate) async {
        guard let activity = currentActivity else { return }
        
        let updatedState = DeliveryAttributes.ContentState(
            status: update.status,
            driverName: update.driverName ?? activity.content.state.driverName,
            driverPhone: update.driverPhone ?? activity.content.state.driverPhone,
            estimatedTime: update.estimatedTime ?? activity.content.state.estimatedTime,
            currentLocation: update.location,
            completionCode: update.completionCode
        )
        
        // Alert 설정 (중요 업데이트 시)
        let alertConfig: AlertConfiguration? = switch update.status {
        case .nearbyHome:
            AlertConfiguration(
                title: "배달원 도착 임박",
                body: "곧 도착합니다. 준비해주세요!",
                sound: .default
            )
        case .delivered:
            AlertConfiguration(
                title: "배달 완료",
                body: "맛있게 드세요!",
                sound: .default
            )
        default:
            nil
        }
        
        await activity.update(
            ActivityContent(
                state: updatedState,
                staleDate: Date().addingTimeInterval(300) // 5분 후 stale
            ),
            alertConfiguration: alertConfig
        )
    }
    
    // Activity 종료
    func endCurrentActivity() async {
        guard let activity = currentActivity else { return }
        
        // 최종 상태로 업데이트
        let finalState = DeliveryAttributes.ContentState(
            status: .delivered,
            driverName: activity.content.state.driverName,
            driverPhone: "",
            estimatedTime: Date(),
            currentLocation: nil,
            completionCode: nil
        )
        
        await activity.end(
            ActivityContent(
                state: finalState,
                staleDate: nil
            ),
            dismissalPolicy: .after(Date().addingTimeInterval(60)) // 1분 후 자동 제거
        )
        
        self.currentActivity = nil
    }
    
    // Push Token 관리
    private func registerPushToken(_ token: Data, for orderID: String) async {
        let tokenString = token.map { String(format: "%02.2hhx", $0) }.joined()
        
        do {
            try await APIClient.shared.registerActivityToken(
                token: tokenString,
                orderID: orderID,
                deviceID: UIDevice.current.identifierForVendor?.uuidString ?? ""
            )
        } catch {
            print("Failed to register push token: \(error)")
        }
    }
    
    private func observePushTokenUpdates() {
        Task {
            guard let activity = currentActivity else { return }
            
            for await token in activity.pushTokenUpdates {
                await registerPushToken(
                    token,
                    for: activity.attributes.orderID
                )
            }
        }
    }
}
```

### Live Activity View

```swift
// MARK: - Lock Screen View
struct DeliveryLiveActivityView: View {
    let context: ActivityViewContext<DeliveryAttributes>
    
    var body: some View {
        HStack(spacing: 16) {
            // 왼쪽: 상태 아이콘
            ZStack {
                Circle()
                    .fill(statusColor.gradient)
                    .frame(width: 60, height: 60)
                
                Image(systemName: statusIcon)
                    .font(.title)
                    .foregroundColor(.white)
            }
            
            // 중앙: 정보
            VStack(alignment: .leading, spacing: 4) {
                Text(context.attributes.restaurantName)
                    .font(.headline)
                    .lineLimit(1)
                
                Text(statusText)
                    .font(.subheadline)
                    .foregroundColor(statusColor)
                
                if context.state.status == .onTheWay {
                    HStack(spacing: 4) {
                        Image(systemName: "person.fill")
                            .font(.caption)
                        Text(context.state.driverName)
                            .font(.caption)
                            .lineLimit(1)
                    }
                    .foregroundColor(.secondary)
                }
            }
            
            Spacer()
            
            // 오른쪽: 시간/코드
            VStack(alignment: .trailing, spacing: 4) {
                if let code = context.state.completionCode {
                    Text(code)
                        .font(.title2)
                        .fontWeight(.bold)
                        .monospacedDigit()
                } else {
                    Text(context.state.estimatedTime, style: .time)
                        .font(.title3)
                        .fontWeight(.semibold)
                    
                    Text("도착 예정")
                        .font(.caption2)
                        .foregroundColor(.secondary)
                }
            }
        }
        .padding()
        .activitySystemActionForegroundColor(.primary)
        .activityBackgroundTint(Color.orange.opacity(0.1))
    }
    
    private var statusIcon: String {
        switch context.state.status {
        case .confirmed: return "checkmark.circle"
        case .preparing: return "flame"
        case .ready: return "bag"
        case .onTheWay: return "bicycle"
        case .nearbyHome: return "location.fill"
        case .delivered: return "house.fill"
        default: return "questionmark"
        }
    }
    
    private var statusColor: Color {
        switch context.state.status {
        case .confirmed, .preparing: return .blue
        case .ready: return .purple
        case .onTheWay: return .orange
        case .nearbyHome, .delivered: return .green
        default: return .gray
        }
    }
    
    private var statusText: String {
        switch context.state.status {
        case .confirmed: return "주문 확인됨"
        case .preparing: return "조리 중"
        case .ready: return "픽업 대기"
        case .onTheWay: return "배달 중"
        case .nearbyHome: return "곧 도착"
        case .delivered: return "배달 완료"
        default: return "상태 확인 중"
        }
    }
}

// MARK: - Dynamic Island
struct DeliveryDynamicIsland: DynamicIsland {
    var expanded: some View {
        DynamicIslandExpandedContent {
            // Leading: Restaurant logo
            DynamicIslandExpandedRegion(.leading) {
                AsyncImage(url: restaurantLogoURL) { image in
                    image
                        .resizable()
                        .aspectRatio(contentMode: .fit)
                } placeholder: {
                    Image(systemName: "fork.knife")
                }
                .frame(width: 40, height: 40)
            }
            
            // Center: Status
            DynamicIslandExpandedRegion(.center) {
                VStack(spacing: 4) {
                    Text(statusText)
                        .font(.headline)
                    
                    if let location = context.state.currentLocation {
                        Text("\(location.distance)km 남음")
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }
                }
            }
            
            // Trailing: Time
            DynamicIslandExpandedRegion(.trailing) {
                Text(context.state.estimatedTime, style: .timer)
                    .font(.title2)
                    .monospacedDigit()
            }
            
            // Bottom: Map or Actions
            DynamicIslandExpandedRegion(.bottom) {
                if context.state.status == .onTheWay {
                    // Mini map
                    MapView(driverLocation: context.state.currentLocation)
                        .frame(height: 120)
                        .cornerRadius(12)
                } else {
                    // Action buttons
                    HStack {
                        Link(destination: phoneURL) {
                            Label("전화", systemImage: "phone")
                        }
                        .buttonStyle(.borderedProminent)
                        
                        Link(destination: detailURL) {
                            Label("상세", systemImage: "arrow.right")
                        }
                        .buttonStyle(.bordered)
                    }
                }
            }
        }
    }
    
    var compactLeading: some View {
        Image(systemName: statusIcon)
            .foregroundColor(statusColor)
    }
    
    var compactTrailing: some View {
        Text(context.state.estimatedTime, style: .timer)
            .monospacedDigit()
    }
    
    var minimal: some View {
        Image(systemName: statusIcon)
            .foregroundColor(statusColor)
    }
}
```

## 3. App Groups 데이터 공유

### Shared Data Manager

```swift
// MARK: - App Group Container
class SharedDataContainer {
    static let shared = SharedDataContainer()
    
    private let suiteName = "group.com.example.delivery"
    private let userDefaults: UserDefaults
    private let fileManager = FileManager.default
    
    private var containerURL: URL {
        fileManager.containerURL(
            forSecurityApplicationGroupIdentifier: suiteName
        )!
    }
    
    init() {
        guard let defaults = UserDefaults(suiteName: suiteName) else {
            fatalError("App Group not configured")
        }
        self.userDefaults = defaults
    }
    
    // MARK: - UserDefaults 기반 (작은 데이터)
    
    func saveCurrentOrder(_ order: Order) throws {
        let encoder = JSONEncoder()
        let data = try encoder.encode(order)
        userDefaults.set(data, forKey: "currentOrder")
        
        // Widget 업데이트 트리거
        WidgetCenter.shared.reloadAllTimelines()
    }
    
    func loadCurrentOrder() -> Order? {
        guard let data = userDefaults.data(forKey: "currentOrder") else {
            return nil
        }
        
        return try? JSONDecoder().decode(Order.self, from: data)
    }
    
    // MARK: - File 기반 (큰 데이터)
    
    func saveOrderHistory(_ orders: [Order]) throws {
        let url = containerURL.appendingPathComponent("orderHistory.json")
        let data = try JSONEncoder().encode(orders)
        try data.write(to: url)
    }
    
    func loadOrderHistory() -> [Order] {
        let url = containerURL.appendingPathComponent("orderHistory.json")
        
        guard let data = try? Data(contentsOf: url) else {
            return []
        }
        
        return (try? JSONDecoder().decode([Order].self, from: data)) ?? []
    }
    
    // MARK: - 이미지 캐싱
    
    func saveImage(_ image: UIImage, for key: String) throws {
        let url = containerURL
            .appendingPathComponent("ImageCache")
            .appendingPathComponent("\(key).jpg")
        
        // 디렉토리 생성
        try fileManager.createDirectory(
            at: url.deletingLastPathComponent(),
            withIntermediateDirectories: true
        )
        
        // JPEG 압축 저장 (Widget 메모리 고려)
        guard let data = image.jpegData(compressionQuality: 0.7) else {
            throw CacheError.compressionFailed
        }
        
        try data.write(to: url)
    }
    
    func loadImage(for key: String) -> UIImage? {
        let url = containerURL
            .appendingPathComponent("ImageCache")
            .appendingPathComponent("\(key).jpg")
        
        guard let data = try? Data(contentsOf: url) else {
            return nil
        }
        
        return UIImage(data: data)
    }
    
    // MARK: - 동기화
    
    func synchronize() {
        userDefaults.synchronize()
    }
}
```

## 4. 배터리 최적화 전략

### Timeline 최적화

```swift
class SmartTimelineProvider: TimelineProvider {
    // 지능형 업데이트 스케줄링
    func calculateUpdatePolicy(
        for order: Order,
        batteryLevel: Float,
        isLowPowerMode: Bool
    ) -> TimelineReloadPolicy {
        // 저전력 모드
        if isLowPowerMode {
            return .after(Date().addingTimeInterval(1800)) // 30분
        }
        
        // 배터리 레벨별
        let baseInterval: TimeInterval = switch batteryLevel {
        case 0..<0.2:
            1800 // 20% 미만: 30분
        case 0.2..<0.5:
            600  // 20-50%: 10분
        default:
            300  // 50% 이상: 5분
        }
        
        // 주문 상태별 조정
        let multiplier: Double = switch order.status {
        case .delivered, .cancelled:
            6.0 // 완료: 드물게 업데이트
        case .preparing:
            2.0 // 준비 중: 보통
        case .onTheWay, .nearbyHome:
            0.5 // 배달 중: 자주
        default:
            1.0
        }
        
        let interval = baseInterval * multiplier
        return .after(Date().addingTimeInterval(interval))
    }
    
    // 스마트 엔트리 생성
    func generateSmartEntries(
        for order: Order,
        count: Int = 12
    ) -> [DeliveryEntry] {
        var entries: [DeliveryEntry] = []
        let now = Date()
        
        // 첫 엔트리는 즉시
        entries.append(createEntry(for: order, at: now))
        
        // 이후 엔트리는 점진적으로 간격 증가
        var interval: TimeInterval = 60 // 1분부터 시작
        
        for i in 1..<count {
            let date = now.addingTimeInterval(
                entries.map { $0.date.timeIntervalSince(now) }.reduce(0, +) + interval
            )
            
            entries.append(createEntry(for: order, at: date))
            
            // 간격 점진적 증가 (최대 15분)
            interval = min(interval * 1.2, 900)
        }
        
        return entries
    }
}
```

## 5. Control Center Widget (iOS 18+)

```swift
import WidgetKit
import ControlWidgetKit

@available(iOS 18.0, *)
struct DeliveryControlWidget: ControlWidget {
    var body: some ControlWidgetConfiguration {
        AppIntentControlConfiguration(
            kind: "DeliveryControl",
            provider: Provider()
        ) { value in
            ControlWidgetButton(action: RefreshOrderIntent()) {
                Label("주문 새로고침", systemImage: "arrow.clockwise")
            }
        }
        .displayName("배달 추적")
        .description("현재 주문을 새로고침합니다")
    }
}

// Control Center Toggle
@available(iOS 18.0, *)
struct DeliveryNotificationToggle: ControlWidget {
    var body: some ControlWidgetConfiguration {
        AppIntentControlConfiguration(
            kind: "NotificationToggle",
            provider: ToggleProvider()
        ) { isEnabled in
            ControlWidgetToggle(
                isOn: isEnabled,
                action: ToggleNotificationIntent()
            ) {
                Label(
                    isEnabled ? "알림 켜짐" : "알림 꺼짐",
                    systemImage: isEnabled ? "bell.fill" : "bell.slash"
                )
            }
            .tint(isEnabled ? .orange : .gray)
        }
    }
}
```

## 6. 실전 체크리스트

### Widget 최적화
- [ ] Timeline 엔트리 12개 이하 유지
- [ ] 이미지 압축 (JPEG 0.7 품질)
- [ ] 3초 이내 스냅샷 생성
- [ ] App Group 데이터 동기화
- [ ] 배터리 상태 고려한 업데이트 주기

### Live Activity 체크리스트
- [ ] Push Token 서버 등록
- [ ] 4KB 이하 payload 유지
- [ ] Stale date 적절히 설정
- [ ] 종료 정책 명확히 정의
- [ ] Dynamic Island 모든 상태 지원

### 성능 모니터링
- [ ] Widget 업데이트 빈도 추적
- [ ] 배터리 소모량 측정
- [ ] 메모리 사용량 확인
- [ ] Crash 리포트 분석

## 결론

Widget과 Live Activity는 **사용자의 주의를 최소한으로 요구하면서 최대의 가치를 전달**해야 합니다. 배터리 효율성과 적시성 사이의 균형을 맞추는 것이 핵심입니다.