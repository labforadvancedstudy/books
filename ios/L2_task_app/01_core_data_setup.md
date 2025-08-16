# L2-01: Core Data와 CloudKit 동기화
## 오프라인 우선, 실시간 동기화 데이터 레이어

---

> **"Data is the foundation of every great app."**

Core Data와 CloudKit을 완벽하게 통합하여 오프라인에서도 작동하고 여러 기기간 실시간 동기화되는 데이터 레이어를 구축합니다.

---

## 🎯 목표

**완성 후 결과물**:
- Core Data 스키마 설계
- CloudKit 자동 동기화
- 충돌 해결 전략
- 마이그레이션 지원

---

## 🚀 실습: Core Data 구현

### 1단계: Core Data Model 정의

**TaskMaster.xcdatamodeld 구조**:

```xml
<!-- TaskEntity -->
<entity name="TaskEntity" representedClassName="TaskEntity" syncable="YES">
    <attribute name="id" attributeType="UUID" usesScalarValueType="NO"/>
    <attribute name="title" attributeType="String"/>
    <attribute name="desc" optional="YES" attributeType="String"/>
    <attribute name="isCompleted" attributeType="Boolean" defaultValueString="NO" usesScalarValueType="YES"/>
    <attribute name="priority" attributeType="Integer 16" defaultValueString="1" usesScalarValueType="YES"/>
    <attribute name="dueDate" optional="YES" attributeType="Date" usesScalarValueType="NO"/>
    <attribute name="reminder" optional="YES" attributeType="Date" usesScalarValueType="NO"/>
    <attribute name="createdAt" attributeType="Date" usesScalarValueType="NO"/>
    <attribute name="updatedAt" attributeType="Date" usesScalarValueType="NO"/>
    <attribute name="completedAt" optional="YES" attributeType="Date" usesScalarValueType="NO"/>
    <attribute name="recurrenceData" optional="YES" attributeType="Binary"/>
    <attribute name="locationData" optional="YES" attributeType="Binary"/>
    <relationship name="project" optional="YES" maxCount="1" deletionRule="Nullify" destinationEntity="ProjectEntity" inverseName="tasks" inverseEntity="ProjectEntity"/>
    <relationship name="tags" optional="YES" toMany="YES" deletionRule="Nullify" destinationEntity="TagEntity" inverseName="tasks" inverseEntity="TagEntity"/>
    <relationship name="subtasks" optional="YES" toMany="YES" deletionRule="Cascade" destinationEntity="SubtaskEntity" inverseName="task" inverseEntity="SubtaskEntity"/>
    <relationship name="attachments" optional="YES" toMany="YES" deletionRule="Cascade" destinationEntity="AttachmentEntity" inverseName="task" inverseEntity="AttachmentEntity"/>
</entity>

<!-- ProjectEntity -->
<entity name="ProjectEntity" representedClassName="ProjectEntity" syncable="YES">
    <attribute name="id" attributeType="UUID" usesScalarValueType="NO"/>
    <attribute name="name" attributeType="String"/>
    <attribute name="desc" optional="YES" attributeType="String"/>
    <attribute name="color" attributeType="String" defaultValueString="blue"/>
    <attribute name="icon" attributeType="String" defaultValueString="folder"/>
    <attribute name="isArchived" attributeType="Boolean" defaultValueString="NO" usesScalarValueType="YES"/>
    <attribute name="sortOrder" attributeType="Integer 32" defaultValueString="0" usesScalarValueType="YES"/>
    <attribute name="createdAt" attributeType="Date" usesScalarValueType="NO"/>
    <attribute name="updatedAt" attributeType="Date" usesScalarValueType="NO"/>
    <relationship name="tasks" optional="YES" toMany="YES" deletionRule="Cascade" destinationEntity="TaskEntity" inverseName="project" inverseEntity="TaskEntity"/>
</entity>

<!-- TagEntity -->
<entity name="TagEntity" representedClassName="TagEntity" syncable="YES">
    <attribute name="id" attributeType="UUID" usesScalarValueType="NO"/>
    <attribute name="name" attributeType="String"/>
    <attribute name="color" attributeType="String" defaultValueString="blue"/>
    <attribute name="icon" optional="YES" attributeType="String"/>
    <relationship name="tasks" optional="YES" toMany="YES" deletionRule="Nullify" destinationEntity="TaskEntity" inverseName="tags" inverseEntity="TaskEntity"/>
</entity>

<!-- SubtaskEntity -->
<entity name="SubtaskEntity" representedClassName="SubtaskEntity" syncable="YES">
    <attribute name="id" attributeType="UUID" usesScalarValueType="NO"/>
    <attribute name="title" attributeType="String"/>
    <attribute name="isCompleted" attributeType="Boolean" defaultValueString="NO" usesScalarValueType="YES"/>
    <attribute name="sortOrder" attributeType="Integer 32" defaultValueString="0" usesScalarValueType="YES"/>
    <relationship name="task" optional="YES" maxCount="1" deletionRule="Nullify" destinationEntity="TaskEntity" inverseName="subtasks" inverseEntity="TaskEntity"/>
</entity>

<!-- AttachmentEntity -->
<entity name="AttachmentEntity" representedClassName="AttachmentEntity" syncable="YES">
    <attribute name="id" attributeType="UUID" usesScalarValueType="NO"/>
    <attribute name="fileName" attributeType="String"/>
    <attribute name="fileType" attributeType="String"/>
    <attribute name="fileSize" attributeType="Integer 64" defaultValueString="0" usesScalarValueType="YES"/>
    <attribute name="url" optional="YES" attributeType="URI"/>
    <attribute name="thumbnailURL" optional="YES" attributeType="URI"/>
    <relationship name="task" optional="YES" maxCount="1" deletionRule="Nullify" destinationEntity="TaskEntity" inverseName="attachments" inverseEntity="TaskEntity"/>
</entity>
```

