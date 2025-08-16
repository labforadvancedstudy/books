# L2-04: Siri Shortcuts & 위젯
## App Intents, Live Activities, Interactive Widgets

---

> **"The best interface is no interface."**

Siri와 위젯으로 앱을 벗어나 시스템 전체에서 작업을 관리합니다.

---

## 🎯 목표

**완성 후 결과물**:
- Siri 음성 명령
- 홈 화면 위젯
- 잠금 화면 위젯
- Live Activities
- App Shortcuts

---

## 🚀 App Intents 구현

### 1단계: App Intent 정의

**Intents/TaskIntents.swift**:

```swift
import AppIntents
import SwiftUI
import WidgetKit

// MARK: - 작업 추가 Intent

struct AddTaskIntent: AppIntent {
    static var title: LocalizedStringResource = "작업 추가"
    static var description = IntentDescription("새로운 작업을 빠르게 추가합니다")
    static var openAppWhenRun: Bool = false
    
    @Parameter(title: "제목", requestValueDialog: "어떤 작업을 추가할까요?")
    var title: String
    
    @Parameter(title: "마감일", optionsProvider: DueDateOptionsProvider())
    var dueDate: DueDateOption?
    
    @Parameter(title: "우선순위", default: .medium)
    var priority: TaskPriority
    
    @Parameter(title: "프로젝트")
    var project: ProjectEntity?
    
    @Parameter(title: "메모")
    var notes: String?
    
    static var parameterSummary: some ParameterSummary {
        Summary("작업 \(\.$title) 추가") {
            \.$dueDate
            \.$priority
            \.$project
            \.$notes
        }
    }
    
    func perform() async throws -> some IntentResult & ReturnsValue<TaskEntity> & ShowsSnippetView {
        let taskManager = TaskManager.shared
        
        let task = try await taskManager.createTask(
            title: title,
            dueDate: dueDate?.date,
            priority: priority.toPriority(),
            projectId: project?.id,
            notes: notes
        )
        
        // 위젯 업데이트
        WidgetCenter.shared.reloadAllTimelines()
        
        return .result(
            value: TaskEntity(from: task),
            view: TaskAddedView(task: task)
        )
    }
}

// MARK: - 오늘 할 일 보기

struct ShowTodayTasksIntent: AppIntent {
    static var title: LocalizedStringResource = "오늘 할 일"
    static var description = IntentDescription("오늘 해야 할 작업을 보여줍니다")
    
    func perform() async throws -> some IntentResult & ShowsSnippetView & OpensIntent {
        let tasks = try await TaskManager.shared.getTodayTasks()
        
        if tasks.isEmpty {
            return .result(
                view: EmptyTodayView(),
                opensIntent: AddTaskIntent()
            )
        } else {
            return .result(
                view: TodayTasksView(tasks: tasks),
                opensIntent: OpenTaskListIntent()
            )
        }
    }
}

// MARK: - 작업 완료

struct CompleteTaskIntent: AppIntent {
    static var title: LocalizedStringResource = "작업 완료"
    static var description = IntentDescription("작업을 완료 처리합니다")
    
    @Parameter(title: "작업", requestValueDialog: "어떤 작업을 완료했나요?")
    var task: TaskEntity
    
    static var parameterSummary: some ParameterSummary {
        Summary("작업 \(\.$task) 완료")
    }
    
    func perform() async throws -> some IntentResult & ShowsSnippetView {
        try await TaskManager.shared.completeTask(task.id)
        
        // 위젯 업데이트
        WidgetCenter.shared.reloadAllTimelines()
        
        // 축하 애니메이션과 함께 결과 표시
        return .result(view: TaskCompletedView(task: task))
    }
}

// MARK: - 포모도로 시작

struct StartPomodoroIntent: AppIntent {
    static var title: LocalizedStringResource = "포모도로 시작"
    static var description = IntentDescription("선택한 작업으로 포모도로 타이머를 시작합니다")
    
    @Parameter(title: "작업")
    var task: TaskEntity?
    
    @Parameter(title: "시간", default: 25)
    var duration: Int
    
    func perform() async throws -> some IntentResult & OpensIntent {
        let pomodoroManager = PomodoroManager.shared
        
        pomodoroManager.startSession(
            task: task,
            duration: TimeInterval(duration * 60)
        )
        
        // Live Activity 시작
        if #available(iOS 16.1, *) {
            try await startPomodoroActivity()
        }
        
        return .result(opensIntent: OpenPomodoroIntent())
    }
    
    @available(iOS 16.1, *)
    private func startPomodoroActivity() async throws {
        let attributes = PomodoroAttributes(
            taskName: task?.title ?? "집중 시간",
            duration: duration
        )
        
        let state = PomodoroAttributes.ContentState(
            timeRemaining: duration * 60,
            isRunning: true
        )
        
        let activity = try Activity.request(
            attributes: attributes,
            content: .init(state: state, staleDate: nil),
            pushType: .token
        )
        
        // 토큰 저장
        Task {
            for await pushToken in activity.pushTokenUpdates {
                let tokenString = pushToken.map { String(format: "%02x", $0) }.joined()
                await PomodoroManager.shared.savePushToken(tokenString)
            }
        }
    }
}

// MARK: - Entity Types

struct TaskEntity: AppEntity {
    let id: UUID
    let title: String
    let dueDate: Date?
    let priority: String
    let isCompleted: Bool
    
    static var typeDisplayRepresentation: TypeDisplayRepresentation = "작업"
    
    static var defaultQuery = TaskQuery()
    
    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(
            title: "\(title)",
            subtitle: dueDate != nil ? "\(dueDate!, format: .dateTime)" : nil,
            image: priority == "urgent" ? .init(systemName: "exclamationmark.3") : .init(systemName: "checklist")
        )
    }
}

struct TaskQuery: EntityQuery {
    func entities(for identifiers: [UUID]) async throws -> [TaskEntity] {
        try await TaskManager.shared.getTasks(with: identifiers)
            .map { TaskEntity(from: $0) }
    }
    
    func suggestedEntities() async throws -> IntentItemCollection<TaskEntity> {
        let tasks = try await TaskManager.shared.getIncompleteTasks(limit: 10)
        return IntentItemCollection(items: tasks.map { TaskEntity(from: $0) })
    }
    
    func entities(matching query: String) async throws -> IntentItemCollection<TaskEntity> {
        let tasks = try await TaskManager.shared.searchTasks(query: query)
        return IntentItemCollection(items: tasks.map { TaskEntity(from: $0) })
    }
}

// MARK: - Options Provider

struct DueDateOptionsProvider: DynamicOptionsProvider {
    func results() async throws -> IntentItemCollection<DueDateOption> {
        return IntentItemCollection(items: [
            DueDateOption(title: "오늘", date: Date()),
            DueDateOption(title: "내일", date: Date().addingTimeInterval(86400)),
            DueDateOption(title: "이번 주말", date: nextWeekend()),
            DueDateOption(title: "다음 주", date: nextWeek())
        ])
    }
}

struct DueDateOption: AppEntity {
    let id = UUID()
    let title: String
    let date: Date
    
    static var typeDisplayRepresentation: TypeDisplayRepresentation = "마감일"
    static var defaultQuery = DueDateQuery()
    
    var displayRepresentation: DisplayRepresentation {
        DisplayRepresentation(title: "\(title)")
    }
}

// MARK: - Snippet Views

struct TaskAddedView: View {
    let task: Task
    
    var body: some View {
        HStack {
            Image(systemName: "checkmark.circle.fill")
                .foregroundColor(.green)
                .font(.title)
            
            VStack(alignment: .leading) {
                Text("작업 추가됨")
                    .font(.headline)
                Text(task.title)
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
            
            Spacer()
        }
        .padding()
    }
}

struct TodayTasksView: View {
    let tasks: [Task]
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            HStack {
                Image(systemName: "calendar.day.timeline.left")
                    .foregroundColor(.blue)
                Text("오늘 할 일 \(tasks.count)개")
                    .font(.headline)
            }
            
            ForEach(tasks.prefix(3)) { task in
                HStack {
                    Image(systemName: task.isCompleted ? "checkmark.circle.fill" : "circle")
                        .foregroundColor(task.isCompleted ? .green : .gray)
                    
                    VStack(alignment: .leading) {
                        Text(task.title)
                            .font(.subheadline)
                            .strikethrough(task.isCompleted)
                        
                        if let dueDate = task.dueDate {
                            Text(dueDate, style: .time)
                                .font(.caption)
                                .foregroundColor(.secondary)
                        }
                    }
                    
                    Spacer()
                    
                    if task.priority == .urgent {
                        Image(systemName: "exclamationmark.3")
                            .foregroundColor(.red)
                            .font(.caption)
                    }
                }
            }
            
            if tasks.count > 3 {
                Text("외 \(tasks.count - 3)개...")
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
        }
        .padding()
    }
}
```

