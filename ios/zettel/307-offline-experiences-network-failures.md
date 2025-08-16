# 오프라인 경험과 네트워크 실패: 연결이 끊어져도 작동하는 앱

## Core Insight
진정으로 훌륭한 앱은 네트워크 연결 상태와 무관하게 일관된 사용자 경험을 제공하며, 오프라인 상황을 제약이 아닌 다른 상호작용 모드로 처리한다.

네트워크는 불안정하다. 지하철, 엘리베이터, 시골 지역, 해외 로밍... 사용자는 언제든 네트워크 연결을 잃을 수 있다. 그때마다 "네트워크에 연결되지 않음" 경고를 보여주는 것은 2024년 기준으로는 변명일 뿐이다. 사용자는 앱이 항상 작동하기를 기대한다.

**오프라인 우선 아키텍처:**

**1. 네트워크 상태 감지와 관리:**
```swift
import Network

class NetworkMonitor: ObservableObject {
    private let monitor = NWPathMonitor()
    private let queue = DispatchQueue(label: "NetworkMonitor")
    
    @Published var isConnected = false
    @Published var connectionType: ConnectionType = .unknown
    @Published var isExpensive = false
    
    enum ConnectionType {
        case wifi
        case cellular
        case ethernet
        case unknown
        
        var description: String {
            switch self {
            case .wifi: return "Wi-Fi"
            case .cellular: return "셀룰러"
            case .ethernet: return "이더넷"
            case .unknown: return "알 수 없음"
            }
        }
    }
    
    init() {
        startMonitoring()
    }
    
    private func startMonitoring() {
        monitor.pathUpdateHandler = { [weak self] path in
            DispatchQueue.main.async {
                self?.isConnected = path.status == .satisfied
                self?.isExpensive = path.isExpensive
                self?.connectionType = self?.getConnectionType(from: path) ?? .unknown
                
                // 연결 상태 변화 알림
                self?.handleConnectionChange(path)
            }
        }
        monitor.start(queue: queue)
    }
    
    private func getConnectionType(from path: NWPath) -> ConnectionType {
        if path.usesInterfaceType(.wifi) {
            return .wifi
        } else if path.usesInterfaceType(.cellular) {
            return .cellular
        } else if path.usesInterfaceType(.wiredEthernet) {
            return .ethernet
        } else {
            return .unknown
        }
    }
    
    private func handleConnectionChange(_ path: NWPath) {
        if path.status == .satisfied {
            NotificationCenter.default.post(name: .networkDidConnect, object: nil)
            // 대기 중인 요청들 실행
            NetworkRequestQueue.shared.processPendingRequests()
        } else {
            NotificationCenter.default.post(name: .networkDidDisconnect, object: nil)
        }
    }
    
    deinit {
        monitor.cancel()
    }
}

extension Notification.Name {
    static let networkDidConnect = Notification.Name("networkDidConnect")
    static let networkDidDisconnect = Notification.Name("networkDidDisconnect")
}
```

