# L3: SwiftUI 패턴 - 상태 관리의 예술

## 핵심 공식: UI = f(State, Environment, Time)

```swift
// 기본 공식
UI = f(State)

// 완전한 공식
UI = f(State, Environment, Time)
```

이 공식의 의미:
- **State**: 앱의 현재 상태 (사용자 데이터, UI 상태)
- **Environment**: 외부 컨텍스트 (디바이스 설정, 시스템 상태, 사용자 설정)
- **Time**: 시간에 따른 변화 (애니메이션, 타이머, 비동기 작업)

```swift
struct SmartView: View {
    // State
    @State private var count = 0
    
    // Environment
    @Environment(\.colorScheme) var colorScheme
    @Environment(\.locale) var locale
    
    // Time
    @State private var timer = Timer.publish(every: 1, on: .main, in: .common).autoconnect()
    
    var body: some View {
        // UI = f(State, Environment, Time)
        Text("\(count)")
            .foregroundColor(colorScheme == .dark ? .white : .black)  // Environment
            .onReceive(timer) { _ in  // Time
                count += 1  // State
            }
    }
}
```

## Side Effects 처리

순수 함수인 UI = f(State)에서 Side Effects를 어떻게 처리할까?

```swift
// Side Effects를 격리하는 패턴
struct DataView: View {
    @State private var data: [Item] = []  // 순수한 State
    
    var body: some View {
        List(data) { item in
            ItemRow(item: item)
        }
        .task {  // Side Effect를 명시적으로 격리
            await loadData()
        }
    }
    
    // Side Effect 함수
    private func loadData() async {
        do {
            // 네트워크 호출 (Side Effect)
            let items = try await API.fetchItems()
            
            // State 업데이트 (순수)
            self.data = items
        } catch {
            // 에러 처리
        }
    }
}
```

## 진실의 단일 소스 (Single Source of Truth)

```swift
// 잘못된 예: 진실이 여러 곳에
struct BadView: View {
    @State var count = 0  // 여기도 count
    
    var body: some View {
        ChildView(count: count)  // 복사본 전달
    }
}

struct ChildView: View {
    let count: Int
    @State var localCount = 0  // 또 다른 count
    
    // 어떤 count가 진짜일까?
}

// 올바른 예: 진실은 하나
struct GoodView: View {
    @State private var count = 0  // 유일한 진실
    
    var body: some View {
        ChildView(count: $count)  // 참조 전달
    }
}

struct ChildView: View {
    @Binding var count: Int  // 진실을 참조
}
```

## 상태 관리 패턴 계층

```
┌─────────────────────────────────┐
│         @AppStorage              │ ← 영구 저장
├─────────────────────────────────┤
│      @EnvironmentObject          │ ← 전역 상태
├─────────────────────────────────┤
│        @StateObject              │ ← 뷰가 소유하는 객체
├─────────────────────────────────┤
│      @ObservedObject             │ ← 뷰가 관찰하는 객체
├─────────────────────────────────┤
│         @Binding                 │ ← 상태 참조
├─────────────────────────────────┤
│          @State                  │ ← 지역 상태
└─────────────────────────────────┘
```

## @State: 뷰의 프라이빗 상태

```swift
struct CounterView: View {
    @State private var count = 0  // private이 중요!
    
    var body: some View {
        VStack {
            Text("\(count)")
            Button("증가") { count += 1 }
        }
    }
}

// 왜 private인가?
// 1. 외부에서 직접 변경 방지
// 2. 뷰의 캡슐화 유지
// 3. SwiftUI가 최적화 가능
```

## @Binding: 양방향 연결

```swift
struct ToggleView: View {
    @Binding var isOn: Bool  // 부모의 상태를 제어
    
    var body: some View {
        Toggle("스위치", isOn: $isOn)
    }
}

// 커스텀 Binding 만들기
struct CustomBindingView: View {
    @State private var celsius = 0.0
    
    var fahrenheit: Binding<Double> {
        Binding(
            get: { celsius * 9/5 + 32 },
            set: { celsius = ($0 - 32) * 5/9 }
        )
    }
    
    var body: some View {
        VStack {
            TextField("섭씨", value: $celsius, format: .number)
            TextField("화씨", value: fahrenheit, format: .number)
        }
    }
}
```

## @Observable (iOS 17+): 새로운 패러다임

