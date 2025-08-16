# L2-00: TaskMaster 프로젝트 아키텍처
## MVVM Clean Architecture로 확장 가능한 할일 관리 앱

---

> **"Architecture is about intent."**

확장 가능하고 테스트 가능한 Clean Architecture로 프로덕션 레벨 할일 관리 앱을 만들어봅시다.

---

## 🎯 목표

**완성 후 결과물**:
- MVVM + Clean Architecture
- 완벽한 의존성 주입
- 테스트 가능한 구조
- 확장 가능한 설계

---

## 🚀 프로젝트 구조

### 전체 아키텍처

```
TaskMaster/
├── App/
│   ├── TaskMasterApp.swift
│   ├── AppDelegate.swift
│   └── SceneDelegate.swift
├── Presentation/
│   ├── Views/
│   │   ├── Tasks/
│   │   ├── Projects/
│   │   ├── Calendar/
│   │   └── Settings/
│   ├── ViewModels/
│   │   ├── TaskViewModel.swift
│   │   ├── ProjectViewModel.swift
│   │   └── CalendarViewModel.swift
│   └── Components/
│       ├── TaskRow.swift
│       ├── ProjectCard.swift
│       └── CalendarView.swift
├── Domain/
│   ├── Entities/
│   │   ├── Task.swift
│   │   ├── Project.swift
│   │   └── Tag.swift
│   ├── UseCases/
│   │   ├── TaskUseCase.swift
│   │   ├── ProjectUseCase.swift
│   │   └── SyncUseCase.swift
│   └── Repositories/
│       ├── TaskRepository.swift
│       └── ProjectRepository.swift
├── Data/
│   ├── Repositories/
│   │   ├── TaskRepositoryImpl.swift
│   │   └── ProjectRepositoryImpl.swift
│   ├── DataSources/
│   │   ├── Local/
│   │   │   ├── CoreDataStack.swift
│   │   │   └── TaskMaster.xcdatamodeld
│   │   └── Remote/
│   │       ├── CloudKitManager.swift
│   │       └── APIService.swift
│   └── Models/
│       ├── TaskEntity+CoreData.swift
│       └── ProjectEntity+CoreData.swift
├── Core/
│   ├── DI/
│   │   └── DIContainer.swift
│   ├── Extensions/
│   ├── Utils/
│   └── Constants/
└── Resources/
```

---

## 🏗️ 실습: Clean Architecture 구현

### 1단계: Domain Layer (비즈니스 로직)

**Domain/Entities/Task.swift**:

```swift
import Foundation

// MARK: - Task Entity
struct Task: Identifiable, Codable, Equatable {
    let id: UUID
    var title: String
    var description: String?
    var isCompleted: Bool
    var priority: Priority
    var dueDate: Date?
    var reminder: Date?
    var projectId: UUID?
    var tags: [Tag]
    var subtasks: [Subtask]
    var attachments: [Attachment]
    var location: Location?
    var recurrence: RecurrenceRule?
    var createdAt: Date
    var updatedAt: Date
    var completedAt: Date?
    
    // 계산 프로퍼티
    var isOverdue: Bool {
        guard let dueDate = dueDate, !isCompleted else { return false }
        return dueDate < Date()
    }
    
    var isDueToday: Bool {
        guard let dueDate = dueDate else { return false }
        return Calendar.current.isDateInToday(dueDate)
    }
    
    var isDueTomorrow: Bool {
        guard let dueDate = dueDate else { return false }
        return Calendar.current.isDateInTomorrow(dueDate)
    }
    
    var progressPercentage: Double {
        guard !subtasks.isEmpty else { return isCompleted ? 100 : 0 }
        let completed = subtasks.filter { $0.isCompleted }.count
        return Double(completed) / Double(subtasks.count) * 100
    }
    
    // 초기화
    init(
        id: UUID = UUID(),
        title: String,
        description: String? = nil,
        isCompleted: Bool = false,
        priority: Priority = .medium,
        dueDate: Date? = nil,
        reminder: Date? = nil,
        projectId: UUID? = nil,
        tags: [Tag] = [],
        subtasks: [Subtask] = [],
        attachments: [Attachment] = [],
        location: Location? = nil,
        recurrence: RecurrenceRule? = nil,
        createdAt: Date = Date(),
        updatedAt: Date = Date(),
        completedAt: Date? = nil
    ) {
        self.id = id
        self.title = title
        self.description = description
        self.isCompleted = isCompleted
        self.priority = priority
        self.dueDate = dueDate
        self.reminder = reminder
        self.projectId = projectId
        self.tags = tags
        self.subtasks = subtasks
        self.attachments = attachments
        self.location = location
        self.recurrence = recurrence
        self.createdAt = createdAt
        self.updatedAt = updatedAt
        self.completedAt = completedAt
    }
}

// MARK: - Priority
enum Priority: Int, CaseIterable, Codable {
    case low = 0
    case medium = 1
    case high = 2
    case urgent = 3
    
    var title: String {
        switch self {
        case .low: return "낮음"
        case .medium: return "보통"
        case .high: return "높음"
        case .urgent: return "긴급"
        }
    }
    
    var color: String {
        switch self {
        case .low: return "gray"
        case .medium: return "blue"
        case .high: return "orange"
        case .urgent: return "red"
        }
    }
    
    var icon: String {
        switch self {
        case .low: return "flag"
        case .medium: return "flag.fill"
        case .high: return "exclamationmark.triangle"
        case .urgent: return "exclamationmark.3"
        }
    }
}

// MARK: - Subtask
struct Subtask: Identifiable, Codable, Equatable {
    let id: UUID
    var title: String
    var isCompleted: Bool
    
    init(id: UUID = UUID(), title: String, isCompleted: Bool = false) {
        self.id = id
        self.title = title
        self.isCompleted = isCompleted
    }
}

// MARK: - Tag
struct Tag: Identifiable, Codable, Equatable, Hashable {
    let id: UUID
    var name: String
    var color: String
    var icon: String?
    
    init(id: UUID = UUID(), name: String, color: String = "blue", icon: String? = nil) {
        self.id = id
        self.name = name
        self.color = color
        self.icon = icon
    }
}

// MARK: - Attachment
struct Attachment: Identifiable, Codable, Equatable {
    let id: UUID
    var fileName: String
    var fileType: FileType
    var fileSize: Int64
    var url: URL?
    var thumbnailURL: URL?
    
    enum FileType: String, Codable {
        case image = "image"
        case document = "document"
        case audio = "audio"
        case video = "video"
        case other = "other"
    }
}

// MARK: - Location
struct Location: Codable, Equatable {
    var latitude: Double
    var longitude: Double
    var address: String?
    var placeName: String?
}

// MARK: - RecurrenceRule
struct RecurrenceRule: Codable, Equatable {
    var frequency: Frequency
    var interval: Int
    var endDate: Date?
    var daysOfWeek: [Int]?
    var dayOfMonth: Int?
    
    enum Frequency: String, CaseIterable, Codable {
        case daily = "daily"
        case weekly = "weekly"
        case monthly = "monthly"
        case yearly = "yearly"
        
        var title: String {
            switch self {
            case .daily: return "매일"
            case .weekly: return "매주"
            case .monthly: return "매월"
            case .yearly: return "매년"
            }
        }
    }
}

// MARK: - Project Entity
struct Project: Identifiable, Codable, Equatable {
    let id: UUID
    var name: String
    var description: String?
    var color: String
    var icon: String
    var tasks: [Task]
    var isArchived: Bool
    var sortOrder: Int
    var createdAt: Date
    var updatedAt: Date
    
    var taskCount: Int {
        tasks.count
    }
    
    var completedTaskCount: Int {
        tasks.filter { $0.isCompleted }.count
    }
    
    var progressPercentage: Double {
        guard !tasks.isEmpty else { return 0 }
        return Double(completedTaskCount) / Double(taskCount) * 100
    }
    
    init(
        id: UUID = UUID(),
        name: String,
        description: String? = nil,
        color: String = "blue",
        icon: String = "folder",
        tasks: [Task] = [],
        isArchived: Bool = false,
        sortOrder: Int = 0,
        createdAt: Date = Date(),
        updatedAt: Date = Date()
    ) {
        self.id = id
        self.name = name
        self.description = description
        self.color = color
        self.icon = icon
        self.tasks = tasks
        self.isArchived = isArchived
        self.sortOrder = sortOrder
        self.createdAt = createdAt
        self.updatedAt = updatedAt
    }
}
```

### 2단계: Repository Interfaces

**Domain/Repositories/TaskRepository.swift**:

```swift
import Foundation
import Combine

protocol TaskRepository {
    // CRUD Operations
    func create(_ task: Task) async throws -> Task
    func read(_ id: UUID) async throws -> Task?
    func update(_ task: Task) async throws -> Task
    func delete(_ id: UUID) async throws
    func list(filter: TaskFilter?, sort: TaskSort?) async throws -> [Task]
    
    // Batch Operations
    func createBatch(_ tasks: [Task]) async throws -> [Task]
    func updateBatch(_ tasks: [Task]) async throws -> [Task]
    func deleteBatch(_ ids: [UUID]) async throws
    
    // Search & Filter
    func search(query: String) async throws -> [Task]
    func getTasksByProject(_ projectId: UUID) async throws -> [Task]
    func getTasksByTag(_ tag: Tag) async throws -> [Task]
    func getTasksDueToday() async throws -> [Task]
    func getOverdueTasks() async throws -> [Task]
    func getCompletedTasks(from: Date, to: Date) async throws -> [Task]
    
    // Sync
    func sync() async throws
    func getLastSyncDate() async -> Date?
    
    // Reactive
    var tasksPublisher: AnyPublisher<[Task], Never> { get }
}

// MARK: - Filter & Sort
struct TaskFilter {
    var isCompleted: Bool?
    var priority: Priority?
    var projectId: UUID?
    var tags: [Tag]?
    var dueDateRange: DateRange?
    var searchText: String?
}

struct TaskSort {
    var field: SortField
    var ascending: Bool
    
    enum SortField {
        case title
        case dueDate
        case priority
        case createdAt
        case updatedAt
    }
}

struct DateRange {
    var start: Date
    var end: Date
}
```

### 3단계: Use Cases

**Domain/UseCases/TaskUseCase.swift**:

```swift
import Foundation
import Combine

protocol TaskUseCase {
    // Task Management
    func createTask(title: String, description: String?, priority: Priority, dueDate: Date?, projectId: UUID?) async throws -> Task
    func updateTask(_ task: Task) async throws -> Task
    func deleteTask(_ id: UUID) async throws
    func toggleTaskCompletion(_ id: UUID) async throws -> Task
    func moveTask(_ id: UUID, to projectId: UUID?) async throws -> Task
    
    // Subtasks
    func addSubtask(to taskId: UUID, title: String) async throws -> Task
    func toggleSubtask(taskId: UUID, subtaskId: UUID) async throws -> Task
    func deleteSubtask(taskId: UUID, subtaskId: UUID) async throws -> Task
    
    // Tags
    func addTag(to taskId: UUID, tag: Tag) async throws -> Task
    func removeTag(from taskId: UUID, tagId: UUID) async throws -> Task
    
    // Attachments
    func addAttachment(to taskId: UUID, attachment: Attachment) async throws -> Task
    func removeAttachment(from taskId: UUID, attachmentId: UUID) async throws -> Task
    
    // Reminders
    func setReminder(for taskId: UUID, date: Date) async throws -> Task
    func removeReminder(from taskId: UUID) async throws -> Task
    
    // Batch Operations
    func markTasksAsCompleted(_ ids: [UUID]) async throws -> [Task]
    func deleteTasks(_ ids: [UUID]) async throws
    func moveTasks(_ ids: [UUID], to projectId: UUID?) async throws -> [Task]
    
    // Smart Features
    func suggestDueDate(for title: String) async -> Date?
    func suggestPriority(for title: String) async -> Priority
    func suggestTags(for title: String) async -> [Tag]
    func getSmartSuggestions() async -> [SmartSuggestion]
}

// MARK: - Implementation
class TaskUseCaseImpl: TaskUseCase {
    private let repository: TaskRepository
    private let notificationService: NotificationService
    private let analyticsService: AnalyticsService
    private let aiService: AIService
    
    init(
        repository: TaskRepository,
        notificationService: NotificationService,
        analyticsService: AnalyticsService,
        aiService: AIService
    ) {
        self.repository = repository
        self.notificationService = notificationService
        self.analyticsService = analyticsService
        self.aiService = aiService
    }
    
    func createTask(
        title: String,
        description: String?,
        priority: Priority,
        dueDate: Date?,
        projectId: UUID?
    ) async throws -> Task {
        // AI로 스마트 제안
        let suggestedPriority = await aiService.suggestPriority(for: title)
        let suggestedTags = await aiService.suggestTags(for: title)
        
        let task = Task(
            title: title,
            description: description,
            priority: priority,
            dueDate: dueDate,
            projectId: projectId,
            tags: suggestedTags
        )
        
        let createdTask = try await repository.create(task)
        
        // 리마인더 설정
        if let dueDate = dueDate {
            await notificationService.scheduleReminder(for: createdTask, at: dueDate)
        }
        
        // 분석 이벤트
        analyticsService.track(.taskCreated, properties: [
            "priority": priority.rawValue,
            "has_due_date": dueDate != nil,
            "has_project": projectId != nil
        ])
        
        return createdTask
    }
    
    func toggleTaskCompletion(_ id: UUID) async throws -> Task {
        guard var task = try await repository.read(id) else {
            throw TaskError.notFound
        }
        
        task.isCompleted.toggle()
        task.completedAt = task.isCompleted ? Date() : nil
        task.updatedAt = Date()
        
        let updatedTask = try await repository.update(task)
        
        // 완료 시 알림 취소
        if updatedTask.isCompleted {
            await notificationService.cancelReminder(for: id)
            
            // 반복 작업 처리
            if let recurrence = task.recurrence {
                try await createRecurringTask(from: task, rule: recurrence)
            }
        }
        
        analyticsService.track(
            updatedTask.isCompleted ? .taskCompleted : .taskReopened,
            properties: ["task_id": id.uuidString]
        )
        
        return updatedTask
    }
    
    private func createRecurringTask(from task: Task, rule: RecurrenceRule) async throws {
        var nextTask = task
        nextTask.id = UUID()
        nextTask.isCompleted = false
        nextTask.completedAt = nil
        
        // 다음 날짜 계산
        if let nextDate = calculateNextDate(from: task.dueDate, rule: rule) {
            nextTask.dueDate = nextDate
            nextTask.reminder = nextDate.addingTimeInterval(-3600) // 1시간 전
        }
        
        _ = try await repository.create(nextTask)
    }
    
    private func calculateNextDate(from date: Date?, rule: RecurrenceRule) -> Date? {
        guard let date = date else { return nil }
        
        let calendar = Calendar.current
        var components = DateComponents()
        
        switch rule.frequency {
        case .daily:
            components.day = rule.interval
        case .weekly:
            components.weekOfYear = rule.interval
        case .monthly:
            components.month = rule.interval
        case .yearly:
            components.year = rule.interval
        }
        
        return calendar.date(byAdding: components, to: date)
    }
    
    func getSmartSuggestions() async -> [SmartSuggestion] {
        var suggestions: [SmartSuggestion] = []
        
        // 오늘 할 일 제안
        let todayTasks = try? await repository.getTasksDueToday()
        if let count = todayTasks?.count, count > 0 {
            suggestions.append(SmartSuggestion(
                type: .todayTasks,
                title: "오늘 할 일 \(count)개",
                description: "오늘 완료해야 할 작업이 있습니다",
                action: .showTodayTasks
            ))
        }
        
        // 기한 지난 작업
        let overdueTasks = try? await repository.getOverdueTasks()
        if let count = overdueTasks?.count, count > 0 {
            suggestions.append(SmartSuggestion(
                type: .overdue,
                title: "기한 지난 작업 \(count)개",
                description: "즉시 처리가 필요합니다",
                action: .showOverdueTasks
            ))
        }
        
        // AI 기반 제안
        let aiSuggestions = await aiService.generateSuggestions()
        suggestions.append(contentsOf: aiSuggestions)
        
        return suggestions
    }
    
    // 나머지 메서드 구현...
}

// MARK: - Supporting Types
struct SmartSuggestion {
    enum SuggestionType {
        case todayTasks
        case overdue
        case recurring
        case productivity
        case recommendation
    }
    
    enum Action {
        case showTodayTasks
        case showOverdueTasks
        case createTask(String)
        case showProject(UUID)
    }
    
    let type: SuggestionType
    let title: String
    let description: String
    let action: Action
}

enum TaskError: Error {
    case notFound
    case invalidData
    case syncFailed
    case unauthorized
}
```

### 4단계: Data Layer Implementation

**Data/DataSources/Local/CoreDataStack.swift**:

```swift
import CoreData
import CloudKit

class CoreDataStack {
    static let shared = CoreDataStack()
    
    lazy var persistentContainer: NSPersistentCloudKitContainer = {
        let container = NSPersistentCloudKitContainer(name: "TaskMaster")
        
        // CloudKit 구성
        guard let description = container.persistentStoreDescriptions.first else {
            fatalError("No store description found")
        }
        
        description.setOption(true as NSNumber, forKey: NSPersistentHistoryTrackingKey)
        description.setOption(true as NSNumber, forKey: NSPersistentStoreRemoteChangeNotificationPostOptionKey)
        
        // CloudKit 컨테이너 설정
        description.cloudKitContainerOptions = NSPersistentCloudKitContainerOptions(
            containerIdentifier: "iCloud.com.yourname.taskmaster"
        )
        
        container.loadPersistentStores { _, error in
            if let error = error {
                fatalError("Core Data 로드 실패: \(error)")
            }
        }
        
        container.viewContext.automaticallyMergesChangesFromParent = true
        container.viewContext.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
        
        return container
    }()
    
    var viewContext: NSManagedObjectContext {
        persistentContainer.viewContext
    }
    
    func newBackgroundContext() -> NSManagedObjectContext {
        persistentContainer.newBackgroundContext()
    }
    
    func save(context: NSManagedObjectContext? = nil) {
        let context = context ?? viewContext
        
        guard context.hasChanges else { return }
        
        do {
            try context.save()
        } catch {
            print("Core Data 저장 실패: \(error)")
        }
    }
    
    func performBackgroundTask<T>(_ block: @escaping (NSManagedObjectContext) throws -> T) async throws -> T {
        try await withCheckedThrowingContinuation { continuation in
            persistentContainer.performBackgroundTask { context in
                do {
                    let result = try block(context)
                    continuation.resume(returning: result)
                } catch {
                    continuation.resume(throwing: error)
                }
            }
        }
    }
}
```