### 2단계: App Shortcuts 정의

**Shortcuts/AppShortcuts.swift**:

```swift
import AppIntents
import Foundation

struct TaskMasterShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: AddTaskIntent(),
            phrases: [
                "TaskMaster에서 작업 추가",
                "TaskMaster에서 \(\.$title) 추가",
                "TaskMaster에서 \(\.$title) 작업 \(\.$dueDate)까지",
                "TaskMaster에서 \(\.$priority) 우선순위로 \(\.$title) 추가"
            ],
            shortTitle: "작업 추가",
            systemImageName: "plus.circle"
        )
        
        AppShortcut(
            intent: ShowTodayTasksIntent(),
            phrases: [
                "TaskMaster에서 오늘 할 일",
                "오늘 뭐해야 되지",
                "TaskMaster 오늘 일정"
            ],
            shortTitle: "오늘 할 일",
            systemImageName: "calendar.day.timeline.left"
        )
        
        AppShortcut(
            intent: CompleteTaskIntent(),
            phrases: [
                "TaskMaster에서 \(\.$task) 완료",
                "TaskMaster에서 작업 완료",
                "\(\.$task) 끝냈어"
            ],
            shortTitle: "작업 완료",
            systemImageName: "checkmark.circle"
        )
        
        AppShortcut(
            intent: StartPomodoroIntent(),
            phrases: [
                "TaskMaster에서 포모도로 시작",
                "TaskMaster에서 \(\.$task)로 집중 시간",
                "\(\.$duration)분 타이머 시작"
            ],
            shortTitle: "포모도로",
            systemImageName: "timer"
        )
    }
}
```