### 2단계: NSManagedObject 서브클래스

**Data/Models/TaskEntity+CoreData.swift**:

```swift
import Foundation
import CoreData

@objc(TaskEntity)
public class TaskEntity: NSManagedObject {
    
}

extension TaskEntity {
    @nonobjc public class func fetchRequest() -> NSFetchRequest<TaskEntity> {
        return NSFetchRequest<TaskEntity>(entityName: "TaskEntity")
    }
    
    @NSManaged public var id: UUID
    @NSManaged public var title: String
    @NSManaged public var desc: String?
    @NSManaged public var isCompleted: Bool
    @NSManaged public var priority: Int16
    @NSManaged public var dueDate: Date?
    @NSManaged public var reminder: Date?
    @NSManaged public var createdAt: Date
    @NSManaged public var updatedAt: Date
    @NSManaged public var completedAt: Date?
    @NSManaged public var recurrenceData: Data?
    @NSManaged public var locationData: Data?
    @NSManaged public var project: ProjectEntity?
    @NSManaged public var tags: NSSet?
    @NSManaged public var subtasks: NSSet?
    @NSManaged public var attachments: NSSet?
}

// MARK: - Relationships
extension TaskEntity {
    @objc(addTagsObject:)
    @NSManaged public func addToTags(_ value: TagEntity)
    
    @objc(removeTagsObject:)
    @NSManaged public func removeFromTags(_ value: TagEntity)
    
    @objc(addTags:)
    @NSManaged public func addToTags(_ values: NSSet)
    
    @objc(removeTags:)
    @NSManaged public func removeFromTags(_ values: NSSet)
    
    public var tagsArray: [TagEntity] {
        let set = tags as? Set<TagEntity> ?? []
        return set.sorted { $0.name < $1.name }
    }
    
    public var subtasksArray: [SubtaskEntity] {
        let set = subtasks as? Set<SubtaskEntity> ?? []
        return set.sorted { $0.sortOrder < $1.sortOrder }
    }
    
    public var attachmentsArray: [AttachmentEntity] {
        let set = attachments as? Set<AttachmentEntity> ?? []
        return Array(set)
    }
}

// MARK: - Domain Conversion
extension TaskEntity {
    func toDomain() -> Task {
        var domainTags: [Tag] = []
        if let tags = tags as? Set<TagEntity> {
            domainTags = tags.map { $0.toDomain() }
        }
        
        var domainSubtasks: [Subtask] = []
        if let subtasks = subtasks as? Set<SubtaskEntity> {
            domainSubtasks = subtasks.sorted { $0.sortOrder < $1.sortOrder }.map { $0.toDomain() }
        }
        
        var domainAttachments: [Attachment] = []
        if let attachments = attachments as? Set<AttachmentEntity> {
            domainAttachments = attachments.map { $0.toDomain() }
        }
        
        var location: Location?
        if let locationData = locationData {
            location = try? JSONDecoder().decode(Location.self, from: locationData)
        }
        
        var recurrence: RecurrenceRule?
        if let recurrenceData = recurrenceData {
            recurrence = try? JSONDecoder().decode(RecurrenceRule.self, from: recurrenceData)
        }
        
        return Task(
            id: id,
            title: title,
            description: desc,
            isCompleted: isCompleted,
            priority: Priority(rawValue: Int(priority)) ?? .medium,
            dueDate: dueDate,
            reminder: reminder,
            projectId: project?.id,
            tags: domainTags,
            subtasks: domainSubtasks,
            attachments: domainAttachments,
            location: location,
            recurrence: recurrence,
            createdAt: createdAt,
            updatedAt: updatedAt,
            completedAt: completedAt
        )
    }
    
    func update(from domain: Task, in context: NSManagedObjectContext) {
        title = domain.title
        desc = domain.description
        isCompleted = domain.isCompleted
        priority = Int16(domain.priority.rawValue)
        dueDate = domain.dueDate
        reminder = domain.reminder
        updatedAt = domain.updatedAt
        completedAt = domain.completedAt
        
        if let location = domain.location {
            locationData = try? JSONEncoder().encode(location)
        } else {
            locationData = nil
        }
        
        if let recurrence = domain.recurrence {
            recurrenceData = try? JSONEncoder().encode(recurrence)
        } else {
            recurrenceData = nil
        }
        
        // Update project
        if let projectId = domain.projectId {
            let request = ProjectEntity.fetchRequest()
            request.predicate = NSPredicate(format: "id == %@", projectId as CVarArg)
            project = try? context.fetch(request).first
        } else {
            project = nil
        }
    }
}

// MARK: - Other Entities
extension ProjectEntity {
    func toDomain() -> Project {
        var domainTasks: [Task] = []
        if let tasks = tasks as? Set<TaskEntity> {
            domainTasks = tasks.map { $0.toDomain() }
        }
        
        return Project(
            id: id,
            name: name,
            description: desc,
            color: color,
            icon: icon,
            tasks: domainTasks,
            isArchived: isArchived,
            sortOrder: Int(sortOrder),
            createdAt: createdAt,
            updatedAt: updatedAt
        )
    }
}

extension TagEntity {
    func toDomain() -> Tag {
        return Tag(
            id: id,
            name: name,
            color: color,
            icon: icon
        )
    }
}

extension SubtaskEntity {
    func toDomain() -> Subtask {
        return Subtask(
            id: id,
            title: title,
            isCompleted: isCompleted
        )
    }
}

extension AttachmentEntity {
    func toDomain() -> Attachment {
        return Attachment(
            id: id,
            fileName: fileName,
            fileType: Attachment.FileType(rawValue: fileType) ?? .other,
            fileSize: fileSize,
            url: url,
            thumbnailURL: thumbnailURL
        )
    }
}
```

