# L2-02: 스마트 작업 관리 시스템
## AI 기반 자연어 입력, 스마트 제안, 포모도로 타이머

---

> **"Great products are built on delightful interactions."**

자연어 처리와 머신러닝을 활용한 지능형 작업 관리 시스템을 구현합니다.

---

## 🎯 목표

**완성 후 결과물**:
- 자연어 작업 입력
- AI 기반 우선순위 제안
- 스마트 알림 시스템
- 포모도로 타이머 통합
- 간트 차트 시각화

---

## 🚀 작업 관리 뷰 구현

### 1단계: 메인 작업 목록 뷰

**Presentation/Views/Tasks/TaskListView.swift**:

```swift
import SwiftUI
import Charts

struct TaskListView: View {
    @StateObject private var viewModel = TaskViewModel()
    @StateObject private var nlpProcessor = NaturalLanguageProcessor()
    @State private var showingAddTask = false
    @State private var naturalLanguageInput = ""
    @State private var selectedFilter: TaskFilter = .all
    @State private var searchText = ""
    @State private var showingPomodoro = false
    @State private var selectedTask: Task?
    @State private var showingGanttChart = false
    
    // 음성 인식
    @StateObject private var speechRecognizer = SpeechRecognizer()
    @State private var isRecording = false
    
    var body: some View {
        NavigationStack {
            ZStack {
                VStack(spacing: 0) {
                    // 스마트 제안 헤더
                    if !viewModel.smartSuggestions.isEmpty {
                        SmartSuggestionsHeader(suggestions: viewModel.smartSuggestions) { suggestion in
                            handleSuggestion(suggestion)
                        }
                        .padding(.horizontal)
                        .padding(.top, 8)
                    }
                    
                    // 필터 및 검색
                    HStack {
                        FilterChips(selectedFilter: $selectedFilter)
                        
                        Spacer()
                        
                        // 간트 차트 버튼
                        Button {
                            showingGanttChart = true
                        } label: {
                            Image(systemName: "chart.bar.xaxis")
                                .foregroundColor(.blue)
                        }
                    }
                    .padding(.horizontal)
                    .padding(.vertical, 8)
                    
                    // 작업 목록
                    if viewModel.isLoading {
                        ProgressView("작업을 불러오는 중...")
                            .frame(maxWidth: .infinity, maxHeight: .infinity)
                    } else if filteredTasks.isEmpty {
                        EmptyStateView(filter: selectedFilter)
                            .frame(maxWidth: .infinity, maxHeight: .infinity)
                    } else {
                        TaskListContent(
                            tasks: filteredTasks,
                            onToggleComplete: { task in
                                Task {
                                    await viewModel.toggleTaskCompletion(task)
                                }
                            },
                            onDelete: { task in
                                Task {
                                    await viewModel.deleteTask(task)
                                }
                            },
                            onStartPomodoro: { task in
                                selectedTask = task
                                showingPomodoro = true
                            }
                        )
                    }
                }
                
                // 플로팅 추가 버튼
                VStack {
                    Spacer()
                    HStack {
                        Spacer()
                        
                        HStack(spacing: 16) {
                            // 음성 입력 버튼
                            Button {
                                toggleVoiceRecording()
                            } label: {
                                Image(systemName: isRecording ? "mic.fill" : "mic")
                                    .font(.title2)
                                    .foregroundColor(.white)
                                    .frame(width: 56, height: 56)
                                    .background(isRecording ? Color.red : Color.blue)
                                    .clipShape(Circle())
                                    .shadow(radius: 4)
                            }
                            .scaleEffect(isRecording ? 1.1 : 1.0)
                            .animation(.easeInOut(duration: 0.5).repeatForever(autoreverses: true), value: isRecording)
                            
                            // 텍스트 추가 버튼
                            Button {
                                showingAddTask = true
                            } label: {
                                Image(systemName: "plus")
                                    .font(.title2)
                                    .foregroundColor(.white)
                                    .frame(width: 56, height: 56)
                                    .background(Color.blue)
                                    .clipShape(Circle())
                                    .shadow(radius: 4)
                            }
                        }
                        .padding()
                    }
                }
            }
            .navigationTitle("작업")
            .searchable(text: $searchText, prompt: "작업 검색...")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Menu {
                        Section("정렬") {
                            Button("마감일순") {
                                viewModel.sortBy = .dueDate
                            }
                            Button("우선순위순") {
                                viewModel.sortBy = .priority
                            }
                            Button("생성일순") {
                                viewModel.sortBy = .createdAt
                            }
                        }
                        
                        Section("보기") {
                            Button("완료된 작업 표시") {
                                viewModel.showCompleted.toggle()
                            }
                            Button("보관된 프로젝트 표시") {
                                viewModel.showArchived.toggle()
                            }
                        }
                    } label: {
                        Image(systemName: "ellipsis.circle")
                    }
                }
            }
            .sheet(isPresented: $showingAddTask) {
                AddTaskView(viewModel: viewModel)
            }
            .sheet(isPresented: $showingPomodoro) {
                if let task = selectedTask {
                    PomodoroTimerView(task: task)
                }
            }
            .sheet(isPresented: $showingGanttChart) {
                GanttChartView(tasks: viewModel.tasks)
            }
            .onChange(of: speechRecognizer.transcript) { newValue in
                if !newValue.isEmpty {
                    processNaturalLanguageInput(newValue)
                }
            }
        }
        .task {
            await viewModel.loadTasks()
            await viewModel.loadSmartSuggestions()
        }
    }
    
    // MARK: - Computed Properties
    
    var filteredTasks: [Task] {
        var tasks = viewModel.tasks
        
        // 필터 적용
        switch selectedFilter {
        case .all:
            break
        case .today:
            tasks = tasks.filter { $0.isDueToday }
        case .upcoming:
            tasks = tasks.filter { task in
                guard let dueDate = task.dueDate else { return false }
                return dueDate > Date() && !task.isDueToday
            }
        case .overdue:
            tasks = tasks.filter { $0.isOverdue }
        case .completed:
            tasks = tasks.filter { $0.isCompleted }
        case .highPriority:
            tasks = tasks.filter { $0.priority == .high || $0.priority == .urgent }
        }
        
        // 검색 적용
        if !searchText.isEmpty {
            tasks = tasks.filter { task in
                task.title.localizedCaseInsensitiveContains(searchText) ||
                (task.description?.localizedCaseInsensitiveContains(searchText) ?? false)
            }
        }
        
        return tasks
    }
    
    // MARK: - Methods
    
    private func toggleVoiceRecording() {
        if isRecording {
            speechRecognizer.stopRecording()
        } else {
            speechRecognizer.startRecording()
        }
        isRecording.toggle()
    }
    
    private func processNaturalLanguageInput(_ input: String) {
        Task {
            let parsedTask = await nlpProcessor.parseTaskFromNaturalLanguage(input)
            await viewModel.createTask(from: parsedTask)
            
            // 음성 피드백
            let utterance = "작업 '\(parsedTask.title)'을(를) 추가했습니다."
            SpeechSynthesizer.shared.speak(utterance)
        }
    }
    
    private func handleSuggestion(_ suggestion: SmartSuggestion) {
        switch suggestion.action {
        case .showTodayTasks:
            selectedFilter = .today
        case .showOverdueTasks:
            selectedFilter = .overdue
        case .createTask(let title):
            naturalLanguageInput = title
            showingAddTask = true
        case .showProject(let projectId):
            // Navigate to project
            break
        }
    }
}

// MARK: - Supporting Views

struct FilterChips: View {
    @Binding var selectedFilter: TaskFilter
    
    var body: some View {
        ScrollView(.horizontal, showsIndicators: false) {
            HStack(spacing: 8) {
                ForEach(TaskFilter.allCases, id: \.self) { filter in
                    FilterChip(
                        title: filter.title,
                        icon: filter.icon,
                        isSelected: selectedFilter == filter
                    ) {
                        withAnimation(.spring(response: 0.3)) {
                            selectedFilter = filter
                        }
                    }
                }
            }
        }
    }
}

struct FilterChip: View {
    let title: String
    let icon: String
    let isSelected: Bool
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            Label(title, systemImage: icon)
                .font(.caption)
                .padding(.horizontal, 12)
                .padding(.vertical, 6)
                .background(isSelected ? Color.blue : Color.gray.opacity(0.2))
                .foregroundColor(isSelected ? .white : .primary)
                .clipShape(Capsule())
        }
    }
}

struct SmartSuggestionsHeader: View {
    let suggestions: [SmartSuggestion]
    let onSelect: (SmartSuggestion) -> Void
    
    var body: some View {
        ScrollView(.horizontal, showsIndicators: false) {
            HStack(spacing: 12) {
                ForEach(suggestions, id: \.title) { suggestion in
                    SmartSuggestionCard(suggestion: suggestion) {
                        onSelect(suggestion)
                    }
                }
            }
        }
        .padding(.vertical, 8)
    }
}

struct SmartSuggestionCard: View {
    let suggestion: SmartSuggestion
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            VStack(alignment: .leading, spacing: 4) {
                Text(suggestion.title)
                    .font(.caption)
                    .fontWeight(.semibold)
                Text(suggestion.description)
                    .font(.caption2)
                    .foregroundColor(.secondary)
            }
            .padding(8)
            .frame(width: 150)
            .background(Color.blue.opacity(0.1))
            .cornerRadius(8)
        }
        .buttonStyle(PlainButtonStyle())
    }
}

struct TaskListContent: View {
    let tasks: [Task]
    let onToggleComplete: (Task) -> Void
    let onDelete: (Task) -> Void
    let onStartPomodoro: (Task) -> Void
    
    var body: some View {
        List {
            ForEach(tasks) { task in
                TaskRowView(
                    task: task,
                    onToggleComplete: { onToggleComplete(task) },
                    onStartPomodoro: { onStartPomodoro(task) }
                )
                .swipeActions(edge: .trailing, allowsFullSwipe: true) {
                    Button(role: .destructive) {
                        onDelete(task)
                    } label: {
                        Label("삭제", systemImage: "trash")
                    }
                }
            }
        }
        .listStyle(PlainListStyle())
    }
}

// MARK: - Task Filter Enum

enum TaskFilter: CaseIterable {
    case all
    case today
    case upcoming
    case overdue
    case completed
    case highPriority
    
    var title: String {
        switch self {
        case .all: return "전체"
        case .today: return "오늘"
        case .upcoming: return "예정"
        case .overdue: return "기한 지남"
        case .completed: return "완료"
        case .highPriority: return "중요"
        }
    }
    
    var icon: String {
        switch self {
        case .all: return "tray"
        case .today: return "calendar.day.timeline.left"
        case .upcoming: return "calendar"
        case .overdue: return "exclamationmark.triangle"
        case .completed: return "checkmark.circle"
        case .highPriority: return "flag.fill"
        }
    }
}
```

