# L2-03: 실시간 협업 기능
## CloudKit 공유, 실시간 동기화, 팀 협업

---

> **"Great things in business are never done by one person."**

CloudKit을 활용한 실시간 협업 시스템으로 팀 생산성을 극대화합니다.

---

## 🎯 목표

**완성 후 결과물**:
- 실시간 작업 공유
- 팀 멤버 초대
- 공동 편집
- 활동 피드
- 댓글 시스템

---

## 🚀 협업 시스템 구현

### 1단계: 팀 프로젝트 관리

**Presentation/Views/Collaboration/TeamProjectView.swift**:

```swift
import SwiftUI
import CloudKit
import Combine

struct TeamProjectView: View {
    @StateObject private var viewModel = TeamProjectViewModel()
    @State private var showingInviteSheet = false
    @State private var showingActivityFeed = false
    @State private var selectedMember: TeamMember?
    @State private var showingVideoCall = false
    
    var body: some View {
        NavigationStack {
            ScrollView {
                VStack(spacing: 20) {
                    // 프로젝트 헤더
                    ProjectHeaderCard(project: viewModel.project)
                        .padding(.horizontal)
                    
                    // 팀 멤버
                    TeamMembersSection(
                        members: viewModel.teamMembers,
                        onMemberTap: { member in
                            selectedMember = member
                        },
                        onInvite: {
                            showingInviteSheet = true
                        }
                    )
                    .padding(.horizontal)
                    
                    // 실시간 활동 인디케이터
                    if !viewModel.activeUsers.isEmpty {
                        ActiveUsersBar(users: viewModel.activeUsers)
                            .padding(.horizontal)
                    }
                    
                    // 진행률 대시보드
                    ProgressDashboard(
                        totalTasks: viewModel.totalTasks,
                        completedTasks: viewModel.completedTasks,
                        overdueCount: viewModel.overdueTasks,
                        todayCount: viewModel.todayTasks
                    )
                    .padding(.horizontal)
                    
                    // 공유 작업 목록
                    SharedTasksSection(
                        tasks: viewModel.sharedTasks,
                        currentUserId: viewModel.currentUserId,
                        onTaskUpdate: { task in
                            Task {
                                await viewModel.updateSharedTask(task)
                            }
                        },
                        onAssignUser: { task, userId in
                            Task {
                                await viewModel.assignTask(task, to: userId)
                            }
                        }
                    )
                    
                    // 최근 활동
                    RecentActivitySection(activities: viewModel.recentActivities)
                        .padding(.horizontal)
                }
                .padding(.vertical)
            }
            .navigationTitle(viewModel.project.name)
            .navigationBarTitleDisplayMode(.large)
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Menu {
                        Button {
                            showingActivityFeed = true
                        } label: {
                            Label("활동 피드", systemImage: "list.bullet.rectangle")
                        }
                        
                        Button {
                            showingVideoCall = true
                        } label: {
                            Label("화상 회의", systemImage: "video")
                        }
                        
                        Divider()
                        
                        Button {
                            Task {
                                await viewModel.exportProject()
                            }
                        } label: {
                            Label("내보내기", systemImage: "square.and.arrow.up")
                        }
                        
                        Button(role: .destructive) {
                            Task {
                                await viewModel.leaveProject()
                            }
                        } label: {
                            Label("프로젝트 나가기", systemImage: "person.crop.circle.badge.minus")
                        }
                    } label: {
                        Image(systemName: "ellipsis.circle")
                    }
                }
            }
            .sheet(isPresented: $showingInviteSheet) {
                InviteMembersView(project: viewModel.project)
            }
            .sheet(isPresented: $showingActivityFeed) {
                ActivityFeedView(projectId: viewModel.project.id)
            }
            .sheet(item: $selectedMember) { member in
                MemberDetailView(member: member, project: viewModel.project)
            }
            .sheet(isPresented: $showingVideoCall) {
                VideoCallView(project: viewModel.project)
            }
            .refreshable {
                await viewModel.refresh()
            }
            .onAppear {
                Task {
                    await viewModel.startRealTimeSync()
                }
            }
            .onDisappear {
                viewModel.stopRealTimeSync()
            }
        }
    }
}

// MARK: - Supporting Views

struct ProjectHeaderCard: View {
    let project: Project
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            HStack {
                Image(systemName: project.icon)
                    .font(.title)
                    .foregroundColor(.white)
                    .frame(width: 50, height: 50)
                    .background(Color(project.color))
                    .clipShape(RoundedRectangle(cornerRadius: 12))
                
                VStack(alignment: .leading, spacing: 4) {
                    Text(project.name)
                        .font(.title2)
                        .fontWeight(.bold)
                    
                    if let description = project.description {
                        Text(description)
                            .font(.caption)
                            .foregroundColor(.secondary)
                            .lineLimit(2)
                    }
                }
                
                Spacer()
                
                // 공유 상태
                Image(systemName: "icloud.and.arrow.up.fill")
                    .foregroundColor(.green)
            }
            
            // 진행률 바
            ProgressView(value: project.progressPercentage / 100)
                .tint(Color(project.color))
            
            Text("\(Int(project.progressPercentage))% 완료")
                .font(.caption)
                .foregroundColor(.secondary)
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(16)
        .shadow(color: .black.opacity(0.1), radius: 5)
    }
}

struct TeamMembersSection: View {
    let members: [TeamMember]
    let onMemberTap: (TeamMember) -> Void
    let onInvite: () -> Void
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            HStack {
                Text("팀 멤버")
                    .font(.headline)
                
                Spacer()
                
                Button(action: onInvite) {
                    Image(systemName: "person.badge.plus")
                        .foregroundColor(.blue)
                }
            }
            
            ScrollView(.horizontal, showsIndicators: false) {
                HStack(spacing: 12) {
                    ForEach(members) { member in
                        MemberCard(member: member) {
                            onMemberTap(member)
                        }
                    }
                    
                    // 초대 버튼
                    Button(action: onInvite) {
                        VStack {
                            Image(systemName: "plus.circle.fill")
                                .font(.title)
                                .foregroundColor(.blue)
                            
                            Text("초대")
                                .font(.caption)
                                .foregroundColor(.blue)
                        }
                        .frame(width: 70, height: 90)
                        .background(Color.blue.opacity(0.1))
                        .cornerRadius(12)
                    }
                }
            }
        }
    }
}

struct MemberCard: View {
    let member: TeamMember
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            VStack(spacing: 8) {
                // 프로필 이미지
                if let avatarURL = member.avatarURL {
                    AsyncImage(url: avatarURL) { image in
                        image
                            .resizable()
                            .scaledToFill()
                    } placeholder: {
                        Image(systemName: "person.fill")
                            .foregroundColor(.white)
                            .frame(width: 30, height: 30)
                    }
                    .frame(width: 50, height: 50)
                    .clipShape(Circle())
                } else {
                    Circle()
                        .fill(Color.blue)
                        .frame(width: 50, height: 50)
                        .overlay(
                            Text(member.initials)
                                .foregroundColor(.white)
                                .fontWeight(.semibold)
                        )
                }
                
                // 온라인 상태 표시
                if member.isOnline {
                    Circle()
                        .fill(Color.green)
                        .frame(width: 12, height: 12)
                        .offset(x: 20, y: -55)
                }
                
                Text(member.name)
                    .font(.caption)
                    .lineLimit(1)
                
                Text(member.role.title)
                    .font(.caption2)
                    .foregroundColor(.secondary)
            }
            .frame(width: 70, height: 90)
            .background(Color(.secondarySystemBackground))
            .cornerRadius(12)
        }
        .buttonStyle(PlainButtonStyle())
    }
}

struct ActiveUsersBar: View {
    let users: [TeamMember]
    
    var body: some View {
        HStack {
            Image(systemName: "circle.fill")
                .foregroundColor(.green)
                .font(.caption)
            
            Text("\(users.count)명이 실시간으로 작업 중")
                .font(.caption)
                .foregroundColor(.secondary)
            
            Spacer()
            
            // 활성 사용자 아바타
            HStack(spacing: -10) {
                ForEach(users.prefix(3)) { user in
                    Circle()
                        .fill(Color.blue)
                        .frame(width: 24, height: 24)
                        .overlay(
                            Text(user.initials)
                                .font(.caption2)
                                .foregroundColor(.white)
                        )
                        .overlay(
                            Circle()
                                .stroke(Color(.systemBackground), lineWidth: 2)
                        )
                }
                
                if users.count > 3 {
                    Circle()
                        .fill(Color.gray)
                        .frame(width: 24, height: 24)
                        .overlay(
                            Text("+\(users.count - 3)")
                                .font(.caption2)
                                .foregroundColor(.white)
                        )
                }
            }
        }
        .padding(.horizontal, 12)
        .padding(.vertical, 8)
        .background(Color.green.opacity(0.1))
        .cornerRadius(8)
    }
}

struct SharedTasksSection: View {
    let tasks: [SharedTask]
    let currentUserId: String
    let onTaskUpdate: (SharedTask) -> Void
    let onAssignUser: (SharedTask, String) -> Void
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            Text("공유 작업")
                .font(.headline)
                .padding(.horizontal)
            
            ForEach(tasks) { task in
                SharedTaskRow(
                    task: task,
                    currentUserId: currentUserId,
                    onUpdate: onTaskUpdate,
                    onAssign: onAssignUser
                )
            }
        }
    }
}

struct SharedTaskRow: View {
    let task: SharedTask
    let currentUserId: String
    let onUpdate: (SharedTask) -> Void
    let onAssign: (SharedTask, String) -> Void
    
    @State private var isExpanded = false
    @State private var showingComments = false
    @State private var showingAssignSheet = false
    
    var body: some View {
        VStack(alignment: .leading, spacing: 0) {
            HStack {
                // 완료 체크박스
                Button {
                    var updatedTask = task
                    updatedTask.isCompleted.toggle()
                    onUpdate(updatedTask)
                } label: {
                    Image(systemName: task.isCompleted ? "checkmark.circle.fill" : "circle")
                        .foregroundColor(task.isCompleted ? .green : .gray)
                        .font(.title3)
                }
                
                VStack(alignment: .leading, spacing: 4) {
                    Text(task.title)
                        .strikethrough(task.isCompleted)
                        .foregroundColor(task.isCompleted ? .secondary : .primary)
                    
                    HStack(spacing: 8) {
                        // 담당자
                        if let assignee = task.assignee {
                            HStack(spacing: 4) {
                                Image(systemName: "person.fill")
                                    .font(.caption2)
                                Text(assignee.name)
                                    .font(.caption)
                            }
                            .padding(.horizontal, 6)
                            .padding(.vertical, 2)
                            .background(Color.blue.opacity(0.1))
                            .cornerRadius(4)
                        }
                        
                        // 마감일
                        if let dueDate = task.dueDate {
                            HStack(spacing: 4) {
                                Image(systemName: "calendar")
                                    .font(.caption2)
                                Text(dueDate, style: .date)
                                    .font(.caption)
                            }
                            .foregroundColor(task.isOverdue ? .red : .secondary)
                        }
                        
                        // 댓글 수
                        if task.commentCount > 0 {
                            HStack(spacing: 4) {
                                Image(systemName: "bubble.left")
                                    .font(.caption2)
                                Text("\(task.commentCount)")
                                    .font(.caption)
                            }
                            .foregroundColor(.secondary)
                        }
                    }
                }
                
                Spacer()
                
                // 실시간 편집 인디케이터
                if let editor = task.currentEditor, editor.id != currentUserId {
                    HStack(spacing: 4) {
                        Circle()
                            .fill(Color.orange)
                            .frame(width: 8, height: 8)
                        Text("\(editor.name) 편집 중")
                            .font(.caption2)
                            .foregroundColor(.orange)
                    }
                }
                
                // 더보기 버튼
                Button {
                    withAnimation {
                        isExpanded.toggle()
                    }
                } label: {
                    Image(systemName: "chevron.down")
                        .rotationEffect(.degrees(isExpanded ? 180 : 0))
                        .foregroundColor(.secondary)
                }
            }
            .padding()
            .background(Color(.secondarySystemBackground))
            
            // 확장된 내용
            if isExpanded {
                VStack(alignment: .leading, spacing: 12) {
                    if let description = task.description {
                        Text(description)
                            .font(.caption)
                            .foregroundColor(.secondary)
                            .padding(.horizontal)
                    }
                    
                    HStack(spacing: 16) {
                        Button {
                            showingAssignSheet = true
                        } label: {
                            Label("담당자 지정", systemImage: "person.badge.plus")
                                .font(.caption)
                        }
                        
                        Button {
                            showingComments = true
                        } label: {
                            Label("댓글", systemImage: "bubble.left")
                                .font(.caption)
                        }
                        
                        Spacer()
                    }
                    .padding(.horizontal)
                    .padding(.bottom, 8)
                }
                .background(Color(.tertiarySystemBackground))
            }
        }
        .cornerRadius(12)
        .padding(.horizontal)
        .sheet(isPresented: $showingComments) {
            CommentsView(task: task)
        }
        .sheet(isPresented: $showingAssignSheet) {
            AssignMemberSheet(task: task, onAssign: { userId in
                onAssign(task, userId)
            })
        }
    }
}
```