**2. 스마트 네트워크 요청 관리:**
```swift
import Foundation

class SmartNetworkManager {
    static let shared = SmartNetworkManager()
    private let cache = NSCache<NSString, CachedResponse>()
    private let networkMonitor = NetworkMonitor()
    
    private init() {
        setupCache()
    }
    
    private func setupCache() {
        cache.countLimit = 100
        cache.totalCostLimit = 50 * 1024 * 1024 // 50MB
    }
    
    func request<T: Codable>(
        url: URL,
        type: T.Type,
        cachePolicy: CachePolicy = .networkFirst,
        completion: @escaping (Result<T, NetworkError>) -> Void
    ) {
        let cacheKey = NSString(string: url.absoluteString)
        
        switch cachePolicy {
        case .cacheFirst:
            // 캐시 우선 - 오프라인 상황에 적합
            if let cached = cache.object(forKey: cacheKey),
               !cached.isExpired {
                if let data = try? JSONDecoder().decode(type, from: cached.data) {
                    completion(.success(data))
                    return
                }
            }
            fallthrough
            
        case .networkFirst:
            // 네트워크 우선 - 일반적인 경우
            if networkMonitor.isConnected {
                performNetworkRequest(url: url, type: type, cacheKey: cacheKey, completion: completion)
            } else {
                returnCachedDataOrError(cacheKey: cacheKey, type: type, completion: completion)
            }
            
        case .networkOnly:
            // 네트워크만 - 실시간 데이터 필요시
            if networkMonitor.isConnected {
                performNetworkRequest(url: url, type: type, cacheKey: cacheKey, completion: completion)
            } else {
                completion(.failure(.noConnection))
            }
            
        case .cacheOnly:
            // 캐시만 - 오프라인 모드
            returnCachedDataOrError(cacheKey: cacheKey, type: type, completion: completion)
        }
    }
    
    private func performNetworkRequest<T: Codable>(
        url: URL,
        type: T.Type,
        cacheKey: NSString,
        completion: @escaping (Result<T, NetworkError>) -> Void
    ) {
        // 연결 타입에 따른 요청 최적화
        var request = URLRequest(url: url)
        
        if networkMonitor.connectionType == .cellular && networkMonitor.isExpensive {
            // 셀룰러 연결시 압축 요청
            request.setValue("gzip", forHTTPHeaderField: "Accept-Encoding")
            request.cachePolicy = .returnCacheDataElseLoad
        }
        
        URLSession.shared.dataTask(with: request) { [weak self] data, response, error in
            if let error = error {
                DispatchQueue.main.async {
                    // 네트워크 오류시 캐시된 데이터 반환
                    self?.returnCachedDataOrError(cacheKey: cacheKey, type: type, completion: completion)
                }
                return
            }
            
            guard let data = data else {
                DispatchQueue.main.async {
                    completion(.failure(.noData))
                }
                return
            }
            
            do {
                let decoded = try JSONDecoder().decode(type, from: data)
                
                // 성공적인 응답 캐싱
                let cachedResponse = CachedResponse(
                    data: data,
                    timestamp: Date(),
                    maxAge: 300 // 5분
                )
                self?.cache.setObject(cachedResponse, forKey: cacheKey)
                
                DispatchQueue.main.async {
                    completion(.success(decoded))
                }
            } catch {
                DispatchQueue.main.async {
                    completion(.failure(.decodingError(error)))
                }
            }
        }.resume()
    }
    
    private func returnCachedDataOrError<T: Codable>(
        cacheKey: NSString,
        type: T.Type,
        completion: @escaping (Result<T, NetworkError>) -> Void
    ) {
        if let cached = cache.object(forKey: cacheKey) {
            do {
                let decoded = try JSONDecoder().decode(type, from: cached.data)
                completion(.success(decoded))
            } catch {
                completion(.failure(.noData))
            }
        } else {
            completion(.failure(.noData))
        }
    }
    
    enum CachePolicy {
        case networkFirst   // 네트워크 우선, 실패시 캐시
        case cacheFirst     // 캐시 우선, 없으면 네트워크
        case networkOnly    // 네트워크만
        case cacheOnly      // 캐시만
    }
}

class CachedResponse {
    let data: Data
    let timestamp: Date
    let maxAge: TimeInterval
    
    var isExpired: Bool {
        return Date().timeIntervalSince(timestamp) > maxAge
    }
    
    init(data: Data, timestamp: Date, maxAge: TimeInterval) {
        self.data = data
        self.timestamp = timestamp
        self.maxAge = maxAge
    }
}

enum NetworkError: Error, LocalizedError {
    case noConnection
    case noData
    case decodingError(Error)
    
    var errorDescription: String? {
        switch self {
        case .noConnection:
            return "인터넷 연결을 확인해주세요"
        case .noData:
            return "데이터를 불러올 수 없습니다"
        case .decodingError(let error):
            return "데이터 처리 오류: \(error.localizedDescription)"
        }
    }
}
```