### 2단계: 자연어 처리 시스템

**Services/NaturalLanguageProcessor.swift**:

```swift
import Foundation
import NaturalLanguage
import CoreML

@MainActor
class NaturalLanguageProcessor: ObservableObject {
    private let tokenizer = NLTokenizer(unit: .word)
    private let tagger = NLTagger(tagSchemes: [.lexicalClass, .nameType])
    private var priorityClassifier: NLModel?
    
    init() {
        loadMLModels()
    }
    
    private func loadMLModels() {
        // Priority 분류 모델 로드
        if let modelURL = Bundle.main.url(forResource: "TaskPriorityClassifier", withExtension: "mlmodelc"),
           let model = try? NLModel(contentsOf: modelURL) {
            priorityClassifier = model
        }
    }
    
    func parseTaskFromNaturalLanguage(_ input: String) async -> ParsedTask {
        var parsedTask = ParsedTask()
        
        // 기본 제목 추출
        parsedTask.title = extractTitle(from: input)
        
        // 날짜 추출
        if let dueDate = extractDate(from: input) {
            parsedTask.dueDate = dueDate
        }
        
        // 우선순위 추출
        parsedTask.priority = extractPriority(from: input)
        
        // 프로젝트 추출
        parsedTask.projectName = extractProject(from: input)
        
        // 태그 추출
        parsedTask.tags = extractTags(from: input)
        
        // 반복 규칙 추출
        parsedTask.recurrence = extractRecurrence(from: input)
        
        // 위치 추출
        parsedTask.location = extractLocation(from: input)
        
        // AI 기반 카테고리 제안
        parsedTask.suggestedCategory = await suggestCategory(for: input)
        
        return parsedTask
    }
    
    private func extractTitle(from input: String) -> String {
        // 날짜, 시간, 우선순위 키워드 제거
        let keywords = ["내일", "오늘", "다음주", "까지", "중요", "긴급", "#", "@", "에서", "반복"]
        var title = input
        
        for keyword in keywords {
            title = title.replacingOccurrences(of: keyword, with: "")
        }
        
        // 불필요한 공백 제거
        title = title.trimmingCharacters(in: .whitespacesAndNewlines)
        title = title.replacingOccurrences(of: "  ", with: " ")
        
        return title.isEmpty ? input : title
    }
    
    private func extractDate(from input: String) -> Date? {
        let detector = try? NSDataDetector(types: NSTextCheckingResult.CheckingType.date.rawValue)
        let matches = detector?.matches(in: input, options: [], range: NSRange(location: 0, length: input.utf16.count))
        
        if let match = matches?.first, let date = match.date {
            return date
        }
        
        // 커스텀 날짜 파싱
        let calendar = Calendar.current
        let now = Date()
        
        if input.contains("오늘") {
            return calendar.startOfDay(for: now)
        } else if input.contains("내일") {
            return calendar.date(byAdding: .day, value: 1, to: now)
        } else if input.contains("모레") {
            return calendar.date(byAdding: .day, value: 2, to: now)
        } else if input.contains("이번주") {
            return calendar.dateInterval(of: .weekOfYear, for: now)?.end
        } else if input.contains("다음주") {
            return calendar.date(byAdding: .weekOfYear, value: 1, to: now)
        } else if input.contains("이번달") {
            return calendar.dateInterval(of: .month, for: now)?.end
        } else if input.contains("다음달") {
            return calendar.date(byAdding: .month, value: 1, to: now)
        }
        
        // 시간 파싱
        let timeRegex = try? NSRegularExpression(pattern: "(\\d{1,2})시", options: [])
        if let match = timeRegex?.firstMatch(in: input, options: [], range: NSRange(location: 0, length: input.utf16.count)) {
            if let hourRange = Range(match.range(at: 1), in: input),
               let hour = Int(input[hourRange]) {
                var components = calendar.dateComponents([.year, .month, .day], from: now)
                components.hour = hour
                return calendar.date(from: components)
            }
        }
        
        return nil
    }
    
    private func extractPriority(from input: String) -> Priority {
        let urgentKeywords = ["긴급", "급함", "즉시", "바로", "당장", "ASAP"]
        let highKeywords = ["중요", "높음", "필수", "꼭"]
        let lowKeywords = ["나중에", "여유있게", "천천히", "낮음"]
        
        if urgentKeywords.contains(where: input.contains) {
            return .urgent
        } else if highKeywords.contains(where: input.contains) {
            return .high
        } else if lowKeywords.contains(where: input.contains) {
            return .low
        }
        
        // ML 모델 사용
        if let classifier = priorityClassifier {
            let prediction = classifier.predictedLabel(for: input)
            switch prediction {
            case "urgent": return .urgent
            case "high": return .high
            case "low": return .low
            default: return .medium
            }
        }
        
        return .medium
    }
    
    private func extractProject(from input: String) -> String? {
        // @ 기호로 프로젝트 표시
        let regex = try? NSRegularExpression(pattern: "@(\\w+)", options: [])
        if let match = regex?.firstMatch(in: input, options: [], range: NSRange(location: 0, length: input.utf16.count)) {
            if let projectRange = Range(match.range(at: 1), in: input) {
                return String(input[projectRange])
            }
        }
        
        return nil
    }
    
    private func extractTags(from input: String) -> [String] {
        var tags: [String] = []
        
        // # 기호로 태그 표시
        let regex = try? NSRegularExpression(pattern: "#(\\w+)", options: [])
        let matches = regex?.matches(in: input, options: [], range: NSRange(location: 0, length: input.utf16.count)) ?? []
        
        for match in matches {
            if let tagRange = Range(match.range(at: 1), in: input) {
                tags.append(String(input[tagRange]))
            }
        }
        
        return tags
    }
    
    private func extractRecurrence(from input: String) -> RecurrenceRule? {
        if input.contains("매일") {
            return RecurrenceRule(frequency: .daily, interval: 1, endDate: nil, daysOfWeek: nil, dayOfMonth: nil)
        } else if input.contains("매주") {
            return RecurrenceRule(frequency: .weekly, interval: 1, endDate: nil, daysOfWeek: nil, dayOfMonth: nil)
        } else if input.contains("매월") {
            return RecurrenceRule(frequency: .monthly, interval: 1, endDate: nil, daysOfWeek: nil, dayOfMonth: nil)
        } else if input.contains("매년") {
            return RecurrenceRule(frequency: .yearly, interval: 1, endDate: nil, daysOfWeek: nil, dayOfMonth: nil)
        }
        
        return nil
    }
    
    private func extractLocation(from input: String) -> String? {
        // "에서" 키워드로 위치 추출
        if let range = input.range(of: "에서") {
            let beforeLocation = input[..<range.lowerBound]
            let words = beforeLocation.split(separator: " ")
            if let lastWord = words.last {
                return String(lastWord)
            }
        }
        
        // 장소 관련 키워드
        let placeKeywords = ["회의실", "사무실", "집", "카페", "스타벅스", "학교", "회사"]
        for keyword in placeKeywords {
            if input.contains(keyword) {
                return keyword
            }
        }
        
        return nil
    }
    
    private func suggestCategory(for input: String) async -> String {
        // 간단한 카테고리 분류
        let workKeywords = ["회의", "미팅", "프레젠테이션", "보고서", "이메일", "전화"]
        let personalKeywords = ["운동", "병원", "약속", "생일", "쇼핑", "영화"]
        let studyKeywords = ["공부", "강의", "숙제", "시험", "과제", "독서"]
        
        if workKeywords.contains(where: input.contains) {
            return "업무"
        } else if personalKeywords.contains(where: input.contains) {
            return "개인"
        } else if studyKeywords.contains(where: input.contains) {
            return "학습"
        }
        
        return "기타"
    }
}

// MARK: - Parsed Task Model

struct ParsedTask {
    var title: String = ""
    var description: String?
    var dueDate: Date?
    var priority: Priority = .medium
    var projectName: String?
    var tags: [String] = []
    var recurrence: RecurrenceRule?
    var location: String?
    var suggestedCategory: String?
}
```