### 3단계: Repository 구현

**Data/Repositories/TaskRepositoryImpl.swift**:

```swift
import Foundation
import CoreData
import Combine

class TaskRepositoryImpl: TaskRepository {
    private let coreDataStack: CoreDataStack
    private let tasksSubject = CurrentValueSubject<[Task], Never>([])
    
    var tasksPublisher: AnyPublisher<[Task], Never> {
        tasksSubject.eraseToAnyPublisher()
    }
    
    init(coreDataStack: CoreDataStack) {
        self.coreDataStack = coreDataStack
        observeChanges()
        loadTasks()
    }
    
    // MARK: - CRUD Operations
    
    func create(_ task: Task) async throws -> Task {
        try await coreDataStack.performBackgroundTask { context in
            let entity = TaskEntity(context: context)
            entity.id = task.id
            entity.update(from: task, in: context)
            
            try context.save()
            self.loadTasks()
            
            return entity.toDomain()
        }
    }
    
    func read(_ id: UUID) async throws -> Task? {
        let context = coreDataStack.viewContext
        let request = TaskEntity.fetchRequest()
        request.predicate = NSPredicate(format: "id == %@", id as CVarArg)
        
        guard let entity = try context.fetch(request).first else {
            return nil
        }
        
        return entity.toDomain()
    }
    
    func update(_ task: Task) async throws -> Task {
        try await coreDataStack.performBackgroundTask { context in
            let request = TaskEntity.fetchRequest()
            request.predicate = NSPredicate(format: "id == %@", task.id as CVarArg)
            
            guard let entity = try context.fetch(request).first else {
                throw TaskError.notFound
            }
            
            entity.update(from: task, in: context)
            try context.save()
            self.loadTasks()
            
            return entity.toDomain()
        }
    }
    
    func delete(_ id: UUID) async throws {
        try await coreDataStack.performBackgroundTask { context in
            let request = TaskEntity.fetchRequest()
            request.predicate = NSPredicate(format: "id == %@", id as CVarArg)
            
            guard let entity = try context.fetch(request).first else {
                throw TaskError.notFound
            }
            
            context.delete(entity)
            try context.save()
            self.loadTasks()
        }
    }
    
    func list(filter: TaskFilter?, sort: TaskSort?) async throws -> [Task] {
        let context = coreDataStack.viewContext
        let request = TaskEntity.fetchRequest()
        
        // Apply filters
        var predicates: [NSPredicate] = []
        
        if let filter = filter {
            if let isCompleted = filter.isCompleted {
                predicates.append(NSPredicate(format: "isCompleted == %@", NSNumber(value: isCompleted)))
            }
            
            if let priority = filter.priority {
                predicates.append(NSPredicate(format: "priority == %d", priority.rawValue))
            }
            
            if let projectId = filter.projectId {
                predicates.append(NSPredicate(format: "project.id == %@", projectId as CVarArg))
            }
            
            if let searchText = filter.searchText, !searchText.isEmpty {
                predicates.append(NSPredicate(format: "title CONTAINS[cd] %@ OR desc CONTAINS[cd] %@", searchText, searchText))
            }
            
            if let dateRange = filter.dueDateRange {
                predicates.append(NSPredicate(format: "dueDate >= %@ AND dueDate <= %@", dateRange.start as NSDate, dateRange.end as NSDate))
            }
        }
        
        if !predicates.isEmpty {
            request.predicate = NSCompoundPredicate(andPredicateWithSubpredicates: predicates)
        }
        
        // Apply sorting
        if let sort = sort {
            let keyPath: String
            switch sort.field {
            case .title:
                keyPath = "title"
            case .dueDate:
                keyPath = "dueDate"
            case .priority:
                keyPath = "priority"
            case .createdAt:
                keyPath = "createdAt"
            case .updatedAt:
                keyPath = "updatedAt"
            }
            
            request.sortDescriptors = [NSSortDescriptor(key: keyPath, ascending: sort.ascending)]
        } else {
            request.sortDescriptors = [NSSortDescriptor(key: "createdAt", ascending: false)]
        }
        
        let entities = try context.fetch(request)
        return entities.map { $0.toDomain() }
    }
    
    // MARK: - Batch Operations
    
    func createBatch(_ tasks: [Task]) async throws -> [Task] {
        try await coreDataStack.performBackgroundTask { context in
            var createdTasks: [Task] = []
            
            for task in tasks {
                let entity = TaskEntity(context: context)
                entity.id = task.id
                entity.update(from: task, in: context)
                createdTasks.append(entity.toDomain())
            }
            
            try context.save()
            self.loadTasks()
            
            return createdTasks
        }
    }
    
    func updateBatch(_ tasks: [Task]) async throws -> [Task] {
        try await coreDataStack.performBackgroundTask { context in
            var updatedTasks: [Task] = []
            
            for task in tasks {
                let request = TaskEntity.fetchRequest()
                request.predicate = NSPredicate(format: "id == %@", task.id as CVarArg)
                
                if let entity = try context.fetch(request).first {
                    entity.update(from: task, in: context)
                    updatedTasks.append(entity.toDomain())
                }
            }
            
            try context.save()
            self.loadTasks()
            
            return updatedTasks
        }
    }
    
    func deleteBatch(_ ids: [UUID]) async throws {
        try await coreDataStack.performBackgroundTask { context in
            for id in ids {
                let request = TaskEntity.fetchRequest()
                request.predicate = NSPredicate(format: "id == %@", id as CVarArg)
                
                if let entity = try context.fetch(request).first {
                    context.delete(entity)
                }
            }
            
            try context.save()
            self.loadTasks()
        }
    }
    
    // MARK: - Search & Filter
    
    func search(query: String) async throws -> [Task] {
        let filter = TaskFilter(searchText: query)
        return try await list(filter: filter, sort: nil)
    }
    
    func getTasksByProject(_ projectId: UUID) async throws -> [Task] {
        let filter = TaskFilter(projectId: projectId)
        return try await list(filter: filter, sort: nil)
    }
    
    func getTasksByTag(_ tag: Tag) async throws -> [Task] {
        let context = coreDataStack.viewContext
        let request = TaskEntity.fetchRequest()
        request.predicate = NSPredicate(format: "ANY tags.id == %@", tag.id as CVarArg)
        
        let entities = try context.fetch(request)
        return entities.map { $0.toDomain() }
    }
    
    func getTasksDueToday() async throws -> [Task] {
        let calendar = Calendar.current
        let startOfDay = calendar.startOfDay(for: Date())
        let endOfDay = calendar.date(byAdding: .day, value: 1, to: startOfDay)!
        
        let filter = TaskFilter(
            isCompleted: false,
            dueDateRange: DateRange(start: startOfDay, end: endOfDay)
        )
        
        return try await list(filter: filter, sort: TaskSort(field: .dueDate, ascending: true))
    }
    
    func getOverdueTasks() async throws -> [Task] {
        let filter = TaskFilter(
            isCompleted: false,
            dueDateRange: DateRange(start: Date.distantPast, end: Date())
        )
        
        return try await list(filter: filter, sort: TaskSort(field: .dueDate, ascending: true))
    }
    
    func getCompletedTasks(from: Date, to: Date) async throws -> [Task] {
        let context = coreDataStack.viewContext
        let request = TaskEntity.fetchRequest()
        request.predicate = NSPredicate(
            format: "isCompleted == YES AND completedAt >= %@ AND completedAt <= %@",
            from as NSDate,
            to as NSDate
        )
        
        let entities = try context.fetch(request)
        return entities.map { $0.toDomain() }
    }
    
    // MARK: - Sync
    
    func sync() async throws {
        // CloudKit sync is handled automatically by NSPersistentCloudKitContainer
        // This method can be used for manual sync triggers or additional sync logic
        
        // Force a sync by saving the context
        try coreDataStack.viewContext.save()
        
        // Update last sync date
        UserDefaults.standard.set(Date(), forKey: "lastSyncDate")
    }
    
    func getLastSyncDate() async -> Date? {
        UserDefaults.standard.object(forKey: "lastSyncDate") as? Date
    }
    
    // MARK: - Private Methods
    
    private func observeChanges() {
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(contextDidChange),
            name: .NSManagedObjectContextObjectsDidChange,
            object: coreDataStack.viewContext
        )
        
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(persistentStoreRemoteChange),
            name: .NSPersistentStoreRemoteChange,
            object: coreDataStack.persistentContainer.persistentStoreCoordinator
        )
    }
    
    @objc private func contextDidChange(_ notification: Notification) {
        loadTasks()
    }
    
    @objc private func persistentStoreRemoteChange(_ notification: Notification) {
        // Handle remote changes from CloudKit
        coreDataStack.viewContext.perform {
            self.loadTasks()
        }
    }
    
    private func loadTasks() {
        let context = coreDataStack.viewContext
        let request = TaskEntity.fetchRequest()
        request.sortDescriptors = [NSSortDescriptor(key: "createdAt", ascending: false)]
        
        do {
            let entities = try context.fetch(request)
            let tasks = entities.map { $0.toDomain() }
            tasksSubject.send(tasks)
        } catch {
            print("Failed to load tasks: \(error)")
        }
    }
}
```