**3. 오프라인 데이터 동기화:**
```swift
import CoreData

class OfflineSyncManager {
    private let persistentContainer: NSPersistentContainer
    private let networkMonitor = NetworkMonitor()
    private var pendingOperations: [SyncOperation] = []
    
    init(container: NSPersistentContainer) {
        self.persistentContainer = container
        setupNotifications()
        loadPendingOperations()
    }
    
    private func setupNotifications() {
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(networkDidConnect),
            name: .networkDidConnect,
            object: nil
        )
    }
    
    @objc private func networkDidConnect() {
        syncPendingOperations()
    }
    
    // 오프라인 상태에서 생성된 데이터
    func createOfflineItem(_ item: DataItem) {
        let context = persistentContainer.viewContext
        
        // 로컬 임시 ID 생성
        item.localID = UUID().uuidString
        item.needsSync = true
        item.isOfflineCreated = true
        
        do {
            try context.save()
            
            // 동기화 작업 큐에 추가
            let operation = SyncOperation(
                type: .create,
                entityID: item.localID!,
                data: item.toJSON()
            )
            pendingOperations.append(operation)
            savePendingOperations()
            
            // 네트워크 연결시 즉시 동기화 시도
            if networkMonitor.isConnected {
                syncOperation(operation)
            }
        } catch {
            print("Failed to save offline item: \(error)")
        }
    }
    
    // 오프라인 상태에서 업데이트된 데이터
    func updateOfflineItem(_ item: DataItem) {
        item.needsSync = true
        item.lastModified = Date()
        
        let context = persistentContainer.viewContext
        do {
            try context.save()
            
            let operation = SyncOperation(
                type: .update,
                entityID: item.serverID ?? item.localID!,
                data: item.toJSON()
            )
            pendingOperations.append(operation)
            savePendingOperations()
        } catch {
            print("Failed to update offline item: \(error)")
        }
    }
    
    // 대기 중인 모든 작업 동기화
    private func syncPendingOperations() {
        let operationsToSync = pendingOperations
        
        for operation in operationsToSync {
            syncOperation(operation)
        }
    }
    
    private func syncOperation(_ operation: SyncOperation) {
        switch operation.type {
        case .create:
            syncCreateOperation(operation)
        case .update:
            syncUpdateOperation(operation)
        case .delete:
            syncDeleteOperation(operation)
        }
    }
    
    private func syncCreateOperation(_ operation: SyncOperation) {
        guard let url = URL(string: "https://api.example.com/items") else { return }
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.httpBody = operation.data
        
        URLSession.shared.dataTask(with: request) { [weak self] data, response, error in
            if let error = error {
                print("Sync failed: \(error)")
                return
            }
            
            guard let httpResponse = response as? HTTPURLResponse,
                  httpResponse.statusCode == 201,
                  let data = data else {
                print("Invalid server response")
                return
            }
            
            do {
                // 서버에서 받은 ID로 로컬 데이터 업데이트
                let serverResponse = try JSONDecoder().decode(ServerResponse.self, from: data)
                self?.updateLocalItemWithServerID(operation.entityID, serverID: serverResponse.id)
                self?.removePendingOperation(operation)
            } catch {
                print("Failed to decode server response: \(error)")
            }
        }.resume()
    }
    
    private func updateLocalItemWithServerID(_ localID: String, serverID: String) {
        let context = persistentContainer.viewContext
        let request: NSFetchRequest<DataItem> = DataItem.fetchRequest()
        request.predicate = NSPredicate(format: "localID == %@", localID)
        
        do {
            let items = try context.fetch(request)
            if let item = items.first {
                item.serverID = serverID
                item.needsSync = false
                item.isOfflineCreated = false
                try context.save()
            }
        } catch {
            print("Failed to update local item: \(error)")
        }
    }
    
    private func removePendingOperation(_ operation: SyncOperation) {
        if let index = pendingOperations.firstIndex(where: { $0.id == operation.id }) {
            pendingOperations.remove(at: index)
            savePendingOperations()
        }
    }
    
    private func savePendingOperations() {
        // UserDefaults나 별도 파일에 대기 작업 저장
        let encoder = JSONEncoder()
        if let data = try? encoder.encode(pendingOperations) {
            UserDefaults.standard.set(data, forKey: "pendingOperations")
        }
    }
    
    private func loadPendingOperations() {
        guard let data = UserDefaults.standard.data(forKey: "pendingOperations") else { return }
        
        let decoder = JSONDecoder()
        if let operations = try? decoder.decode([SyncOperation].self, from: data) {
            pendingOperations = operations
        }
    }
}

struct SyncOperation: Codable {
    let id = UUID()
    let type: OperationType
    let entityID: String
    let data: Data
    let timestamp = Date()
    
    enum OperationType: String, Codable {
        case create, update, delete
    }
}

struct ServerResponse: Codable {
    let id: String
    let status: String
}
```

