# L2: 첫 앱 만들기 - "배고파" 앱

## 프로젝트: 가장 단순한 배달 앱

목표: 배고픈 사람이 음식을 주문하는 최소한의 경험

```
시작 → 메뉴 선택 → 주문 → 완료
```

## Step 1: 프로젝트 생성

1. Xcode 열기
2. Create New Project
3. iOS → App 선택
4. 설정:
   - Product Name: `Baegopa` (배고파)
   - Interface: SwiftUI
   - Language: Swift
   - Use Core Data: 체크 해제
   - Include Tests: 체크 (나중을 위해)

## Step 2: 데이터 모델

```swift
// Models/MenuItem.swift
import Foundation

struct MenuItem: Identifiable, Codable {
    let id = UUID()
    let name: String
    let price: Int
    let emoji: String
    let preparationTime: Int  // 분 단위
    let calories: Int
    
    var formattedPrice: String {
        let formatter = NumberFormatter()
        formatter.numberStyle = .currency
        formatter.locale = Locale(identifier: "ko_KR")
        return formatter.string(from: NSNumber(value: price)) ?? ""
    }
}

// Models/Order.swift
struct Order: Identifiable {
    let id = UUID()
    let items: [MenuItem]
    let orderedAt: Date
    var status: OrderStatus = .pending
    
    var totalPrice: Int {
        items.reduce(0) { $0 + $1.price }
    }
    
    var estimatedTime: Int {
        items.map(\.preparationTime).max() ?? 0
    }
}

enum OrderStatus: String, CaseIterable {
    case pending = "주문 확인 중"
    case preparing = "준비 중"
    case delivering = "배달 중"
    case completed = "배달 완료"
    
    var icon: String {
        switch self {
        case .pending: return "clock"
        case .preparing: return "flame"
        case .delivering: return "bicycle"
        case .completed: return "checkmark.circle.fill"
        }
    }
}
```

## Step 3: ViewModel

```swift
// ViewModels/MenuViewModel.swift
import SwiftUI

@Observable
class MenuViewModel {
    var menuItems: [MenuItem] = []
    var cart: [MenuItem] = []
    var currentOrder: Order?
    
    init() {
        loadMenu()
    }
    
    func loadMenu() {
        // 실제 앱에서는 API 호출
        menuItems = [
            MenuItem(name: "김치찌개", price: 8000, emoji: "🍲", 
                    preparationTime: 15, calories: 450),
            MenuItem(name: "불고기", price: 12000, emoji: "🥘", 
                    preparationTime: 20, calories: 550),
            MenuItem(name: "비빔밥", price: 9000, emoji: "🍚", 
                    preparationTime: 10, calories: 500),
            MenuItem(name: "떡볶이", price: 6000, emoji: "🌶️", 
                    preparationTime: 12, calories: 380),
            MenuItem(name: "김밥", price: 3500, emoji: "🍙", 
                    preparationTime: 5, calories: 300),
            MenuItem(name: "라면", price: 4500, emoji: "🍜", 
                    preparationTime: 8, calories: 400)
        ]
    }
    
    func addToCart(_ item: MenuItem) {
        cart.append(item)
        
        // 햅틱 피드백
        let impact = UIImpactFeedbackGenerator(style: .light)
        impact.impactOccurred()
    }
    
    func removeFromCart(_ item: MenuItem) {
        if let index = cart.firstIndex(where: { $0.id == item.id }) {
            cart.remove(at: index)
        }
    }
    
    func placeOrder() {
        guard !cart.isEmpty else { return }
        
        currentOrder = Order(items: cart, orderedAt: Date())
        cart.removeAll()
        
        // 상태 업데이트 시뮬레이션
        simulateOrderProgress()
    }
    
    private func simulateOrderProgress() {
        Task {
            try? await Task.sleep(for: .seconds(2))
            currentOrder?.status = .preparing
            
            try? await Task.sleep(for: .seconds(5))
            currentOrder?.status = .delivering
            
            try? await Task.sleep(for: .seconds(10))
            currentOrder?.status = .completed
        }
    }
}
```

## Step 4: 메인 화면

```swift
// Views/ContentView.swift
import SwiftUI

struct ContentView: View {
    @State private var viewModel = MenuViewModel()
    @State private var selectedTab = 0
    
    var body: some View {
        TabView(selection: $selectedTab) {
            MenuView(viewModel: viewModel)
                .tabItem {
                    Label("메뉴", systemImage: "list.bullet")
                }
                .tag(0)
            
            CartView(viewModel: viewModel)
                .tabItem {
                    Label("장바구니", systemImage: "cart")
                }
                .badge(viewModel.cart.count)
                .tag(1)
            
            OrderStatusView(viewModel: viewModel)
                .tabItem {
                    Label("주문 현황", systemImage: "clock")
                }
                .tag(2)
        }
    }
}
```