### 3단계: 포모도로 타이머

**Presentation/Views/Tasks/PomodoroTimerView.swift**:

```swift
import SwiftUI
import UserNotifications
import AVFoundation

struct PomodoroTimerView: View {
    let task: Task
    @StateObject private var timer = PomodoroTimer()
    @State private var showingSettings = false
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        NavigationStack {
            ZStack {
                // 배경 그라데이션
                LinearGradient(
                    colors: timer.isBreak ? [.green.opacity(0.3), .blue.opacity(0.3)] : [.red.opacity(0.3), .orange.opacity(0.3)],
                    startPoint: .topLeading,
                    endPoint: .bottomTrailing
                )
                .ignoresSafeArea()
                
                VStack(spacing: 40) {
                    // 작업 정보
                    VStack(spacing: 8) {
                        Text(task.title)
                            .font(.title2)
                            .fontWeight(.bold)
                        
                        if timer.isBreak {
                            Text("휴식 시간")
                                .font(.headline)
                                .foregroundColor(.green)
                        } else {
                            Text("집중 시간")
                                .font(.headline)
                                .foregroundColor(.orange)
                        }
                        
                        Text("세션 \(timer.currentSession)/\(timer.totalSessions)")
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }
                    
                    // 타이머 디스플레이
                    ZStack {
                        // 진행률 링
                        Circle()
                            .stroke(Color.gray.opacity(0.2), lineWidth: 20)
                            .frame(width: 250, height: 250)
                        
                        Circle()
                            .trim(from: 0, to: timer.progress)
                            .stroke(
                                timer.isBreak ? Color.green : Color.orange,
                                style: StrokeStyle(lineWidth: 20, lineCap: .round)
                            )
                            .frame(width: 250, height: 250)
                            .rotationEffect(.degrees(-90))
                            .animation(.linear(duration: 1), value: timer.progress)
                        
                        // 시간 표시
                        VStack {
                            Text(timer.timeString)
                                .font(.system(size: 60, weight: .bold, design: .monospaced))
                            
                            if timer.isPaused && !timer.isRunning {
                                Text("일시정지")
                                    .font(.caption)
                                    .foregroundColor(.secondary)
                            }
                        }
                    }
                    
                    // 컨트롤 버튼
                    HStack(spacing: 40) {
                        // 리셋 버튼
                        Button {
                            timer.reset()
                        } label: {
                            Image(systemName: "arrow.counterclockwise")
                                .font(.title2)
                                .foregroundColor(.gray)
                                .frame(width: 60, height: 60)
                                .background(Color.gray.opacity(0.2))
                                .clipShape(Circle())
                        }
                        .disabled(timer.timeRemaining == timer.isBreak ? timer.breakDuration : timer.workDuration)
                        
                        // 시작/일시정지 버튼
                        Button {
                            if timer.isRunning {
                                timer.pause()
                            } else {
                                timer.start()
                            }
                        } label: {
                            Image(systemName: timer.isRunning ? "pause.fill" : "play.fill")
                                .font(.title)
                                .foregroundColor(.white)
                                .frame(width: 80, height: 80)
                                .background(timer.isBreak ? Color.green : Color.orange)
                                .clipShape(Circle())
                                .shadow(radius: 5)
                        }
                        
                        // 스킵 버튼
                        Button {
                            timer.skip()
                        } label: {
                            Image(systemName: "forward.end")
                                .font(.title2)
                                .foregroundColor(.gray)
                                .frame(width: 60, height: 60)
                                .background(Color.gray.opacity(0.2))
                                .clipShape(Circle())
                        }
                    }
                    
                    // 통계
                    HStack(spacing: 30) {
                        StatView(title: "오늘 완료", value: "\(timer.todayCompletedSessions)", icon: "checkmark.circle")
                        StatView(title: "총 시간", value: timer.totalTimeString, icon: "clock")
                        StatView(title: "집중도", value: "\(Int(timer.focusScore))%", icon: "brain")
                    }
                    .padding(.horizontal)
                    
                    Spacer()
                }
                .padding()
            }
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .navigationBarLeading) {
                    Button("닫기") {
                        dismiss()
                    }
                }
                
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button {
                        showingSettings = true
                    } label: {
                        Image(systemName: "gearshape")
                    }
                }
            }
            .sheet(isPresented: $showingSettings) {
                PomodoroSettingsView(timer: timer)
            }
            .onReceive(timer.$isSessionComplete) { complete in
                if complete {
                    handleSessionComplete()
                }
            }
        }
    }
    
    private func handleSessionComplete() {
        // 햅틱 피드백
        let impactFeedback = UIImpactFeedbackGenerator(style: .heavy)
        impactFeedback.impactOccurred()
        
        // 사운드 재생
        AudioServicesPlaySystemSound(1005)
        
        // 알림 전송
        if timer.isBreak {
            sendNotification(title: "휴식 시간 종료", body: "다시 집중할 시간입니다!")
        } else {
            sendNotification(title: "집중 시간 종료", body: "잠시 휴식을 취하세요!")
        }
    }
    
    private func sendNotification(title: String, body: String) {
        let content = UNMutableNotificationContent()
        content.title = title
        content.body = body
        content.sound = .default
        
        let request = UNNotificationRequest(
            identifier: UUID().uuidString,
            content: content,
            trigger: nil
        )
        
        UNUserNotificationCenter.current().add(request)
    }
}

struct StatView: View {
    let title: String
    let value: String
    let icon: String
    
    var body: some View {
        VStack(spacing: 8) {
            Image(systemName: icon)
                .font(.title2)
                .foregroundColor(.secondary)
            
            Text(value)
                .font(.headline)
                .fontWeight(.bold)
            
            Text(title)
                .font(.caption)
                .foregroundColor(.secondary)
        }
    }
}

// MARK: - Pomodoro Timer Model

@MainActor
class PomodoroTimer: ObservableObject {
    @Published var workDuration: TimeInterval = 25 * 60 // 25분
    @Published var breakDuration: TimeInterval = 5 * 60 // 5분
    @Published var longBreakDuration: TimeInterval = 15 * 60 // 15분
    @Published var totalSessions = 4
    
    @Published var currentSession = 1
    @Published var timeRemaining: TimeInterval = 25 * 60
    @Published var isRunning = false
    @Published var isPaused = false
    @Published var isBreak = false
    @Published var isSessionComplete = false
    
    @Published var todayCompletedSessions = 0
    @Published var totalWorkTime: TimeInterval = 0
    @Published var focusScore: Double = 85.0
    
    private var timer: Timer?
    
    var progress: Double {
        let total = isBreak ? 
            (currentSession == totalSessions ? longBreakDuration : breakDuration) : 
            workDuration
        return 1 - (timeRemaining / total)
    }
    
    var timeString: String {
        let minutes = Int(timeRemaining) / 60
        let seconds = Int(timeRemaining) % 60
        return String(format: "%02d:%02d", minutes, seconds)
    }
    
    var totalTimeString: String {
        let hours = Int(totalWorkTime) / 3600
        let minutes = (Int(totalWorkTime) % 3600) / 60
        return String(format: "%dh %dm", hours, minutes)
    }
    
    func start() {
        isRunning = true
        isPaused = false
        isSessionComplete = false
        
        timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
            self.tick()
        }
    }
    
    func pause() {
        isRunning = false
        isPaused = true
        timer?.invalidate()
        timer = nil
    }
    
    func reset() {
        timer?.invalidate()
        timer = nil
        isRunning = false
        isPaused = false
        timeRemaining = isBreak ? breakDuration : workDuration
    }
    
    func skip() {
        timer?.invalidate()
        timer = nil
        isRunning = false
        
        if isBreak {
            // 휴식 스킵 -> 다음 작업 세션
            isBreak = false
            timeRemaining = workDuration
            if currentSession < totalSessions {
                currentSession += 1
            }
        } else {
            // 작업 스킵 -> 휴식
            isBreak = true
            timeRemaining = currentSession == totalSessions ? longBreakDuration : breakDuration
        }
    }
    
    private func tick() {
        if timeRemaining > 0 {
            timeRemaining -= 1
            
            if !isBreak {
                totalWorkTime += 1
            }
        } else {
            // 세션 완료
            handleSessionComplete()
        }
    }
    
    private func handleSessionComplete() {
        isSessionComplete = true
        timer?.invalidate()
        timer = nil
        isRunning = false
        
        if !isBreak {
            todayCompletedSessions += 1
            
            // 휴식 시간으로 전환
            isBreak = true
            timeRemaining = currentSession == totalSessions ? longBreakDuration : breakDuration
        } else {
            // 다음 작업 세션으로 전환
            isBreak = false
            timeRemaining = workDuration
            
            if currentSession >= totalSessions {
                // 전체 사이클 완료
                currentSession = 1
            } else {
                currentSession += 1
            }
        }
        
        // 자동 시작 옵션
        if UserDefaults.standard.bool(forKey: "autoStartPomodoro") {
            DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
                self.start()
            }
        }
    }
}
```