```swift
import Observation

@Observable
class TodoStore {
    var todos: [Todo] = []
    var filter = Filter.all
    
    var filteredTodos: [Todo] {
        switch filter {
        case .all: return todos
        case .active: return todos.filter { !$0.isCompleted }
        case .completed: return todos.filter { $0.isCompleted }
        }
    }
    
    func add(_ title: String) {
        todos.append(Todo(title: title))
    }
    
    func toggle(_ todo: Todo) {
        if let index = todos.firstIndex(where: { $0.id == todo.id }) {
            todos[index].isCompleted.toggle()
        }
    }
}

struct TodoListView: View {
    @State private var store = TodoStore()
    
    var body: some View {
        List(store.filteredTodos) { todo in
            TodoRow(todo: todo) {
                store.toggle(todo)
            }
        }
    }
}
```

## Environment 패턴

### 시스템 Environment
```swift
struct AdaptiveView: View {
    @Environment(\.colorScheme) var colorScheme
    @Environment(\.horizontalSizeClass) var sizeClass
    @Environment(\.locale) var locale
    @Environment(\.accessibilityEnabled) var accessibilityEnabled
    
    var body: some View {
        if sizeClass == .compact {
            CompactLayout()
        } else {
            RegularLayout()
        }
    }
}
```

### Custom Environment Values
```swift
// 1. EnvironmentKey 정의
private struct ThemeKey: EnvironmentKey {
    static let defaultValue = Theme.light
}

// 2. EnvironmentValues 확장
extension EnvironmentValues {
    var theme: Theme {
        get { self[ThemeKey.self] }
        set { self[ThemeKey.self] = newValue }
    }
}

// 3. View 확장 (편의)
extension View {
    func theme(_ theme: Theme) -> some View {
        environment(\.theme, theme)
    }
}

// 4. 사용
struct ThemedView: View {
    @Environment(\.theme) var theme
    
    var body: some View {
        Text("Hello")
            .foregroundColor(theme.primaryColor)
    }
}
```

## Dependency Injection 패턴

```swift
// 프로토콜 정의
protocol DataServiceProtocol {
    func fetchData() async throws -> [Item]
}

// 실제 구현
class DataService: DataServiceProtocol {
    func fetchData() async throws -> [Item] {
        // API 호출
    }
}

// 테스트용 Mock
class MockDataService: DataServiceProtocol {
    func fetchData() async throws -> [Item] {
        return [Item(name: "Test")]
    }
}

// ViewModel with DI
@Observable
class ItemViewModel {
    private let service: DataServiceProtocol
    var items: [Item] = []
    
    init(service: DataServiceProtocol = DataService()) {
        self.service = service
    }
    
    func loadItems() async {
        do {
            items = try await service.fetchData()
        } catch {
            // 에러 처리
        }
    }
}

// View
struct ItemListView: View {
    @State private var viewModel: ItemViewModel
    
    init(service: DataServiceProtocol = DataService()) {
        self._viewModel = State(wrappedValue: ItemViewModel(service: service))
    }
    
    var body: some View {
        List(viewModel.items) { item in
            Text(item.name)
        }
        .task {
            await viewModel.loadItems()
        }
    }
}

// 테스트
struct ItemListView_Previews: PreviewProvider {
    static var previews: some View {
        ItemListView(service: MockDataService())
    }
}
```

## Container/Presentation 패턴

```swift
// Container: 로직과 상태 관리
struct UserListContainer: View {
    @State private var users: [User] = []
    @State private var isLoading = false
    @State private var error: Error?
    
    var body: some View {
        UserListPresentation(
            users: users,
            isLoading: isLoading,
            error: error,
            onRefresh: loadUsers
        )
        .task {
            await loadUsers()
        }
    }
    
    private func loadUsers() async {
        isLoading = true
        do {
            users = try await UserAPI.fetchUsers()
        } catch {
            self.error = error
        }
        isLoading = false
    }
}

// Presentation: 순수한 UI
struct UserListPresentation: View {
    let users: [User]
    let isLoading: Bool
    let error: Error?
    let onRefresh: () async -> Void
    
    var body: some View {
        if isLoading {
            ProgressView()
        } else if let error = error {
            ErrorView(error: error, retry: onRefresh)
        } else {
            List(users) { user in
                UserRow(user: user)
            }
            .refreshable {
                await onRefresh()
            }
        }
    }
}
```

## Coordinator 패턴 (Navigation)