## Step 5: 메뉴 화면

```swift
// Views/MenuView.swift
struct MenuView: View {
    @Bindable var viewModel: MenuViewModel
    @State private var searchText = ""
    
    var filteredItems: [MenuItem] {
        if searchText.isEmpty {
            return viewModel.menuItems
        } else {
            return viewModel.menuItems.filter { 
                $0.name.contains(searchText) 
            }
        }
    }
    
    var body: some View {
        NavigationStack {
            ScrollView {
                LazyVGrid(columns: [
                    GridItem(.flexible()),
                    GridItem(.flexible())
                ], spacing: 16) {
                    ForEach(filteredItems) { item in
                        MenuItemCard(item: item) {
                            viewModel.addToCart(item)
                        }
                    }
                }
                .padding()
            }
            .navigationTitle("오늘 뭐 먹지?")
            .searchable(text: $searchText, prompt: "메뉴 검색")
            .overlay(alignment: .bottom) {
                if !viewModel.cart.isEmpty {
                    CartBadge(count: viewModel.cart.count)
                }
            }
        }
    }
}

struct MenuItemCard: View {
    let item: MenuItem
    let action: () -> Void
    
    @State private var isPressed = false
    
    var body: some View {
        VStack(spacing: 8) {
            Text(item.emoji)
                .font(.system(size: 50))
                .scaleEffect(isPressed ? 0.9 : 1.0)
            
            Text(item.name)
                .font(.headline)
            
            Text(item.formattedPrice)
                .font(.subheadline)
                .foregroundColor(.secondary)
            
            HStack(spacing: 12) {
                Label("\(item.preparationTime)분", 
                      systemImage: "clock")
                Label("\(item.calories)kcal", 
                      systemImage: "flame")
            }
            .font(.caption)
            .foregroundColor(.secondary)
            
            Button(action: action) {
                Text("담기")
                    .font(.footnote.bold())
                    .foregroundColor(.white)
                    .frame(maxWidth: .infinity)
                    .padding(.vertical, 8)
                    .background(Color.blue)
                    .cornerRadius(8)
            }
            .buttonStyle(.plain)
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(radius: 2)
        .onTapGesture {
            withAnimation(.spring(response: 0.3, dampingFraction: 0.6)) {
                isPressed = true
            }
            
            DispatchQueue.main.asyncAfter(deadline: .now() + 0.1) {
                isPressed = false
                action()
            }
        }
    }
}

struct CartBadge: View {
    let count: Int
    
    var body: some View {
        HStack {
            Image(systemName: "cart.fill")
            Text("\(count)개 담김")
                .font(.footnote.bold())
        }
        .foregroundColor(.white)
        .padding(.horizontal, 16)
        .padding(.vertical, 10)
        .background(Color.blue)
        .cornerRadius(25)
        .shadow(radius: 4)
        .padding()
        .transition(.move(edge: .bottom).combined(with: .opacity))
    }
}
```

## Step 6: 장바구니 화면