### 3단계: 홈 화면 위젯

**Widgets/TaskWidget.swift**:

```swift
import WidgetKit
import SwiftUI
import AppIntents

struct TaskWidgetProvider: AppIntentTimelineProvider {
    func placeholder(in context: Context) -> TaskEntry {
        TaskEntry(date: Date(), tasks: Task.previewTasks, configuration: ConfigurationAppIntent())
    }
    
    func snapshot(for configuration: ConfigurationAppIntent, in context: Context) async -> TaskEntry {
        let tasks = await fetchTasks(for: configuration)
        return TaskEntry(date: Date(), tasks: tasks, configuration: configuration)
    }
    
    func timeline(for configuration: ConfigurationAppIntent, in context: Context) async -> Timeline<TaskEntry> {
        var entries: [TaskEntry] = []
        let tasks = await fetchTasks(for: configuration)
        
        // 15분마다 업데이트
        for hourOffset in 0..<4 {
            let entryDate = Calendar.current.date(
                byAdding: .minute,
                value: hourOffset * 15,
                to: Date()
            )!
            
            entries.append(TaskEntry(
                date: entryDate,
                tasks: tasks,
                configuration: configuration
            ))
        }
        
        return Timeline(entries: entries, policy: .atEnd)
    }
    
    private func fetchTasks(for configuration: ConfigurationAppIntent) async -> [Task] {
        do {
            switch configuration.filter {
            case .today:
                return try await TaskManager.shared.getTodayTasks()
            case .upcoming:
                return try await TaskManager.shared.getUpcomingTasks()
            case .overdue:
                return try await TaskManager.shared.getOverdueTasks()
            case .highPriority:
                return try await TaskManager.shared.getHighPriorityTasks()
            }
        } catch {
            return []
        }
    }
}

struct TaskEntry: TimelineEntry {
    let date: Date
    let tasks: [Task]
    let configuration: ConfigurationAppIntent
}

struct TaskWidgetView: View {
    @Environment(\.widgetFamily) var widgetFamily
    var entry: TaskWidgetProvider.Entry
    
    var body: some View {
        switch widgetFamily {
        case .systemSmall:
            SmallTaskWidget(entry: entry)
        case .systemMedium:
            MediumTaskWidget(entry: entry)
        case .systemLarge:
            LargeTaskWidget(entry: entry)
        case .accessoryCircular:
            CircularTaskWidget(entry: entry)
        case .accessoryRectangular:
            RectangularTaskWidget(entry: entry)
        case .accessoryInline:
            InlineTaskWidget(entry: entry)
        default:
            EmptyView()
        }
    }
}

// MARK: - Small Widget

struct SmallTaskWidget: View {
    let entry: TaskWidgetProvider.Entry
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            HStack {
                Image(systemName: entry.configuration.filter.icon)
                    .foregroundColor(.blue)
                Text(entry.configuration.filter.title)
                    .font(.caption)
                    .fontWeight(.semibold)
            }
            
            if entry.tasks.isEmpty {
                Spacer()
                Text("작업 없음")
                    .font(.caption)
                    .foregroundColor(.secondary)
                Spacer()
            } else {
                ForEach(entry.tasks.prefix(3)) { task in
                    TaskRowCompact(task: task)
                }
            }
            
            Spacer()
            
            HStack {
                Text("\(entry.tasks.count)개")
                    .font(.caption2)
                    .foregroundColor(.secondary)
                
                Spacer()
                
                Text(entry.date, style: .time)
                    .font(.caption2)
                    .foregroundColor(.secondary)
            }
        }
        .padding()
        .widgetURL(URL(string: "taskmaster://tasks/\(entry.configuration.filter.rawValue)"))
    }
}

struct TaskRowCompact: View {
    let task: Task
    
    var body: some View {
        HStack(spacing: 4) {
            Image(systemName: task.isCompleted ? "checkmark.circle.fill" : "circle")
                .font(.caption2)
                .foregroundColor(task.isCompleted ? .green : priorityColor)
            
            Text(task.title)
                .font(.caption)
                .lineLimit(1)
                .strikethrough(task.isCompleted)
            
            Spacer()
            
            if task.isOverdue {
                Image(systemName: "exclamationmark.triangle.fill")
                    .font(.caption2)
                    .foregroundColor(.red)
            }
        }
    }
    
    var priorityColor: Color {
        switch task.priority {
        case .urgent: return .red
        case .high: return .orange
        case .medium: return .blue
        case .low: return .gray
        }
    }
}

// MARK: - Medium Widget

struct MediumTaskWidget: View {
    let entry: TaskWidgetProvider.Entry
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            // 헤더
            HStack {
                VStack(alignment: .leading) {
                    Text(entry.configuration.filter.title)
                        .font(.headline)
                    Text("\(entry.tasks.count)개 작업")
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
                
                Spacer()
                
                // 진행률
                if !entry.tasks.isEmpty {
                    ProgressCircle(
                        progress: Double(entry.tasks.filter { $0.isCompleted }.count) / Double(entry.tasks.count)
                    )
                    .frame(width: 40, height: 40)
                }
            }
            
            // 작업 목록
            if entry.tasks.isEmpty {
                HStack {
                    Spacer()
                    Text("작업 없음")
                        .foregroundColor(.secondary)
                    Spacer()
                }
            } else {
                VStack(spacing: 6) {
                    ForEach(entry.tasks.prefix(3)) { task in
                        MediumTaskRow(task: task)
                    }
                }
            }
            
            Spacer()
        }
        .padding()
        .widgetURL(URL(string: "taskmaster://tasks/\(entry.configuration.filter.rawValue)"))
    }
}

struct MediumTaskRow: View {
    let task: Task
    
    var body: some View {
        HStack {
            Button(intent: CompleteTaskIntent(task: TaskEntity(from: task))) {
                Image(systemName: task.isCompleted ? "checkmark.circle.fill" : "circle")
                    .foregroundColor(task.isCompleted ? .green : .gray)
            }
            .buttonStyle(.plain)
            
            VStack(alignment: .leading, spacing: 2) {
                Text(task.title)
                    .font(.subheadline)
                    .lineLimit(1)
                    .strikethrough(task.isCompleted)
                
                if let dueDate = task.dueDate {
                    Text(dueDate, style: .time)
                        .font(.caption2)
                        .foregroundColor(.secondary)
                }
            }
            
            Spacer()
            
            if task.priority == .urgent {
                Image(systemName: "exclamationmark.3")
                    .foregroundColor(.red)
                    .font(.caption)
            }
        }
    }
}

// MARK: - Large Widget

struct LargeTaskWidget: View {
    let entry: TaskWidgetProvider.Entry
    @State private var selectedTab = 0
    
    var body: some View {
        VStack(spacing: 0) {
            // 헤더 with tabs
            HStack {
                Text("TaskMaster")
                    .font(.title2)
                    .fontWeight(.bold)
                
                Spacer()
                
                Button(intent: AddTaskIntent()) {
                    Image(systemName: "plus.circle.fill")
                        .font(.title2)
                        .foregroundColor(.blue)
                }
                .buttonStyle(.plain)
            }
            .padding()
            
            // 통계
            HStack(spacing: 16) {
                StatCard(
                    title: "오늘",
                    value: "\(entry.tasks.filter { $0.isDueToday }.count)",
                    color: .blue
                )
                
                StatCard(
                    title: "기한 지남",
                    value: "\(entry.tasks.filter { $0.isOverdue }.count)",
                    color: .red
                )
                
                StatCard(
                    title: "완료",
                    value: "\(entry.tasks.filter { $0.isCompleted }.count)",
                    color: .green
                )
                
                StatCard(
                    title: "진행률",
                    value: "\(Int((Double(entry.tasks.filter { $0.isCompleted }.count) / Double(max(entry.tasks.count, 1))) * 100))%",
                    color: .orange
                )
            }
            .padding(.horizontal)
            
            // 작업 목록
            ScrollView {
                VStack(spacing: 8) {
                    ForEach(entry.tasks.prefix(6)) { task in
                        LargeTaskRow(task: task)
                            .padding(.horizontal)
                    }
                }
                .padding(.vertical, 8)
            }
            
            Spacer()
        }
        .widgetURL(URL(string: "taskmaster://tasks"))
    }
}

struct StatCard: View {
    let title: String
    let value: String
    let color: Color
    
    var body: some View {
        VStack(spacing: 4) {
            Text(value)
                .font(.headline)
                .foregroundColor(color)
            Text(title)
                .font(.caption2)
                .foregroundColor(.secondary)
        }
        .frame(maxWidth: .infinity)
        .padding(.vertical, 8)
        .background(color.opacity(0.1))
        .cornerRadius(8)
    }
}

// MARK: - Lock Screen Widgets

struct CircularTaskWidget: View {
    let entry: TaskWidgetProvider.Entry
    
    var body: some View {
        ZStack {
            AccessoryWidgetBackground()
            
            VStack {
                Image(systemName: "checklist")
                    .font(.title2)
                
                Text("\(entry.tasks.filter { !$0.isCompleted }.count)")
                    .font(.headline)
            }
        }
    }
}

struct RectangularTaskWidget: View {
    let entry: TaskWidgetProvider.Entry
    
    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            HStack {
                Image(systemName: "checklist")
                Text("작업")
                Spacer()
                Text("\(entry.tasks.filter { !$0.isCompleted }.count)")
            }
            .font(.caption)
            
            if let nextTask = entry.tasks.first(where: { !$0.isCompleted }) {
                Text(nextTask.title)
                    .font(.caption2)
                    .lineLimit(2)
            }
        }
    }
}

// MARK: - Widget Configuration

struct ConfigurationAppIntent: WidgetConfigurationIntent {
    static var title: LocalizedStringResource = "위젯 설정"
    static var description = IntentDescription("표시할 작업을 선택하세요")
    
    @Parameter(title: "필터", default: .today)
    var filter: TaskFilterOption
}

enum TaskFilterOption: String, AppEnum {
    case today = "today"
    case upcoming = "upcoming"
    case overdue = "overdue"
    case highPriority = "high"
    
    static var typeDisplayRepresentation: TypeDisplayRepresentation = "필터"
    
    static var caseDisplayRepresentations: [TaskFilterOption: DisplayRepresentation] = [
        .today: "오늘",
        .upcoming: "예정",
        .overdue: "기한 지남",
        .highPriority: "중요"
    ]
    
    var title: String {
        switch self {
        case .today: return "오늘"
        case .upcoming: return "예정"
        case .overdue: return "기한 지남"
        case .highPriority: return "중요"
        }
    }
    
    var icon: String {
        switch self {
        case .today: return "calendar.day.timeline.left"
        case .upcoming: return "calendar"
        case .overdue: return "exclamationmark.triangle"
        case .highPriority: return "flag.fill"
        }
    }
}

// MARK: - Widget Bundle

@main
struct TaskWidgetBundle: WidgetBundle {
    var body: some Widget {
        TaskWidget()
        PomodoroWidget()
        TaskStatsWidget()
    }
}

struct TaskWidget: Widget {
    let kind: String = "TaskWidget"
    
    var body: some WidgetConfiguration {
        AppIntentConfiguration(
            kind: kind,
            intent: ConfigurationAppIntent.self,
            provider: TaskWidgetProvider()
        ) { entry in
            TaskWidgetView(entry: entry)
                .containerBackground(.fill.tertiary, for: .widget)
        }
        .configurationDisplayName("작업 목록")
        .description("할 일 목록을 한눈에 확인하세요")
        .supportedFamilies([
            .systemSmall,
            .systemMedium,
            .systemLarge,
            .accessoryCircular,
            .accessoryRectangular,
            .accessoryInline
        ])
    }
}
```