### 4단계: CloudKit 설정

**Data/DataSources/Remote/CloudKitManager.swift**:

```swift
import CloudKit
import Combine

class CloudKitManager {
    private let container: CKContainer
    private let privateDatabase: CKDatabase
    private let sharedDatabase: CKDatabase
    
    init() {
        container = CKContainer(identifier: "iCloud.com.yourname.taskmaster")
        privateDatabase = container.privateCloudDatabase
        sharedDatabase = container.sharedCloudDatabase
    }
    
    // MARK: - User Authentication
    
    func checkAccountStatus() async throws -> CKAccountStatus {
        try await container.accountStatus()
    }
    
    func requestPermission() async throws -> CKContainer.ApplicationPermissionStatus {
        try await container.requestApplicationPermission(.userDiscoverability)
    }
    
    // MARK: - Sharing
    
    func shareProject(_ project: Project, with participants: [String]) async throws -> CKShare {
        // Create CKRecord for project
        let projectRecord = CKRecord(recordType: "Project", recordID: CKRecord.ID(recordName: project.id.uuidString))
        projectRecord["name"] = project.name
        projectRecord["color"] = project.color
        projectRecord["icon"] = project.icon
        
        // Create share
        let share = CKShare(rootRecord: projectRecord)
        share.publicPermission = .none
        
        // Add participants
        for email in participants {
            let lookupInfo = CKUserIdentity.LookupInfo(emailAddress: email)
            
            if let participant = try await container.discoverUserIdentity(with: lookupInfo) {
                share.addParticipant(
                    CKShare.Participant(
                        userIdentity: participant,
                        permission: .readWrite,
                        role: .privateUser,
                        acceptanceStatus: .pending
                    )
                )
            }
        }
        
        // Save to CloudKit
        let operation = CKModifyRecordsOperation(
            recordsToSave: [projectRecord, share],
            recordIDsToDelete: nil
        )
        
        operation.modifyRecordsResultBlock = { result in
            switch result {
            case .success:
                print("Share created successfully")
            case .failure(let error):
                print("Failed to create share: \(error)")
            }
        }
        
        privateDatabase.add(operation)
        
        return share
    }
    
    // MARK: - Subscription
    
    func subscribeToChanges() {
        // Subscribe to task changes
        let taskSubscription = CKQuerySubscription(
            recordType: "Task",
            predicate: NSPredicate(value: true),
            options: [.firesOnRecordCreation, .firesOnRecordUpdate, .firesOnRecordDeletion]
        )
        
        let notificationInfo = CKSubscription.NotificationInfo()
        notificationInfo.alertBody = "작업이 업데이트되었습니다"
        notificationInfo.soundName = "default"
        notificationInfo.shouldBadge = true
        
        taskSubscription.notificationInfo = notificationInfo
        
        privateDatabase.save(taskSubscription) { subscription, error in
            if let error = error {
                print("Subscription failed: \(error)")
            } else {
                print("Subscription created: \(subscription?.subscriptionID ?? "")")
            }
        }
    }
}
```

---

## 🎯 여기서 배운 것

### 1. **Core Data 설계**
- Entity 관계 모델링
- 마이그레이션 전략
- 성능 최적화
- 배치 작업 처리

### 2. **CloudKit 통합**
- NSPersistentCloudKitContainer
- 자동 동기화
- 충돌 해결
- 공유 기능

### 3. **Repository 패턴**
- 데이터 추상화
- 비동기 처리
- 에러 핸들링
- 리액티브 스트림

---

## 🎉 성공 확인

**데이터 레이어 체크리스트**:
- [ ] Core Data 모델 생성됨
- [ ] CloudKit 동기화 작동
- [ ] 오프라인 모드 지원
- [ ] 실시간 업데이트 반영
- [ ] 충돌 자동 해결

---

**완벽합니다! 이제 오프라인 우선, 실시간 동기화되는 데이터 레이어가 완성되었습니다.**