```swift
// Views/CartView.swift
struct CartView: View {
    @Bindable var viewModel: MenuViewModel
    @State private var showingOrderConfirmation = false
    
    var body: some View {
        NavigationStack {
            if viewModel.cart.isEmpty {
                EmptyCartView()
            } else {
                List {
                    Section {
                        ForEach(viewModel.cart) { item in
                            CartItemRow(item: item)
                        }
                        .onDelete { indexSet in
                            for index in indexSet {
                                viewModel.cart.remove(at: index)
                            }
                        }
                    }
                    
                    Section {
                        HStack {
                            Text("총 금액")
                                .font(.headline)
                            Spacer()
                            Text("\(viewModel.cart.reduce(0) { $0 + $1.price })원")
                                .font(.title3.bold())
                                .foregroundColor(.blue)
                        }
                        
                        HStack {
                            Image(systemName: "clock")
                            Text("예상 시간")
                            Spacer()
                            Text("약 \(viewModel.cart.map(\.preparationTime).max() ?? 0)분")
                        }
                        .font(.subheadline)
                        .foregroundColor(.secondary)
                    }
                }
                .navigationTitle("장바구니")
                .toolbar {
                    ToolbarItem(placement: .navigationBarTrailing) {
                        Button("비우기") {
                            withAnimation {
                                viewModel.cart.removeAll()
                            }
                        }
                    }
                }
                .safeAreaInset(edge: .bottom) {
                    OrderButton {
                        showingOrderConfirmation = true
                    }
                }
                .alert("주문하시겠습니까?", isPresented: $showingOrderConfirmation) {
                    Button("취소", role: .cancel) { }
                    Button("주문") {
                        viewModel.placeOrder()
                    }
                } message: {
                    Text("총 \(viewModel.cart.count)개 메뉴를 주문합니다.")
                }
            }
        }
    }
}

struct CartItemRow: View {
    let item: MenuItem
    
    var body: some View {
        HStack {
            Text(item.emoji)
                .font(.title2)
            
            VStack(alignment: .leading) {
                Text(item.name)
                    .font(.headline)
                Text(item.formattedPrice)
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            
            Spacer()
        }
        .padding(.vertical, 4)
    }
}

struct EmptyCartView: View {
    var body: some View {
        VStack(spacing: 20) {
            Image(systemName: "cart")
                .font(.system(size: 80))
                .foregroundColor(.gray)
            
            Text("장바구니가 비어있어요")
                .font(.title2)
                .fontWeight(.medium)
            
            Text("메뉴에서 맛있는 음식을 담아보세요")
                .foregroundColor(.secondary)
        }
        .frame(maxWidth: .infinity, maxHeight: .infinity)
        .background(Color(.systemGroupedBackground))
    }
}

struct OrderButton: View {
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            Text("주문하기")
                .font(.headline)
                .foregroundColor(.white)
                .frame(maxWidth: .infinity)
                .padding()
                .background(Color.blue)
                .cornerRadius(12)
        }
        .padding()
        .background(.regularMaterial)
    }
}
```

## Step 7: 주문 현황 화면

```swift
// Views/OrderStatusView.swift
struct OrderStatusView: View {
    @Bindable var viewModel: MenuViewModel
    
    var body: some View {
        NavigationStack {
            if let order = viewModel.currentOrder {
                OrderTrackingView(order: order)
            } else {
                NoOrderView()
            }
        }
        .navigationTitle("주문 현황")
    }
}

struct OrderTrackingView: View {
    let order: Order
    @State private var animationAmount = 0.0
    
    var body: some View {
        ScrollView {
            VStack(spacing: 30) {
                // 상태 표시
                OrderStatusHeader(status: order.status)
                    .padding(.top)
                
                // 진행 상황
                OrderProgressView(status: order.status)
                
                // 주문 정보
                OrderDetailsCard(order: order)
                
                // 예상 시간
                if order.status != .completed {
                    EstimatedTimeCard(minutes: order.estimatedTime)
                }
            }
            .padding()
        }
    }
}

struct OrderStatusHeader: View {
    let status: OrderStatus
    
    var body: some View {
        VStack(spacing: 12) {
            Image(systemName: status.icon)
                .font(.system(size: 60))
                .foregroundColor(status == .completed ? .green : .blue)
                .symbolEffect(.pulse, isActive: status != .completed)
            
            Text(status.rawValue)
                .font(.title2.bold())
        }
    }
}

struct OrderProgressView: View {
    let status: OrderStatus
    
    var progress: Double {
        switch status {
        case .pending: return 0.25
        case .preparing: return 0.5
        case .delivering: return 0.75
        case .completed: return 1.0
        }
    }
    
    var body: some View {
        VStack(spacing: 20) {
            ProgressView(value: progress)
                .tint(.blue)
                .scaleEffect(y: 2)
            
            HStack {
                ForEach(OrderStatus.allCases, id: \.self) { orderStatus in
                    VStack(spacing: 8) {
                        Image(systemName: orderStatus.icon)
                            .foregroundColor(
                                status.isAfterOrEqual(orderStatus) ? .blue : .gray
                            )
                        
                        Text(orderStatus.rawValue)
                            .font(.caption2)
                            .foregroundColor(
                                status.isAfterOrEqual(orderStatus) ? .primary : .secondary
                            )
                    }
                    
                    if orderStatus != OrderStatus.allCases.last {
                        Spacer()
                    }
                }
            }
        }
        .padding()
        .background(Color(.systemGray6))
        .cornerRadius(12)
    }
}

struct OrderDetailsCard: View {
    let order: Order
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            Text("주문 내역")
                .font(.headline)
            
            Divider()
            
            ForEach(order.items) { item in
                HStack {
                    Text(item.emoji)
                    Text(item.name)
                    Spacer()
                    Text(item.formattedPrice)
                        .foregroundColor(.secondary)
                }
                .font(.subheadline)
            }
            
            Divider()
            
            HStack {
                Text("총 금액")
                    .font(.headline)
                Spacer()
                Text("\(order.totalPrice)원")
                    .font(.headline)
                    .foregroundColor(.blue)
            }
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(radius: 2)
    }
}

struct EstimatedTimeCard: View {
    let minutes: Int
    @State private var timeRemaining: Int
    
    init(minutes: Int) {
        self.minutes = minutes
        self._timeRemaining = State(initialValue: minutes * 60)
    }
    
    var body: some View {
        VStack(spacing: 8) {
            Text("예상 남은 시간")
                .font(.headline)
            
            Text("\(timeRemaining / 60)분 \(timeRemaining % 60)초")
                .font(.largeTitle.monospacedDigit())
                .foregroundColor(.blue)
        }
        .padding()
        .background(Color(.systemGray6))
        .cornerRadius(12)
        .onAppear {
            Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
                if timeRemaining > 0 {
                    timeRemaining -= 1
                }
            }
        }
    }
}

struct NoOrderView: View {
    var body: some View {
        VStack(spacing: 20) {
            Image(systemName: "doc.text.magnifyingglass")
                .font(.system(size: 80))
                .foregroundColor(.gray)
            
            Text("진행 중인 주문이 없습니다")
                .font(.title2)
                .fontWeight(.medium)
            
            Text("메뉴에서 주문을 시작해보세요")
                .foregroundColor(.secondary)
        }
        .frame(maxWidth: .infinity, maxHeight: .infinity)
        .background(Color(.systemGroupedBackground))
    }
}

// Helper Extension
extension OrderStatus {
    func isAfterOrEqual(_ other: OrderStatus) -> Bool {
        let allCases = OrderStatus.allCases
        guard let selfIndex = allCases.firstIndex(of: self),
              let otherIndex = allCases.firstIndex(of: other) else {
            return false
        }
        return selfIndex >= otherIndex
    }
}
```