### 4단계: Live Activities

**LiveActivities/PomodoroActivity.swift**:

```swift
import ActivityKit
import SwiftUI
import WidgetKit

struct PomodoroAttributes: ActivityAttributes {
    public struct ContentState: Codable, Hashable {
        var timeRemaining: Int
        var isRunning: Bool
        var isBreak: Bool = false
    }
    
    var taskName: String
    var duration: Int
}

struct PomodoroActivityWidget: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: PomodoroAttributes.self) { context in
            // Lock Screen View
            PomodoroLockScreenView(context: context)
                .padding()
                .activityBackgroundTint(context.state.isBreak ? Color.green : Color.orange)
                .activitySystemActionForegroundColor(Color.white)
            
        } dynamicIsland: { context in
            DynamicIsland {
                // Expanded View
                DynamicIslandExpandedRegion(.leading) {
                    Image(systemName: context.state.isBreak ? "cup.and.saucer" : "timer")
                        .font(.title2)
                        .foregroundColor(.white)
                }
                
                DynamicIslandExpandedRegion(.center) {
                    VStack {
                        Text(context.attributes.taskName)
                            .font(.headline)
                            .lineLimit(1)
                        
                        Text(timeString(from: context.state.timeRemaining))
                            .font(.system(size: 36, weight: .bold, design: .monospaced))
                            .foregroundColor(context.state.isBreak ? .green : .orange)
                        
                        HStack(spacing: 20) {
                            Button(intent: PausePomodoroIntent()) {
                                Image(systemName: context.state.isRunning ? "pause.fill" : "play.fill")
                                    .font(.title3)
                            }
                            
                            Button(intent: SkipPomodoroIntent()) {
                                Image(systemName: "forward.end.fill")
                                    .font(.title3)
                            }
                            
                            Button(intent: StopPomodoroIntent()) {
                                Image(systemName: "stop.fill")
                                    .font(.title3)
                            }
                        }
                        .padding(.top, 8)
                    }
                }
                
                DynamicIslandExpandedRegion(.trailing) {
                    ProgressCircle(
                        progress: Double(context.attributes.duration * 60 - context.state.timeRemaining) / Double(context.attributes.duration * 60)
                    )
                    .frame(width: 50, height: 50)
                }
                
            } compactLeading: {
                Image(systemName: "timer")
                    .foregroundColor(context.state.isBreak ? .green : .orange)
            } compactTrailing: {
                Text(timeString(from: context.state.timeRemaining))
                    .font(.caption)
                    .fontDesign(.monospaced)
                    .foregroundColor(context.state.isBreak ? .green : .orange)
            } minimal: {
                Image(systemName: "timer")
                    .foregroundColor(context.state.isBreak ? .green : .orange)
            }
            .widgetURL(URL(string: "taskmaster://pomodoro"))
            .keylineTint(context.state.isBreak ? .green : .orange)
        }
    }
    
    private func timeString(from seconds: Int) -> String {
        let minutes = seconds / 60
        let remainingSeconds = seconds % 60
        return String(format: "%02d:%02d", minutes, remainingSeconds)
    }
}

struct PomodoroLockScreenView: View {
    let context: ActivityViewContext<PomodoroAttributes>
    
    var body: some View {
        VStack(spacing: 16) {
            HStack {
                VStack(alignment: .leading) {
                    Text(context.state.isBreak ? "휴식 시간" : "집중 시간")
                        .font(.caption)
                        .foregroundColor(.white.opacity(0.8))
                    
                    Text(context.attributes.taskName)
                        .font(.headline)
                        .foregroundColor(.white)
                }
                
                Spacer()
                
                // 타이머
                Text(timeString)
                    .font(.system(size: 32, weight: .bold, design: .monospaced))
                    .foregroundColor(.white)
            }
            
            // 진행률 바
            GeometryReader { geometry in
                ZStack(alignment: .leading) {
                    RoundedRectangle(cornerRadius: 4)
                        .fill(Color.white.opacity(0.2))
                        .frame(height: 8)
                    
                    RoundedRectangle(cornerRadius: 4)
                        .fill(Color.white)
                        .frame(width: geometry.size.width * progress, height: 8)
                }
            }
            .frame(height: 8)
            
            // 컨트롤 버튼
            HStack(spacing: 20) {
                Button(intent: PausePomodoroIntent()) {
                    Image(systemName: context.state.isRunning ? "pause.fill" : "play.fill")
                        .font(.title3)
                        .foregroundColor(.white)
                }
                
                Button(intent: SkipPomodoroIntent()) {
                    Image(systemName: "forward.end.fill")
                        .font(.title3)
                        .foregroundColor(.white)
                }
            }
        }
    }
    
    var timeString: String {
        let minutes = context.state.timeRemaining / 60
        let seconds = context.state.timeRemaining % 60
        return String(format: "%02d:%02d", minutes, seconds)
    }
    
    var progress: Double {
        let total = context.attributes.duration * 60
        let elapsed = total - context.state.timeRemaining
        return Double(elapsed) / Double(total)
    }
}

// MARK: - Control Intents

struct PausePomodoroIntent: AppIntent {
    static var title: LocalizedStringResource = "일시정지"
    
    func perform() async throws -> some IntentResult {
        PomodoroManager.shared.togglePause()
        return .result()
    }
}

struct SkipPomodoroIntent: AppIntent {
    static var title: LocalizedStringResource = "건너뛰기"
    
    func perform() async throws -> some IntentResult {
        PomodoroManager.shared.skip()
        return .result()
    }
}

struct StopPomodoroIntent: AppIntent {
    static var title: LocalizedStringResource = "중지"
    
    func perform() async throws -> some IntentResult {
        PomodoroManager.shared.stop()
        
        // Live Activity 종료
        for activity in Activity<PomodoroAttributes>.activities {
            await activity.end(nil, dismissalPolicy: .immediate)
        }
        
        return .result()
    }
}

// MARK: - Progress Circle

struct ProgressCircle: View {
    let progress: Double
    
    var body: some View {
        ZStack {
            Circle()
                .stroke(Color.gray.opacity(0.2), lineWidth: 4)
            
            Circle()
                .trim(from: 0, to: progress)
                .stroke(
                    progress > 0.8 ? Color.green : Color.orange,
                    style: StrokeStyle(lineWidth: 4, lineCap: .round)
                )
                .rotationEffect(.degrees(-90))
                .animation(.linear, value: progress)
            
            Text("\(Int(progress * 100))%")
                .font(.caption2)
                .fontWeight(.bold)
        }
    }
}
```

---

## 🎯 여기서 배운 것

### 1. **App Intents**
- Siri 통합
- 파라미터 처리
- 음성 피드백
- Shortcuts 앱 연동

### 2. **위젯 개발**
- Timeline Provider
- 다양한 크기 지원
- 인터랙티브 버튼
- 설정 가능한 위젯

### 3. **Live Activities**
- Dynamic Island
- 잠금 화면 표시
- 실시간 업데이트
- 푸시 알림 연동

### 4. **시스템 통합**
- URL Scheme
- Deep Linking
- Background Tasks
- Widget 업데이트

---

## 🎉 성공 확인

**Shortcuts & 위젯 체크리스트**:
- [ ] Siri로 작업 추가 가능
- [ ] 홈 화면 위젯 표시
- [ ] Dynamic Island에서 포모도로 표시
- [ ] 잠금 화면 위젯 작동
- [ ] Shortcuts 앱에서 자동화 가능

---

**완벽합니다! TaskMaster 앱이 시스템 전체와 통합되었습니다.**