**4. 오프라인 UI 패턴:**
```swift
import SwiftUI

struct OfflineAwareView: View {
    @StateObject private var networkMonitor = NetworkMonitor()
    @State private var showOfflineMessage = false
    
    var body: some View {
        NavigationView {
            VStack {
                if !networkMonitor.isConnected {
                    OfflineIndicator()
                }
                
                ContentView()
                    .overlay(alignment: .bottom) {
                        if showOfflineMessage {
                            OfflineActionBar()
                                .transition(.move(edge: .bottom))
                        }
                    }
            }
        }
        .onChange(of: networkMonitor.isConnected) { isConnected in
            withAnimation {
                showOfflineMessage = !isConnected
            }
        }
    }
}

struct OfflineIndicator: View {
    var body: some View {
        HStack {
            Image(systemName: "wifi.slash")
                .foregroundColor(.white)
            Text("오프라인 모드")
                .font(.caption)
                .foregroundColor(.white)
        }
        .padding(.horizontal, 12)
        .padding(.vertical, 6)
        .background(Color.orange)
        .cornerRadius(8)
        .padding(.horizontal)
    }
}

struct OfflineActionBar: View {
    @State private var pendingCount = 0
    
    var body: some View {
        HStack {
            HStack {
                Image(systemName: "clock")
                Text("\(pendingCount)개 항목이 동기화 대기 중")
                    .font(.caption)
            }
            
            Spacer()
            
            Button("수동 동기화") {
                // 수동 동기화 트리거
                NotificationCenter.default.post(name: .manualSyncRequested, object: nil)
            }
            .font(.caption)
            .foregroundColor(.blue)
        }
        .padding()
        .background(Color.gray.opacity(0.1))
        .cornerRadius(8)
        .padding(.horizontal)
    }
}

// 오프라인 상태에서 사용자 액션 처리
struct OfflineActionHandler: ViewModifier {
    @StateObject private var networkMonitor = NetworkMonitor()
    @State private var showOfflineAlert = false
    
    func body(content: Content) -> some View {
        content
            .alert("오프라인 상태", isPresented: $showOfflineAlert) {
                Button("나중에 동기화") { }
                Button("취소", role: .cancel) { }
            } message: {
                Text("이 작업은 인터넷 연결 시 자동으로 동기화됩니다.")
            }
    }
    
    func handleOfflineAction() {
        if !networkMonitor.isConnected {
            showOfflineAlert = true
        }
    }
}

extension View {
    func handleOfflineActions() -> some View {
        modifier(OfflineActionHandler())
    }
}
```

**5. 지능적 데이터 프리페칭:**
```swift
class DataPrefetchManager {
    private let networkMonitor = NetworkMonitor()
    private var prefetchTasks: [URLSessionTask] = []
    
    func startIntelligentPrefetching() {
        // Wi-Fi 연결시에만 프리페칭
        if networkMonitor.connectionType == .wifi && !networkMonitor.isExpensive {
            prefetchCriticalData()
            prefetchUserPreferences()
            prefetchRelatedContent()
        }
    }
    
    private func prefetchCriticalData() {
        // 사용자가 자주 접근하는 데이터 미리 다운로드
        let criticalEndpoints = [
            "https://api.example.com/user/profile",
            "https://api.example.com/user/settings",
            "https://api.example.com/recent-items"
        ]
        
        for endpoint in criticalEndpoints {
            prefetchData(from: endpoint, priority: .high)
        }
    }
    
    private func prefetchUserPreferences() {
        // 사용자 행동 패턴 기반 프리페칭
        let userBehavior = AnalyticsManager.shared.getUserBehaviorPattern()
        
        if userBehavior.frequentlyAccessedCategories.contains("news") {
            prefetchData(from: "https://api.example.com/news/latest", priority: .medium)
        }
        
        if userBehavior.usuallyChecksWeatherInMorning {
            let currentHour = Calendar.current.component(.hour, from: Date())
            if currentHour >= 6 && currentHour <= 10 {
                prefetchData(from: "https://api.example.com/weather", priority: .medium)
            }
        }
    }
    
    private func prefetchData(from urlString: String, priority: URLSessionTask.Priority) {
        guard let url = URL(string: urlString) else { return }
        
        let task = URLSession.shared.dataTask(with: url) { data, response, error in
            if let data = data, error == nil {
                // 캐시에 저장
                self.cacheResponse(data: data, for: url)
            }
        }
        
        task.priority = priority
        task.resume()
        
        prefetchTasks.append(task)
    }
    
    private func cacheResponse(data: Data, for url: URL) {
        let cache = URLCache.shared
        let request = URLRequest(url: url)
        let response = HTTPURLResponse(url: url, statusCode: 200, httpVersion: nil, headerFields: nil)!
        let cachedResponse = CachedURLResponse(response: response, data: data)
        
        cache.storeCachedResponse(cachedResponse, for: request)
    }
    
    func stopPrefetching() {
        prefetchTasks.forEach { $0.cancel() }
        prefetchTasks.removeAll()
    }
}
```