## Step 8: 위젯 추가

```swift
// Widget/BaegopaWidget.swift
import WidgetKit
import SwiftUI

struct BaegopaWidget: Widget {
    let kind: String = "BaegopaWidget"
    
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: Provider()) { entry in
            BaegopaWidgetView(entry: entry)
        }
        .configurationDisplayName("배고파")
        .description("빠른 주문을 위한 위젯")
        .supportedFamilies([.systemSmall, .systemMedium])
    }
}

struct Provider: TimelineProvider {
    func placeholder(in context: Context) -> SimpleEntry {
        SimpleEntry(date: Date(), recommendation: "김치찌개")
    }
    
    func getSnapshot(in context: Context, completion: @escaping (SimpleEntry) -> ()) {
        let entry = SimpleEntry(date: Date(), recommendation: "불고기")
        completion(entry)
    }
    
    func getTimeline(in context: Context, completion: @escaping (Timeline<Entry>) -> ()) {
        var entries: [SimpleEntry] = []
        let recommendations = ["김치찌개", "불고기", "비빔밥", "떡볶이", "김밥", "라면"]
        
        // 1시간마다 추천 메뉴 변경
        for hourOffset in 0..<5 {
            let entryDate = Calendar.current.date(byAdding: .hour, value: hourOffset, to: Date())!
            let recommendation = recommendations.randomElement()!
            let entry = SimpleEntry(date: entryDate, recommendation: recommendation)
            entries.append(entry)
        }
        
        let timeline = Timeline(entries: entries, policy: .atEnd)
        completion(timeline)
    }
}

struct SimpleEntry: TimelineEntry {
    let date: Date
    let recommendation: String
}

struct BaegopaWidgetView: View {
    var entry: Provider.Entry
    @Environment(\.widgetFamily) var family
    
    var body: some View {
        switch family {
        case .systemSmall:
            SmallWidgetView(recommendation: entry.recommendation)
        case .systemMedium:
            MediumWidgetView(recommendation: entry.recommendation)
        default:
            EmptyView()
        }
    }
}

struct SmallWidgetView: View {
    let recommendation: String
    
    var body: some View {
        VStack(spacing: 8) {
            Text("🍽️")
                .font(.largeTitle)
            
            Text("오늘의 추천")
                .font(.caption)
                .foregroundColor(.secondary)
            
            Text(recommendation)
                .font(.headline)
                .minimumScaleFactor(0.7)
        }
        .padding()
        .containerBackground(for: .widget) {
            Color(.systemBackground)
        }
    }
}

struct MediumWidgetView: View {
    let recommendation: String
    
    var body: some View {
        HStack {
            VStack(alignment: .leading, spacing: 4) {
                Text("배고파?")
                    .font(.headline)
                
                Text("오늘의 추천")
                    .font(.caption)
                    .foregroundColor(.secondary)
                
                Text(recommendation)
                    .font(.title2.bold())
                    .foregroundColor(.blue)
            }
            
            Spacer()
            
            Text("🍲")
                .font(.system(size: 60))
        }
        .padding()
        .containerBackground(for: .widget) {
            Color(.systemBackground)
        }
    }
}
```