```swift
// NavigationPath를 사용한 Coordinator
@Observable
class AppCoordinator {
    var path = NavigationPath()
    
    func showDetail(_ item: Item) {
        path.append(item)
    }
    
    func showSettings() {
        path.append(Route.settings)
    }
    
    func popToRoot() {
        path.removeLast(path.count)
    }
    
    func pop() {
        if !path.isEmpty {
            path.removeLast()
        }
    }
}

enum Route: Hashable {
    case settings
    case profile(userId: String)
    case detail(item: Item)
}

struct CoordinatorView: View {
    @State private var coordinator = AppCoordinator()
    
    var body: some View {
        NavigationStack(path: $coordinator.path) {
            HomeView()
                .navigationDestination(for: Item.self) { item in
                    ItemDetailView(item: item)
                }
                .navigationDestination(for: Route.self) { route in
                    switch route {
                    case .settings:
                        SettingsView()
                    case .profile(let userId):
                        ProfileView(userId: userId)
                    case .detail(let item):
                        ItemDetailView(item: item)
                    }
                }
        }
        .environment(coordinator)
    }
}
```

## Repository 패턴

```swift
// Repository Protocol
protocol UserRepositoryProtocol {
    func getUser(id: String) async throws -> User
    func saveUser(_ user: User) async throws
    func deleteUser(id: String) async throws
    func getAllUsers() async throws -> [User]
}

// Implementation
class UserRepository: UserRepositoryProtocol {
    private let localDB: LocalDatabase
    private let remoteAPI: RemoteAPI
    private let cache: Cache<String, User>
    
    init(localDB: LocalDatabase, remoteAPI: RemoteAPI) {
        self.localDB = localDB
        self.remoteAPI = remoteAPI
        self.cache = Cache()
    }
    
    func getUser(id: String) async throws -> User {
        // 1. 캐시 확인
        if let cached = cache[id] {
            return cached
        }
        
        // 2. 로컬 DB 확인
        if let local = try? await localDB.getUser(id) {
            cache[id] = local
            return local
        }
        
        // 3. 원격 API 호출
        let remote = try await remoteAPI.fetchUser(id)
        try? await localDB.saveUser(remote)
        cache[id] = remote
        return remote
    }
    
    func saveUser(_ user: User) async throws {
        // 병렬로 저장
        async let localSave = localDB.saveUser(user)
        async let remoteSave = remoteAPI.updateUser(user)
        
        _ = try await (localSave, remoteSave)
        cache[user.id] = user
    }
}
```

## Redux-like 패턴

```swift
// State
struct AppState {
    var user: User?
    var items: [Item] = []
    var isLoading = false
    var error: Error?
}

// Actions
enum AppAction {
    case login(username: String, password: String)
    case logout
    case loadItems
    case itemsLoaded([Item])
    case errorOccurred(Error)
}

// Reducer
func appReducer(state: inout AppState, action: AppAction) {
    switch action {
    case .login(let username, let password):
        state.isLoading = true
        // Side effect 처리는 middleware에서
        
    case .logout:
        state.user = nil
        state.items = []
        
    case .loadItems:
        state.isLoading = true
        
    case .itemsLoaded(let items):
        state.items = items
        state.isLoading = false
        
    case .errorOccurred(let error):
        state.error = error
        state.isLoading = false
    }
}

// Store
@Observable
class AppStore {
    private(set) var state = AppState()
    
    func dispatch(_ action: AppAction) {
        appReducer(state: &state, action: action)
        
        // Side effects
        Task {
            switch action {
            case .login(let username, let password):
                do {
                    let user = try await AuthAPI.login(username, password)
                    state.user = user
                    dispatch(.loadItems)
                } catch {
                    dispatch(.errorOccurred(error))
                }
                
            case .loadItems:
                do {
                    let items = try await ItemAPI.fetchItems()
                    dispatch(.itemsLoaded(items))
                } catch {
                    dispatch(.errorOccurred(error))
                }
                
            default:
                break
            }
        }
    }
}
```

## ViewModifier 패턴

```swift
// 재사용 가능한 ViewModifier
struct CardStyle: ViewModifier {
    func body(content: Content) -> some View {
        content
            .padding()
            .background(Color(.systemBackground))
            .cornerRadius(12)
            .shadow(color: .black.opacity(0.1), radius: 5)
    }
}

extension View {
    func cardStyle() -> some View {
        modifier(CardStyle())
    }
}

// 조건부 Modifier
extension View {
    @ViewBuilder
    func `if`<Transform: View>(
        _ condition: Bool,
        transform: (Self) -> Transform
    ) -> some View {
        if condition {
            transform(self)
        } else {
            self
        }
    }
}

// 사용
Text("Hello")
    .cardStyle()
    .if(isImportant) { view in
        view.foregroundColor(.red)
    }
```