### 5단계: Dependency Injection

**Core/DI/DIContainer.swift**:

```swift
import Foundation

protocol DIContainerProtocol {
    func register<T>(_ type: T.Type, factory: @escaping () -> T)
    func resolve<T>(_ type: T.Type) -> T
}

final class DIContainer: DIContainerProtocol {
    static let shared = DIContainer()
    
    private var factories = [String: Any]()
    
    private init() {
        registerDependencies()
    }
    
    func register<T>(_ type: T.Type, factory: @escaping () -> T) {
        let key = String(describing: type)
        factories[key] = factory
    }
    
    func resolve<T>(_ type: T.Type) -> T {
        let key = String(describing: type)
        
        guard let factory = factories[key] as? () -> T else {
            fatalError("Dependency \(T.self) not registered")
        }
        
        return factory()
    }
    
    private func registerDependencies() {
        // Data Layer
        register(CoreDataStack.self) { CoreDataStack.shared }
        
        // Repositories
        register(TaskRepository.self) {
            TaskRepositoryImpl(coreDataStack: self.resolve(CoreDataStack.self))
        }
        
        register(ProjectRepository.self) {
            ProjectRepositoryImpl(coreDataStack: self.resolve(CoreDataStack.self))
        }
        
        // Services
        register(NotificationService.self) { NotificationService() }
        register(AnalyticsService.self) { AnalyticsService() }
        register(AIService.self) { AIService() }
        register(CloudKitManager.self) { CloudKitManager() }
        
        // Use Cases
        register(TaskUseCase.self) {
            TaskUseCaseImpl(
                repository: self.resolve(TaskRepository.self),
                notificationService: self.resolve(NotificationService.self),
                analyticsService: self.resolve(AnalyticsService.self),
                aiService: self.resolve(AIService.self)
            )
        }
        
        // ViewModels
        register(TaskViewModel.self) {
            TaskViewModel(useCase: self.resolve(TaskUseCase.self))
        }
        
        register(ProjectViewModel.self) {
            ProjectViewModel(
                projectRepository: self.resolve(ProjectRepository.self),
                taskRepository: self.resolve(TaskRepository.self)
            )
        }
    }
}

// Property Wrapper for Dependency Injection
@propertyWrapper
struct Injected<T> {
    private var value: T
    
    init() {
        value = DIContainer.shared.resolve(T.self)
    }
    
    var wrappedValue: T {
        get { value }
        set { value = newValue }
    }
}
```

---

## 🎯 여기서 배운 것

### 1. **Clean Architecture**
- Domain/Data/Presentation 분리
- 의존성 역전 원칙
- 테스트 가능한 구조
- 확장 가능한 설계

### 2. **MVVM 패턴**
- View와 비즈니스 로직 분리
- 데이터 바인딩
- Reactive Programming
- ViewModel 테스팅

### 3. **의존성 주입**
- DIContainer 구현
- Property Wrapper 활용
- 테스트 모킹 지원
- 느슨한 결합

### 4. **도메인 모델링**
- Rich Domain Model
- Value Objects
- Business Rules
- Domain Services

---

## 🎉 성공 확인

**아키텍처 체크리스트**:
- [ ] 각 레이어가 독립적으로 테스트 가능
- [ ] 의존성이 단방향으로 흐름
- [ ] 비즈니스 로직이 도메인 레이어에 격리
- [ ] UI 변경이 비즈니스 로직에 영향 없음
- [ ] 새 기능 추가가 기존 코드 수정 없이 가능

---

**완벽합니다! 이제 확장 가능하고 테스트 가능한 Clean Architecture 기반이 완성되었습니다.**