## Step 9: App Clip 추가 (선택)

```swift
// AppClip/QuickOrderView.swift
import SwiftUI

struct QuickOrderView: View {
    @State private var selectedItem: MenuItem?
    
    var body: some View {
        VStack(spacing: 20) {
            Text("빠른 주문")
                .font(.largeTitle.bold())
            
            Text("자주 주문하는 메뉴")
                .foregroundColor(.secondary)
            
            // 인기 메뉴 3개만 표시
            VStack(spacing: 12) {
                QuickOrderButton(item: MenuItem(
                    name: "김치찌개", 
                    price: 8000, 
                    emoji: "🍲",
                    preparationTime: 15, 
                    calories: 450
                ))
                
                QuickOrderButton(item: MenuItem(
                    name: "불고기", 
                    price: 12000, 
                    emoji: "🥘",
                    preparationTime: 20, 
                    calories: 550
                ))
                
                QuickOrderButton(item: MenuItem(
                    name: "비빔밥", 
                    price: 9000, 
                    emoji: "🍚",
                    preparationTime: 10, 
                    calories: 500
                ))
            }
            .padding()
            
            Spacer()
            
            Link("전체 앱 다운로드", destination: URL(string: "https://apps.apple.com/app/baegopa")!)
                .font(.footnote)
        }
        .padding()
    }
}

struct QuickOrderButton: View {
    let item: MenuItem
    
    var body: some View {
        Button(action: {
            // App Clip에서 주문 처리
        }) {
            HStack {
                Text(item.emoji)
                    .font(.title)
                
                VStack(alignment: .leading) {
                    Text(item.name)
                        .font(.headline)
                    Text(item.formattedPrice)
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
                
                Spacer()
                
                Image(systemName: "chevron.right")
                    .foregroundColor(.secondary)
            }
            .padding()
            .background(Color(.systemGray6))
            .cornerRadius(12)
        }
        .buttonStyle(.plain)
    }
}
```

## Step 10: 앱 최적화

### 성능 최적화
```swift
// 이미지 캐싱
class ImageCache {
    static let shared = ImageCache()
    private let cache = NSCache<NSString, UIImage>()
    
    func image(for key: String) -> UIImage? {
        cache.object(forKey: key as NSString)
    }
    
    func setImage(_ image: UIImage, for key: String) {
        cache.setObject(image, forKey: key as NSString)
    }
}

// 무거운 연산 최적화
struct OptimizedView: View {
    let items: [MenuItem]
    
    // 비싼 연산은 한 번만
    private var sortedItems: [MenuItem] {
        items.sorted { $0.price < $1.price }
    }
    
    var body: some View {
        List(sortedItems) { item in
            // ...
        }
    }
}
```

## 배포 준비

### 1. 앱 아이콘
- 1024×1024 마스터 이미지 준비
- Xcode가 자동으로 모든 크기 생성

### 2. 스플래시 스크린
```swift
struct LaunchScreen: View {
    var body: some View {
        ZStack {
            Color.blue
            
            VStack {
                Text("🍽️")
                    .font(.system(size: 100))
                
                Text("배고파")
                    .font(.largeTitle.bold())
                    .foregroundColor(.white)
            }
        }
        .ignoresSafeArea()
    }
}
```

### 3. App Store Connect 설정
1. 번들 ID 등록
2. 앱 설명 작성
3. 스크린샷 준비 (각 기기별)
4. 심사 제출

## 핵심 배운 것들

1. **SwiftUI의 선언적 특성**: 상태만 관리하면 UI는 자동
2. **MVVM 패턴**: View와 로직의 분리
3. **비동기 처리**: async/await로 깔끔한 비동기 코드
4. **위젯 통합**: 앱의 존재감을 홈 화면으로 확장
5. **사용자 경험**: 작은 디테일이 큰 차이를 만든다

## 다음 단계

이제 패턴과 아키텍처를 배울 시간이다.

→ [L3: SwiftUI 패턴 - 상태 관리의 예술](../L3_patterns/00_state_management.md)

---

*"Done is better than perfect" - Facebook's Original Motto*