**6. 오프라인 검색과 필터링:**
```swift
class OfflineSearchManager {
    private let searchIndex = NSMutableSet()
    private var cachedData: [SearchableItem] = []
    
    func indexDataForOfflineSearch(_ items: [SearchableItem]) {
        cachedData = items
        
        // 검색 인덱스 구축
        for item in items {
            let searchTerms = extractSearchTerms(from: item)
            searchTerms.forEach { searchIndex.add($0.lowercased()) }
        }
    }
    
    func searchOffline(query: String) -> [SearchableItem] {
        let normalizedQuery = query.lowercased().trimmingCharacters(in: .whitespacesAndNewlines)
        
        return cachedData.filter { item in
            // 제목 검색
            if item.title.lowercased().contains(normalizedQuery) {
                return true
            }
            
            // 내용 검색
            if item.content.lowercased().contains(normalizedQuery) {
                return true
            }
            
            // 태그 검색
            if item.tags.contains(where: { $0.lowercased().contains(normalizedQuery) }) {
                return true
            }
            
            return false
        }.sorted { item1, item2 in
            // 관련성에 따른 정렬
            let score1 = calculateRelevanceScore(item: item1, query: normalizedQuery)
            let score2 = calculateRelevanceScore(item: item2, query: normalizedQuery)
            return score1 > score2
        }
    }
    
    private func extractSearchTerms(from item: SearchableItem) -> [String] {
        var terms: [String] = []
        
        // 제목에서 단어 추출
        terms.append(contentsOf: item.title.components(separatedBy: .whitespacesAndNewlines))
        
        // 태그 추가
        terms.append(contentsOf: item.tags)
        
        // 내용에서 키워드 추출 (긴 텍스트는 제한)
        let contentWords = item.content.components(separatedBy: .whitespacesAndNewlines)
        terms.append(contentsOf: Array(contentWords.prefix(50)))
        
        return terms.filter { !$0.isEmpty }
    }
    
    private func calculateRelevanceScore(item: SearchableItem, query: String) -> Int {
        var score = 0
        
        // 제목에 정확히 포함되면 높은 점수
        if item.title.lowercased().contains(query) {
            score += 10
        }
        
        // 제목이 쿼리로 시작하면 더 높은 점수
        if item.title.lowercased().hasPrefix(query) {
            score += 15
        }
        
        // 태그 일치
        if item.tags.contains(where: { $0.lowercased() == query }) {
            score += 8
        }
        
        // 내용 포함
        if item.content.lowercased().contains(query) {
            score += 3
        }
        
        return score
    }
}

struct SearchableItem {
    let id: String
    let title: String
    let content: String
    let tags: [String]
    let lastModified: Date
}
```

오프라인 경험 설계는 단순히 "연결되지 않았습니다" 메시지를 보여주는 것이 아니다. 사용자가 언제든 자신의 데이터에 접근하고, 작업을 계속할 수 있게 하는 것이다. 네트워크는 한계가 있지만, 좋은 앱은 그 한계를 사용자가 느끼지 못하게 한다.

## Connections
→ [[308-device-specific-optimizations]]
→ [[309-legacy-ios-support-strategies]]
← [[306-memory-management-edge-cases]]
← [[200-urlsession-modern-networking]]

---
Level: L3
Date: 2025-08-16
Tags: #offline #network-failures #caching #sync #user-experience #resilient-design