## Task & Async 패턴

```swift
struct DataView: View {
    @State private var data: [Item] = []
    @State private var isLoading = false
    @State private var error: Error?
    
    var body: some View {
        List(data) { item in
            ItemRow(item: item)
        }
        .task {  // 뷰가 나타날 때 자동 실행, 사라질 때 자동 취소
            await loadData()
        }
        .refreshable {  // Pull to refresh
            await loadData()
        }
    }
    
    private func loadData() async {
        isLoading = true
        do {
            // 병렬 로딩
            async let items = API.fetchItems()
            async let user = API.fetchUser()
            
            let (itemsResult, userResult) = await (items, user)
            self.data = itemsResult
        } catch {
            self.error = error
        }
        isLoading = false
    }
}

// Task 취소 패턴
class SearchViewModel: ObservableObject {
    @Published var searchText = ""
    @Published var results: [SearchResult] = []
    
    private var searchTask: Task<Void, Never>?
    
    func search() {
        searchTask?.cancel()  // 이전 검색 취소
        
        searchTask = Task {
            try? await Task.sleep(for: .milliseconds(300))  // Debounce
            
            guard !Task.isCancelled else { return }
            
            do {
                let results = try await API.search(searchText)
                
                guard !Task.isCancelled else { return }
                
                await MainActor.run {
                    self.results = results
                }
            } catch {
                // 에러 처리
            }
        }
    }
}
```

## Phase 패턴 (AsyncImage처럼)

```swift
enum DataPhase<T> {
    case empty
    case loading
    case success(T)
    case failure(Error)
}

struct PhaseView<T>: View {
    let phase: DataPhase<T>
    let content: (T) -> AnyView
    let placeholder: () -> AnyView
    let error: (Error) -> AnyView
    
    var body: some View {
        switch phase {
        case .empty, .loading:
            placeholder()
        case .success(let data):
            content(data)
        case .failure(let error):
            self.error(error)
        }
    }
}

// 사용
struct UserProfileView: View {
    @State private var phase = DataPhase<User>.empty
    
    var body: some View {
        PhaseView(
            phase: phase,
            content: { user in
                AnyView(UserDetails(user: user))
            },
            placeholder: {
                AnyView(ProgressView())
            },
            error: { error in
                AnyView(ErrorView(error: error))
            }
        )
        .task {
            phase = .loading
            do {
                let user = try await API.fetchUser()
                phase = .success(user)
            } catch {
                phase = .failure(error)
            }
        }
    }
}
```

## 정리: 패턴 선택 가이드

### 언제 어떤 패턴을?

1. **단순한 앱**: @State + @Binding
2. **중간 복잡도**: @Observable + Environment
3. **복잡한 앱**: Repository + Coordinator + DI
4. **대규모 앱**: Redux-like + Clean Architecture

### 안티패턴 피하기

```swift
// ❌ 안티패턴 1: Massive View
struct BadView: View {
    // 100줄의 상태
    // 200줄의 로직
    // 300줄의 UI
}

// ✅ 개선: 분리
struct GoodView: View {
    @StateObject private var viewModel = ViewModel()
    var body: some View {
        ContentView(viewModel: viewModel)
    }
}

// ❌ 안티패턴 2: 과도한 @State
struct BadStateView: View {
    @State var name = ""
    @State var age = 0
    @State var email = ""
    // ... 20개 더
}

// ✅ 개선: 그룹화
struct GoodStateView: View {
    @State private var user = User()
}

// ❌ 안티패턴 3: Force Unwrap
Text(optionalValue!)  // 크래시 위험

// ✅ 개선: 안전한 처리
Text(optionalValue ?? "기본값")
```

## First Principles: 왜 이런 패턴들이 필요한가?

1. **분리 (Separation)**: UI와 로직의 분리
2. **재사용 (Reusability)**: 컴포넌트의 재사용
3. **테스트 (Testability)**: 단위 테스트 가능
4. **확장성 (Scalability)**: 앱이 커져도 관리 가능
5. **협업 (Collaboration)**: 팀원이 이해하기 쉬움

## 다음 레벨로

패턴을 넘어 아키텍처로 나아갈 시간이다.

→ [L4: 시스템 아키텍처 - Clean Architecture in SwiftUI](../L4_architecture/00_clean_architecture.md)

---

*"패턴은 도구다. 모든 문제를 못으로 보게 만드는 망치가 되지 않도록 조심하라." - Anonymous*