### 2단계: CloudKit 실시간 동기화

**Services/RealtimeSync/CloudKitSyncManager.swift**:

```swift
import CloudKit
import Combine

class CloudKitSyncManager: ObservableObject {
    private let container: CKContainer
    private let privateDatabase: CKDatabase
    private let sharedDatabase: CKDatabase
    
    @Published var syncStatus: SyncStatus = .idle
    @Published var lastSyncDate: Date?
    @Published var conflictedRecords: [CKRecord] = []
    
    private var subscriptions: Set<AnyCancellable> = []
    private var changeToken: CKServerChangeToken?
    private var syncTimer: Timer?
    
    init() {
        container = CKContainer(identifier: "iCloud.com.yourapp.taskmaster")
        privateDatabase = container.privateCloudDatabase
        sharedDatabase = container.sharedCloudDatabase
        
        setupSubscriptions()
        startAutoSync()
    }
    
    // MARK: - Real-time Subscriptions
    
    private func setupSubscriptions() {
        // 작업 변경 구독
        let taskSubscription = CKDatabaseSubscription(subscriptionID: "task-changes")
        
        let notificationInfo = CKSubscription.NotificationInfo()
        notificationInfo.shouldSendContentAvailable = true
        notificationInfo.shouldSendMutableContent = true
        taskSubscription.notificationInfo = notificationInfo
        
        privateDatabase.save(taskSubscription) { subscription, error in
            if let error = error {
                print("구독 설정 실패: \(error)")
            } else {
                print("구독 설정 성공")
                self.listenForChanges()
            }
        }
    }
    
    private func listenForChanges() {
        // 변경 사항 감지
        let operation = CKFetchDatabaseChangesOperation(previousServerChangeToken: changeToken)
        
        operation.recordZoneWithIDChangedBlock = { zoneID in
            self.fetchChanges(in: zoneID)
        }
        
        operation.recordZoneWithIDWasDeletedBlock = { zoneID in
            self.handleDeletedZone(zoneID)
        }
        
        operation.changeTokenUpdatedBlock = { token in
            self.changeToken = token
            self.saveChangeToken(token)
        }
        
        operation.fetchDatabaseChangesResultBlock = { result in
            switch result {
            case .success(let token):
                self.changeToken = token
                self.saveChangeToken(token)
                self.syncStatus = .synced
                self.lastSyncDate = Date()
                
            case .failure(let error):
                self.syncStatus = .error(error)
                print("변경 사항 가져오기 실패: \(error)")
            }
        }
        
        privateDatabase.add(operation)
    }
    
    private func fetchChanges(in zoneID: CKRecordZone.ID) {
        let configuration = CKFetchRecordZoneChangesOperation.ZoneConfiguration()
        configuration.previousServerChangeToken = loadZoneChangeToken(for: zoneID)
        
        let operation = CKFetchRecordZoneChangesOperation(
            recordZoneIDs: [zoneID],
            configurationsByRecordZoneID: [zoneID: configuration]
        )
        
        var changedRecords: [CKRecord] = []
        var deletedRecordIDs: [CKRecord.ID] = []
        
        operation.recordWasChangedBlock = { recordID, result in
            switch result {
            case .success(let record):
                changedRecords.append(record)
            case .failure(let error):
                print("레코드 변경 처리 실패: \(error)")
            }
        }
        
        operation.recordWithIDWasDeletedBlock = { recordID, recordType in
            deletedRecordIDs.append(recordID)
        }
        
        operation.recordZoneFetchResultBlock = { zoneID, result in
            switch result {
            case .success(let (token, _, _)):
                self.saveZoneChangeToken(token, for: zoneID)
                
                // 로컬 데이터베이스 업데이트
                Task {
                    await self.updateLocalDatabase(
                        changed: changedRecords,
                        deleted: deletedRecordIDs
                    )
                }
                
            case .failure(let error):
                print("Zone 변경 사항 가져오기 실패: \(error)")
            }
        }
        
        privateDatabase.add(operation)
    }
    
    // MARK: - Conflict Resolution
    
    func resolveConflicts(_ conflicts: [CKRecord]) async {
        for conflict in conflicts {
            await resolveConflict(conflict)
        }
    }
    
    private func resolveConflict(_ serverRecord: CKRecord) async {
        // 충돌 해결 전략: 최신 업데이트 우선
        guard let localRecord = await fetchLocalRecord(with: serverRecord.recordID) else {
            // 로컬 레코드가 없으면 서버 레코드 사용
            await saveToLocal(serverRecord)
            return
        }
        
        let serverModified = serverRecord.modificationDate ?? Date.distantPast
        let localModified = localRecord.modificationDate ?? Date.distantPast
        
        if serverModified > localModified {
            // 서버가 더 최신
            await saveToLocal(serverRecord)
        } else if localModified > serverModified {
            // 로컬이 더 최신
            await uploadToServer(localRecord)
        } else {
            // 병합 필요
            let mergedRecord = await mergeRecords(local: localRecord, server: serverRecord)
            await saveToLocal(mergedRecord)
            await uploadToServer(mergedRecord)
        }
    }
    
    private func mergeRecords(local: CKRecord, server: CKRecord) async -> CKRecord {
        let merged = CKRecord(recordType: local.recordType, recordID: local.recordID)
        
        // 각 필드별로 병합 전략 적용
        for key in local.allKeys() {
            if let localValue = local[key],
               let serverValue = server[key] {
                // 필드별 병합 로직
                merged[key] = mergeField(
                    key: key,
                    localValue: localValue,
                    serverValue: serverValue
                )
            }
        }
        
        return merged
    }
    
    private func mergeField(key: String, localValue: CKRecordValue, serverValue: CKRecordValue) -> CKRecordValue {
        // 필드별 커스텀 병합 로직
        switch key {
        case "title", "description":
            // 텍스트 필드: 더 긴 값 사용
            if let localString = localValue as? String,
               let serverString = serverValue as? String {
                return localString.count > serverString.count ? localString : serverString
            }
            
        case "isCompleted":
            // 완료 상태: true 우선
            if let localBool = localValue as? Bool,
               let serverBool = serverValue as? Bool {
                return localBool || serverBool ? true : false
            }
            
        case "tags":
            // 배열: 합집합
            if let localArray = localValue as? [String],
               let serverArray = serverValue as? [String] {
                return Array(Set(localArray + serverArray)) as CKRecordValue
            }
            
        default:
            // 기본: 서버 값 사용
            return serverValue
        }
        
        return serverValue
    }
    
    // MARK: - Auto Sync
    
    private func startAutoSync() {
        syncTimer = Timer.scheduledTimer(withTimeInterval: 30, repeats: true) { _ in
            Task {
                await self.performSync()
            }
        }
    }
    
    func performSync() async {
        syncStatus = .syncing
        
        do {
            // 1. 로컬 변경사항 업로드
            let localChanges = await fetchLocalChanges()
            try await uploadChanges(localChanges)
            
            // 2. 서버 변경사항 다운로드
            listenForChanges()
            
            // 3. 충돌 해결
            if !conflictedRecords.isEmpty {
                await resolveConflicts(conflictedRecords)
                conflictedRecords.removeAll()
            }
            
            syncStatus = .synced
            lastSyncDate = Date()
            
        } catch {
            syncStatus = .error(error)
        }
    }
    
    // MARK: - Sharing
    
    func shareRecord(_ record: CKRecord, with users: [String]) async throws -> CKShare {
        let share = CKShare(rootRecord: record)
        share[CKShare.SystemFieldKey.title] = "공유 작업" as CKRecordValue
        share[CKShare.SystemFieldKey.shareType] = "com.yourapp.task" as CKRecordValue
        
        // 참가자 추가
        for email in users {
            let lookupInfo = CKUserIdentity.LookupInfo(emailAddress: email)
            
            if let userIdentity = try await container.discoverUserIdentity(with: lookupInfo) {
                let participant = CKShare.Participant(
                    userIdentity: userIdentity,
                    permission: .readWrite,
                    role: .privateUser,
                    acceptanceStatus: .pending
                )
                share.addParticipant(participant)
            }
        }
        
        // CloudKit에 저장
        let operation = CKModifyRecordsOperation(
            recordsToSave: [record, share],
            recordIDsToDelete: nil
        )
        
        return try await withCheckedThrowingContinuation { continuation in
            operation.modifyRecordsResultBlock = { result in
                switch result {
                case .success:
                    continuation.resume(returning: share)
                case .failure(let error):
                    continuation.resume(throwing: error)
                }
            }
            
            sharedDatabase.add(operation)
        }
    }
    
    // MARK: - Helper Methods
    
    private func saveChangeToken(_ token: CKServerChangeToken) {
        if let data = try? NSKeyedArchiver.archivedData(
            withRootObject: token,
            requiringSecureCoding: false
        ) {
            UserDefaults.standard.set(data, forKey: "DatabaseChangeToken")
        }
    }
    
    private func loadChangeToken() -> CKServerChangeToken? {
        guard let data = UserDefaults.standard.data(forKey: "DatabaseChangeToken") else {
            return nil
        }
        
        return try? NSKeyedUnarchiver.unarchivedObject(
            ofClass: CKServerChangeToken.self,
            from: data
        )
    }
    
    private func saveZoneChangeToken(_ token: CKServerChangeToken, for zoneID: CKRecordZone.ID) {
        if let data = try? NSKeyedArchiver.archivedData(
            withRootObject: token,
            requiringSecureCoding: false
        ) {
            UserDefaults.standard.set(data, forKey: "ZoneChangeToken-\(zoneID.zoneName)")
        }
    }
    
    private func loadZoneChangeToken(for zoneID: CKRecordZone.ID) -> CKServerChangeToken? {
        guard let data = UserDefaults.standard.data(forKey: "ZoneChangeToken-\(zoneID.zoneName)") else {
            return nil
        }
        
        return try? NSKeyedUnarchiver.unarchivedObject(
            ofClass: CKServerChangeToken.self,
            from: data
        )
    }
}

// MARK: - Supporting Types

enum SyncStatus: Equatable {
    case idle
    case syncing
    case synced
    case error(Error)
    
    static func == (lhs: SyncStatus, rhs: SyncStatus) -> Bool {
        switch (lhs, rhs) {
        case (.idle, .idle), (.syncing, .syncing), (.synced, .synced):
            return true
        case (.error, .error):
            return true
        default:
            return false
        }
    }
}
```