### 4단계: 간트 차트 시각화

**Presentation/Views/Tasks/GanttChartView.swift**:

```swift
import SwiftUI
import Charts

struct GanttChartView: View {
    let tasks: [Task]
    @State private var selectedTimeRange: TimeRange = .week
    @State private var scrollPosition: Date = Date()
    @Environment(\.dismiss) private var dismiss
    
    var filteredTasks: [Task] {
        tasks.filter { task in
            guard let dueDate = task.dueDate else { return false }
            
            let calendar = Calendar.current
            switch selectedTimeRange {
            case .week:
                return calendar.isDate(dueDate, equalTo: Date(), toGranularity: .weekOfYear)
            case .month:
                return calendar.isDate(dueDate, equalTo: Date(), toGranularity: .month)
            case .quarter:
                let quarterStart = calendar.date(from: calendar.dateComponents([.year], from: Date()))!
                let quarterEnd = calendar.date(byAdding: .month, value: 3, to: quarterStart)!
                return dueDate >= quarterStart && dueDate <= quarterEnd
            case .year:
                return calendar.isDate(dueDate, equalTo: Date(), toGranularity: .year)
            }
        }
        .sorted { ($0.dueDate ?? .distantFuture) < ($1.dueDate ?? .distantFuture) }
    }
    
    var body: some View {
        NavigationStack {
            VStack {
                // 시간 범위 선택
                Picker("시간 범위", selection: $selectedTimeRange) {
                    ForEach(TimeRange.allCases, id: \.self) { range in
                        Text(range.title).tag(range)
                    }
                }
                .pickerStyle(SegmentedPickerStyle())
                .padding()
                
                // 간트 차트
                ScrollView([.horizontal, .vertical]) {
                    GanttChart(tasks: filteredTasks, timeRange: selectedTimeRange)
                        .frame(width: max(CGFloat(filteredTasks.count * 100), 800), 
                               height: max(CGFloat(filteredTasks.count * 50), 400))
                }
                
                // 범례
                HStack(spacing: 20) {
                    LegendItem(color: .red, title: "긴급")
                    LegendItem(color: .orange, title: "높음")
                    LegendItem(color: .blue, title: "보통")
                    LegendItem(color: .gray, title: "낮음")
                }
                .padding()
            }
            .navigationTitle("간트 차트")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .navigationBarLeading) {
                    Button("닫기") {
                        dismiss()
                    }
                }
                
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button {
                        exportChart()
                    } label: {
                        Image(systemName: "square.and.arrow.up")
                    }
                }
            }
        }
    }
    
    private func exportChart() {
        // 차트를 이미지로 내보내기
        let renderer = ImageRenderer(content: GanttChart(tasks: filteredTasks, timeRange: selectedTimeRange))
        renderer.scale = 2.0
        
        if let image = renderer.uiImage {
            let activityController = UIActivityViewController(
                activityItems: [image],
                applicationActivities: nil
            )
            
            if let windowScene = UIApplication.shared.connectedScenes.first as? UIWindowScene,
               let window = windowScene.windows.first,
               let rootViewController = window.rootViewController {
                rootViewController.present(activityController, animated: true)
            }
        }
    }
}

struct GanttChart: View {
    let tasks: [Task]
    let timeRange: TimeRange
    
    var body: some View {
        Chart {
            ForEach(tasks) { task in
                if let startDate = task.createdAt,
                   let endDate = task.dueDate {
                    RectangleMark(
                        xStart: .value("시작", startDate),
                        xEnd: .value("종료", endDate),
                        y: .value("작업", task.title)
                    )
                    .foregroundStyle(by: .value("우선순위", task.priority.title))
                    .annotation(position: .overlay) {
                        Text(task.title)
                            .font(.caption)
                            .foregroundColor(.white)
                            .lineLimit(1)
                    }
                }
            }
        }
        .chartForegroundStyleScale([
            "긴급": Color.red,
            "높음": Color.orange,
            "보통": Color.blue,
            "낮음": Color.gray
        ])
        .chartXAxis {
            AxisMarks(values: .stride(by: timeRange.strideComponent)) { value in
                AxisGridLine()
                AxisValueLabel(format: timeRange.dateFormat)
            }
        }
        .chartYAxis {
            AxisMarks { value in
                AxisValueLabel()
            }
        }
        .padding()
    }
}

struct LegendItem: View {
    let color: Color
    let title: String
    
    var body: some View {
        HStack(spacing: 4) {
            Circle()
                .fill(color)
                .frame(width: 12, height: 12)
            Text(title)
                .font(.caption)
        }
    }
}

enum TimeRange: CaseIterable {
    case week
    case month
    case quarter
    case year
    
    var title: String {
        switch self {
        case .week: return "주"
        case .month: return "월"
        case .quarter: return "분기"
        case .year: return "년"
        }
    }
    
    var strideComponent: Calendar.Component {
        switch self {
        case .week: return .day
        case .month: return .weekOfMonth
        case .quarter: return .month
        case .year: return .month
        }
    }
    
    var dateFormat: Date.FormatStyle {
        switch self {
        case .week: return .dateTime.day().month()
        case .month: return .dateTime.day()
        case .quarter: return .dateTime.month(.abbreviated)
        case .year: return .dateTime.month(.abbreviated)
        }
    }
}
```

---

## 🎯 여기서 배운 것

### 1. **자연어 처리**
- NaturalLanguage 프레임워크
- 날짜/시간 추출
- 의도 파악
- ML 모델 통합

### 2. **포모도로 기법**
- 타이머 구현
- 백그라운드 작업
- 알림 시스템
- 통계 추적

### 3. **데이터 시각화**
- Charts 프레임워크
- 간트 차트
- 인터랙티브 그래프
- 데이터 내보내기

### 4. **음성 인터페이스**
- Speech 프레임워크
- 실시간 음성 인식
- 음성 합성
- 핸즈프리 작업

---

## 🎉 성공 확인

**작업 관리 체크리스트**:
- [ ] 자연어로 작업 추가 가능
- [ ] AI가 우선순위 제안
- [ ] 포모도로 타이머 작동
- [ ] 간트 차트로 일정 확인
- [ ] 음성 명령 지원

---

**완벽합니다! 스마트한 작업 관리 시스템이 완성되었습니다.**