### 3단계: 댓글 시스템

**Presentation/Views/Collaboration/CommentsView.swift**:

```swift
import SwiftUI

struct CommentsView: View {
    let task: SharedTask
    @StateObject private var viewModel = CommentsViewModel()
    @State private var newComment = ""
    @State private var replyingTo: Comment?
    @State private var editingComment: Comment?
    @FocusState private var isInputFocused: Bool
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 0) {
                // 댓글 목록
                if viewModel.comments.isEmpty {
                    EmptyCommentsView()
                        .frame(maxWidth: .infinity, maxHeight: .infinity)
                } else {
                    ScrollViewReader { proxy in
                        ScrollView {
                            LazyVStack(spacing: 12) {
                                ForEach(viewModel.comments) { comment in
                                    CommentRow(
                                        comment: comment,
                                        currentUserId: viewModel.currentUserId,
                                        onReply: { replyingTo = comment },
                                        onEdit: { editingComment = comment },
                                        onDelete: { 
                                            Task {
                                                await viewModel.deleteComment(comment)
                                            }
                                        },
                                        onReaction: { reaction in
                                            Task {
                                                await viewModel.addReaction(to: comment, reaction: reaction)
                                            }
                                        }
                                    )
                                    .id(comment.id)
                                }
                            }
                            .padding()
                        }
                        .onChange(of: viewModel.comments.count) { _ in
                            // 새 댓글이 추가되면 스크롤
                            if let lastComment = viewModel.comments.last {
                                withAnimation {
                                    proxy.scrollTo(lastComment.id, anchor: .bottom)
                                }
                            }
                        }
                    }
                }
                
                Divider()
                
                // 입력 영역
                CommentInputView(
                    text: $newComment,
                    replyingTo: replyingTo,
                    onSend: {
                        Task {
                            await sendComment()
                        }
                    },
                    onCancelReply: {
                        replyingTo = nil
                    }
                )
                .focused($isInputFocused)
                .padding()
                .background(Color(.systemBackground))
            }
            .navigationTitle("댓글")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("닫기") {
                        // Dismiss
                    }
                }
            }
            .task {
                await viewModel.loadComments(for: task.id)
                await viewModel.startListening()
            }
            .onDisappear {
                viewModel.stopListening()
            }
        }
    }
    
    private func sendComment() async {
        guard !newComment.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty else { return }
        
        let comment = Comment(
            id: UUID(),
            taskId: task.id,
            text: newComment,
            authorId: viewModel.currentUserId,
            authorName: viewModel.currentUserName,
            createdAt: Date(),
            replyToId: replyingTo?.id,
            mentions: extractMentions(from: newComment),
            attachments: []
        )
        
        await viewModel.addComment(comment)
        
        newComment = ""
        replyingTo = nil
        isInputFocused = false
        
        // 햅틱 피드백
        let impact = UIImpactFeedbackGenerator(style: .light)
        impact.impactOccurred()
    }
    
    private func extractMentions(from text: String) -> [String] {
        let pattern = "@(\\w+)"
        let regex = try? NSRegularExpression(pattern: pattern)
        let matches = regex?.matches(
            in: text,
            range: NSRange(location: 0, length: text.utf16.count)
        ) ?? []
        
        return matches.compactMap { match in
            if let range = Range(match.range(at: 1), in: text) {
                return String(text[range])
            }
            return nil
        }
    }
}

struct CommentRow: View {
    let comment: Comment
    let currentUserId: String
    let onReply: () -> Void
    let onEdit: () -> Void
    let onDelete: () -> Void
    let onReaction: (String) -> Void
    
    @State private var showingActions = false
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            // 헤더
            HStack {
                // 프로필 이미지
                Circle()
                    .fill(Color.blue)
                    .frame(width: 32, height: 32)
                    .overlay(
                        Text(comment.authorName.prefix(1))
                            .foregroundColor(.white)
                            .font(.caption)
                            .fontWeight(.semibold)
                    )
                
                VStack(alignment: .leading, spacing: 2) {
                    Text(comment.authorName)
                        .font(.caption)
                        .fontWeight(.semibold)
                    
                    Text(comment.createdAt, style: .relative)
                        .font(.caption2)
                        .foregroundColor(.secondary)
                }
                
                Spacer()
                
                if comment.authorId == currentUserId {
                    Menu {
                        Button {
                            onEdit()
                        } label: {
                            Label("편집", systemImage: "pencil")
                        }
                        
                        Button(role: .destructive) {
                            onDelete()
                        } label: {
                            Label("삭제", systemImage: "trash")
                        }
                    } label: {
                        Image(systemName: "ellipsis")
                            .foregroundColor(.secondary)
                            .font(.caption)
                    }
                }
            }
            
            // 답글 표시
            if let replyToId = comment.replyToId {
                HStack {
                    Image(systemName: "arrow.turn.down.right")
                        .font(.caption2)
                        .foregroundColor(.secondary)
                    
                    Text("답글")
                        .font(.caption2)
                        .foregroundColor(.secondary)
                }
                .padding(.leading, 40)
            }
            
            // 댓글 내용
            Text(comment.text)
                .font(.callout)
                .padding(.leading, comment.replyToId != nil ? 40 : 0)
            
            // 첨부 파일
            if !comment.attachments.isEmpty {
                ScrollView(.horizontal, showsIndicators: false) {
                    HStack {
                        ForEach(comment.attachments) { attachment in
                            AttachmentPreview(attachment: attachment)
                        }
                    }
                }
                .padding(.leading, 40)
            }
            
            // 반응 및 액션
            HStack(spacing: 16) {
                // 반응
                HStack(spacing: 8) {
                    ForEach(["👍", "❤️", "😄", "🎉", "🤔"], id: \.self) { emoji in
                        Button {
                            onReaction(emoji)
                        } label: {
                            Text(emoji)
                                .font(.caption)
                        }
                    }
                }
                
                Spacer()
                
                // 답글 버튼
                Button {
                    onReply()
                } label: {
                    Label("답글", systemImage: "bubble.left")
                        .font(.caption)
                        .foregroundColor(.blue)
                }
            }
            .padding(.leading, 40)
            
            // 반응 표시
            if !comment.reactions.isEmpty {
                HStack {
                    ForEach(Array(comment.reactions.keys), id: \.self) { reaction in
                        HStack(spacing: 2) {
                            Text(reaction)
                            Text("\(comment.reactions[reaction]?.count ?? 0)")
                                .font(.caption2)
                                .foregroundColor(.secondary)
                        }
                        .padding(.horizontal, 6)
                        .padding(.vertical, 2)
                        .background(Color.gray.opacity(0.1))
                        .cornerRadius(8)
                    }
                }
                .padding(.leading, 40)
            }
        }
        .padding(.horizontal, 12)
        .padding(.vertical, 8)
        .background(Color(.secondarySystemBackground))
        .cornerRadius(12)
    }
}

struct CommentInputView: View {
    @Binding var text: String
    let replyingTo: Comment?
    let onSend: () -> Void
    let onCancelReply: () -> Void
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            // 답글 표시
            if let replyingTo = replyingTo {
                HStack {
                    Image(systemName: "arrow.turn.down.right")
                        .font(.caption)
                    
                    Text("\(replyingTo.authorName)에게 답글")
                        .font(.caption)
                    
                    Spacer()
                    
                    Button("취소") {
                        onCancelReply()
                    }
                    .font(.caption)
                    .foregroundColor(.blue)
                }
                .padding(.horizontal, 12)
                .padding(.vertical, 6)
                .background(Color.blue.opacity(0.1))
                .cornerRadius(8)
            }
            
            // 입력 필드
            HStack(alignment: .bottom) {
                TextField("댓글 입력...", text: $text, axis: .vertical)
                    .textFieldStyle(RoundedBorderTextFieldStyle())
                    .lineLimit(1...5)
                
                Button(action: onSend) {
                    Image(systemName: "paperplane.fill")
                        .foregroundColor(.white)
                        .frame(width: 36, height: 36)
                        .background(text.isEmpty ? Color.gray : Color.blue)
                        .clipShape(Circle())
                }
                .disabled(text.isEmpty)
            }
        }
    }
}
```

---

## 🎯 여기서 배운 것

### 1. **CloudKit 공유**
- CKShare 구현
- 참가자 관리
- 권한 제어
- 공유 URL 생성

### 2. **실시간 동기화**
- 변경 감지
- 충돌 해결
- 자동 병합
- 오프라인 지원

### 3. **협업 UI/UX**
- 실시간 인디케이터
- 동시 편집 표시
- 활동 피드
- 멘션 시스템

### 4. **댓글 시스템**
- 스레드 댓글
- 반응 기능
- 멘션 알림
- 첨부 파일

---

## 🎉 성공 확인

**협업 기능 체크리스트**:
- [ ] 실시간으로 작업 동기화
- [ ] 여러 사용자가 동시 편집 가능
- [ ] 충돌 자동 해결
- [ ] 댓글과 반응 추가 가능
- [ ] 활동 내역 추적

---

**완벽합니다! 실시간 협업 시스템이 완성되었습니다.**