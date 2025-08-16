# L1: 인터랙티브 플레이그라운드 - 손으로 만지며 배우기

## 배우는 자의 진리

> "나는 듣는 것을 잊고, 보는 것을 기억하며, 하는 것을 이해한다" - 공자

SwiftUI는 읽는 것이 아니라 만지는 것이다. 코드를 타이핑하고, 시뮬레이터에서 탭하고, 프리뷰에서 실시간으로 변화를 봐야 한다.

## 실습 1: 카운터 앱 - 상태의 핵심

### 목표: @State의 마법을 체험하기

```swift
import SwiftUI

struct CounterPlayground: View {
    @State private var count = 0
    @State private var step = 1
    @State private var isAutomatic = false
    @State private var timer: Timer?
    
    var body: some View {
        VStack(spacing: 30) {
            // 현재 카운트 표시
            Text("\(count)")
                .font(.system(size: 72, weight: .bold, design: .rounded))
                .foregroundColor(count > 0 ? .green : count < 0 ? .red : .blue)
                .scaleEffect(count == 0 ? 1.0 : 1.2)
                .animation(.spring(response: 0.3), value: count)
            
            // 증감 스텝 설정
            VStack {
                Text("증감 단위: \(step)")
                Stepper("", value: $step, in: 1...10)
                    .labelsHidden()
            }
            
            // 수동 조작 버튼들
            HStack(spacing: 20) {
                Button("-\(step)") {
                    withAnimation(.easeInOut) {
                        count -= step
                    }
                }
                .buttonStyle(.bordered)
                .disabled(isAutomatic)
                
                Button("리셋") {
                    withAnimation(.spring()) {
                        count = 0
                    }
                }
                .buttonStyle(.borderedProminent)
                .disabled(isAutomatic)
                
                Button("+\(step)") {
                    withAnimation(.easeInOut) {
                        count += step
                    }
                }
                .buttonStyle(.bordered)
                .disabled(isAutomatic)
            }
            
            // 자동 모드
            Toggle("자동 증가", isOn: $isAutomatic)
                .onChange(of: isAutomatic) { _, newValue in
                    if newValue {
                        timer = Timer.scheduledTimer(withTimeInterval: 0.5, repeats: true) { _ in
                            withAnimation(.easeInOut) {
                                count += step
                            }
                        }
                    } else {
                        timer?.invalidate()
                        timer = nil
                    }
                }
        }
        .padding()
        .onDisappear {
            timer?.invalidate()
        }
    }
}
```

### 체험해볼 것들:
1. 버튼을 탭하면서 `@State`가 어떻게 UI를 자동 업데이트하는지 관찰
2. `step` 값을 바꿔가며 증감 단위 변경
3. 자동 모드로 애니메이션과 타이머 동작 확인
4. 색상과 크기가 값에 따라 어떻게 변하는지 체험

## 실습 2: 색상 믹서 - Binding의 파워

### 목표: 부모-자식 간 데이터 흐름 이해하기

```swift
struct ColorMixer: View {
    @State private var red: Double = 0.5
    @State private var green: Double = 0.5
    @State private var blue: Double = 0.5
    @State private var opacity: Double = 1.0
    
    var currentColor: Color {
        Color(red: red, green: green, blue: blue, opacity: opacity)
    }
    
    var body: some View {
        VStack(spacing: 20) {
            // 결과 색상 미리보기
            RoundedRectangle(cornerRadius: 20)
                .fill(currentColor)
                .frame(height: 200)
                .shadow(radius: 10)
                .animation(.easeInOut, value: currentColor)
            
            // 색상 정보
            VStack(alignment: .leading) {
                Text("RGB(\(Int(red*255)), \(Int(green*255)), \(Int(blue*255)))")
                Text("투명도: \(Int(opacity*100))%")
                Text("Hex: #\(colorToHex(r: red, g: green, b: blue))")
            }
            .font(.monospaced(.body)())
            .padding()
            .background(Color.gray.opacity(0.1))
            .cornerRadius(10)
            
            // 슬라이더들
            ColorSlider(value: $red, color: .red, label: "빨강")
            ColorSlider(value: $green, color: .green, label: "초록")  
            ColorSlider(value: $blue, color: .blue, label: "파랑")
            ColorSlider(value: $opacity, color: .gray, label: "투명도")
            
            // 프리셋 버튼들
            LazyVGrid(columns: Array(repeating: GridItem(.flexible()), count: 4)) {
                ColorPreset(name: "빨강", r: 1, g: 0, b: 0, action: setColor)
                ColorPreset(name: "초록", r: 0, g: 1, b: 0, action: setColor)
                ColorPreset(name: "파랑", r: 0, g: 0, b: 1, action: setColor)
                ColorPreset(name: "노랑", r: 1, g: 1, b: 0, action: setColor)
                ColorPreset(name: "보라", r: 0.5, g: 0, b: 1, action: setColor)
                ColorPreset(name: "오렌지", r: 1, g: 0.5, b: 0, action: setColor)
                ColorPreset(name: "분홍", r: 1, g: 0.2, b: 0.8, action: setColor)
                ColorPreset(name: "무작위", r: -1, g: -1, b: -1, action: setColor)
            }
        }
        .padding()
    }
    
    func setColor(r: Double, g: Double, b: Double) {
        withAnimation(.easeInOut(duration: 0.5)) {
            if r < 0 {  // 무작위
                red = Double.random(in: 0...1)
                green = Double.random(in: 0...1)
                blue = Double.random(in: 0...1)
            } else {
                red = r
                green = g
                blue = b
            }
        }
    }
    
    func colorToHex(r: Double, g: Double, b: Double) -> String {
        let red = Int(r * 255)
        let green = Int(g * 255)
        let blue = Int(b * 255)
        return String(format: "%02X%02X%02X", red, green, blue)
    }
}

struct ColorSlider: View {
    @Binding var value: Double
    let color: Color
    let label: String
    
    var body: some View {
        VStack {
            HStack {
                Text(label)
                    .font(.headline)
                Spacer()
                Text("\(Int(value * 255))")
                    .font(.monospaced(.body)())
                    .foregroundColor(color)
            }
            
            Slider(value: $value, in: 0...1)
                .accentColor(color)
        }
    }
}

struct ColorPreset: View {
    let name: String
    let r, g, b: Double
    let action: (Double, Double, Double) -> Void
    
    var body: some View {
        Button(name) {
            action(r, g, b)
        }
        .buttonStyle(.bordered)
        .foregroundColor(r >= 0 ? Color(red: r, green: g, blue: b) : .primary)
    }
}
```

### 체험해볼 것들:
1. 슬라이더를 움직이며 실시간 색상 변화 관찰
2. `@Binding`으로 자식 뷰(ColorSlider)가 부모 상태를 어떻게 변경하는지 확인
3. 프리셋 버튼으로 애니메이션이 적용된 색상 전환 체험
4. Hex 코드가 실시간으로 계산되는 과정 관찰

## 실습 3: 할 일 관리자 - List와 ForEach 마스터

### 목표: 동적 리스트와 데이터 조작 패턴 익히기

```swift
struct TodoItem: Identifiable, Codable {
    let id = UUID()
    var title: String
    var isCompleted: Bool = false
    var priority: Priority = .medium
    var createdAt = Date()
    
    enum Priority: String, CaseIterable, Codable {
        case low = "낮음"
        case medium = "보통"
        case high = "높음"
        
        var color: Color {
            switch self {
            case .low: return .green
            case .medium: return .orange
            case .high: return .red
            }
        }
    }
}

struct TodoManager: View {
    @State private var todos: [TodoItem] = []
    @State private var newTodoTitle = ""
    @State private var selectedPriority = TodoItem.Priority.medium
    @State private var showingCompletedOnly = false
    
    var filteredTodos: [TodoItem] {
        if showingCompletedOnly {
            return todos.filter { $0.isCompleted }
        } else {
            return todos.filter { !$0.isCompleted }
        }
    }
    
    var completionStats: (completed: Int, total: Int) {
        let completed = todos.filter { $0.isCompleted }.count
        return (completed, todos.count)
    }
    
    var body: some View {
        NavigationStack {
            VStack {
                // 통계 섹션
                HStack {
                    VStack(alignment: .leading) {
                        Text("완료: \(completionStats.completed)")
                        Text("전체: \(completionStats.total)")
                    }
                    .font(.caption)
                    
                    Spacer()
                    
                    if completionStats.total > 0 {
                        CircularProgressView(
                            progress: Double(completionStats.completed) / Double(completionStats.total)
                        )
                        .frame(width: 50, height: 50)
                    }
                }
                .padding()
                .background(Color.gray.opacity(0.1))
                .cornerRadius(10)
                
                // 입력 섹션
                VStack {
                    HStack {
                        TextField("새로운 할 일", text: $newTodoTitle)
                            .textFieldStyle(.roundedBorder)
                        
                        Picker("우선순위", selection: $selectedPriority) {
                            ForEach(TodoItem.Priority.allCases, id: \.self) { priority in
                                Text(priority.rawValue)
                                    .tag(priority)
                            }
                        }
                        .pickerStyle(.menu)
                        
                        Button("추가") {
                            addTodo()
                        }
                        .disabled(newTodoTitle.trimmingCharacters(in: .whitespaces).isEmpty)
                    }
                }
                .padding()
                
                // 필터 토글
                Toggle("완료된 항목만 보기", isOn: $showingCompletedOnly)
                    .padding(.horizontal)
                
                // 할 일 리스트
                List {
                    ForEach(filteredTodos.sorted(by: sortTodos)) { todo in
                        TodoRow(todo: todo) { updatedTodo in
                            updateTodo(updatedTodo)
                        }
                    }
                    .onDelete(perform: deleteTodos)
                }
                .listStyle(.insetGrouped)
            }
            .navigationTitle("할 일 관리")
            .navigationBarTitleDisplayMode(.large)
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("전체 삭제") {
                        withAnimation {
                            todos.removeAll()
                        }
                    }
                    .disabled(todos.isEmpty)
                }
            }
        }
    }
    
    func addTodo() {
        guard !newTodoTitle.trimmingCharacters(in: .whitespaces).isEmpty else { return }
        
        withAnimation(.spring()) {
            let newTodo = TodoItem(
                title: newTodoTitle,
                priority: selectedPriority
            )
            todos.append(newTodo)
            newTodoTitle = ""
        }
    }
    
    func updateTodo(_ updatedTodo: TodoItem) {
        withAnimation {
            if let index = todos.firstIndex(where: { $0.id == updatedTodo.id }) {
                todos[index] = updatedTodo
            }
        }
    }
    
    func deleteTodos(at offsets: IndexSet) {
        withAnimation {
            let sortedTodos = filteredTodos.sorted(by: sortTodos)
            for offset in offsets {
                if let index = todos.firstIndex(where: { $0.id == sortedTodos[offset].id }) {
                    todos.remove(at: index)
                }
            }
        }
    }
    
    func sortTodos(lhs: TodoItem, rhs: TodoItem) -> Bool {
        // 우선순위 순, 그 다음 생성 시간 순
        if lhs.priority != rhs.priority {
            return priorityValue(lhs.priority) > priorityValue(rhs.priority)
        }
        return lhs.createdAt < rhs.createdAt
    }
    
    func priorityValue(_ priority: TodoItem.Priority) -> Int {
        switch priority {
        case .high: return 3
        case .medium: return 2
        case .low: return 1
        }
    }
}

struct TodoRow: View {
    let todo: TodoItem
    let onUpdate: (TodoItem) -> Void
    
    var body: some View {
        HStack {
            // 완료 체크박스
            Button {
                var updatedTodo = todo
                updatedTodo.isCompleted.toggle()
                onUpdate(updatedTodo)
            } label: {
                Image(systemName: todo.isCompleted ? "checkmark.circle.fill" : "circle")
                    .foregroundColor(todo.isCompleted ? .green : .gray)
                    .font(.title2)
            }
            .buttonStyle(.plain)
            
            VStack(alignment: .leading, spacing: 4) {
                Text(todo.title)
                    .strikethrough(todo.isCompleted)
                    .foregroundColor(todo.isCompleted ? .gray : .primary)
                
                HStack {
                    // 우선순위 표시
                    Text(todo.priority.rawValue)
                        .font(.caption)
                        .padding(.horizontal, 8)
                        .padding(.vertical, 2)
                        .background(todo.priority.color.opacity(0.2))
                        .foregroundColor(todo.priority.color)
                        .cornerRadius(4)
                    
                    Spacer()
                    
                    // 생성 시간
                    Text(todo.createdAt, style: .time)
                        .font(.caption2)
                        .foregroundColor(.gray)
                }
            }
            
            Spacer()
        }
        .padding(.vertical, 4)
    }
}

struct CircularProgressView: View {
    let progress: Double
    
    var body: some View {
        ZStack {
            Circle()
                .stroke(Color.gray.opacity(0.3), lineWidth: 4)
            
            Circle()
                .trim(from: 0, to: progress)
                .stroke(Color.blue, style: StrokeStyle(lineWidth: 4, lineCap: .round))
                .rotationEffect(.degrees(-90))
                .animation(.easeInOut, value: progress)
            
            Text("\(Int(progress * 100))%")
                .font(.caption)
                .bold()
        }
    }
}
```

### 체험해볼 것들:
1. 할 일 추가하며 List의 동적 업데이트 확인
2. 체크박스로 완료 상태 토글하며 `@Binding` 패턴 체험
3. 스와이프해서 삭제 기능으로 `onDelete` 동작 확인
4. 필터 토글로 computed property가 실시간 반영되는 과정 관찰
5. 우선순위별 정렬과 색상 코딩으로 데이터 변환 체험

## 실습 4: 제스처 인식기 - 터치의 언어

### 목표: 다양한 제스처와 애니메이션 조합 체험

```swift
struct GesturePlayground: View {
    @State private var position = CGPoint(x: 100, y: 100)
    @State private var scale: CGFloat = 1.0
    @State private var rotation: Angle = .zero
    @State private var opacity: Double = 1.0
    @State private var isDragging = false
    @State private var gestureHistory: [String] = []
    
    var body: some View {
        VStack {
            // 제스처 히스토리
            ScrollView {
                LazyVStack(alignment: .leading) {
                    ForEach(Array(gestureHistory.enumerated()), id: \.offset) { index, gesture in
                        Text("\(index + 1). \(gesture)")
                            .font(.caption)
                            .padding(.horizontal)
                    }
                }
            }
            .frame(height: 100)
            .background(Color.gray.opacity(0.1))
            .cornerRadius(10)
            .padding()
            
            Spacer()
            
            // 인터랙티브 오브젝트
            GeometryReader { geometry in
                RoundedRectangle(cornerRadius: 20)
                    .fill(
                        LinearGradient(
                            colors: [.blue, .purple],
                            startPoint: .topLeading,
                            endPoint: .bottomTrailing
                        )
                    )
                    .frame(width: 80, height: 80)
                    .scaleEffect(scale)
                    .rotationEffect(rotation)
                    .opacity(opacity)
                    .position(position)
                    .shadow(
                        color: .black.opacity(isDragging ? 0.3 : 0.1),
                        radius: isDragging ? 10 : 5,
                        y: isDragging ? 10 : 2
                    )
                    .animation(.spring(response: 0.3, dampingFraction: 0.8), value: isDragging)
                    .animation(.easeInOut, value: scale)
                    .animation(.easeInOut, value: rotation)
                    .animation(.easeInOut, value: opacity)
                    .gesture(
                        SimultaneousGesture(
                            // 드래그 제스처
                            DragGesture()
                                .onChanged { value in
                                    if !isDragging {
                                        isDragging = true
                                        addGestureHistory("드래그 시작")
                                    }
                                    position = value.location
                                }
                                .onEnded { _ in
                                    isDragging = false
                                    addGestureHistory("드래그 끝")
                                    
                                    // 화면 경계 체크하고 바운스백
                                    let bounds = geometry.frame(in: .local)
                                    if position.x < 40 || position.x > bounds.width - 40 ||
                                       position.y < 40 || position.y > bounds.height - 40 {
                                        withAnimation(.spring()) {
                                            position = CGPoint(
                                                x: min(max(position.x, 40), bounds.width - 40),
                                                y: min(max(position.y, 40), bounds.height - 40)
                                            )
                                        }
                                    }
                                },
                            
                            // 매그니파이 제스처 (핀치)
                            MagnificationGesture()
                                .onChanged { value in
                                    scale = value
                                }
                                .onEnded { value in
                                    addGestureHistory("핀치: \(String(format: "%.1f", value))배")
                                    withAnimation(.spring()) {
                                        scale = max(0.5, min(scale, 3.0))  // 제한
                                    }
                                }
                        )
                        .simultaneously(with:
                            // 회전 제스처
                            RotationGesture()
                                .onChanged { value in
                                    rotation = value
                                }
                                .onEnded { value in
                                    addGestureHistory("회전: \(Int(value.degrees))도")
                                }
                        )
                    )
                    .onTapGesture(count: 2) {
                        // 더블 탭으로 리셋
                        addGestureHistory("더블 탭 - 리셋")
                        withAnimation(.spring()) {
                            position = CGPoint(x: geometry.size.width/2, y: geometry.size.height/2)
                            scale = 1.0
                            rotation = .zero
                            opacity = 1.0
                        }
                    }
                    .onLongPressGesture(minimumDuration: 1.0) {
                        addGestureHistory("롱 프레스 - 투명도 토글")
                        withAnimation {
                            opacity = opacity > 0.5 ? 0.3 : 1.0
                        }
                    }
                    .onAppear {
                        position = CGPoint(x: geometry.size.width/2, y: geometry.size.height/2)
                    }
            }
            
            Spacer()
            
            // 컨트롤 버튼들
            HStack {
                Button("히스토리 클리어") {
                    gestureHistory.removeAll()
                }
                .buttonStyle(.bordered)
                
                Button("무작위 변형") {
                    addGestureHistory("무작위 변형 적용")
                    withAnimation(.spring()) {
                        scale = Double.random(in: 0.5...2.0)
                        rotation = .degrees(Double.random(in: 0...360))
                        opacity = Double.random(in: 0.3...1.0)
                    }
                }
                .buttonStyle(.borderedProminent)
            }
            .padding()
        }
    }
    
    func addGestureHistory(_ gesture: String) {
        let timestamp = DateFormatter.localizedString(from: Date(), dateStyle: .none, timeStyle: .medium)
        gestureHistory.append("\(timestamp) - \(gesture)")
        
        // 최대 20개까지만 유지
        if gestureHistory.count > 20 {
            gestureHistory.removeFirst()
        }
    }
}
```

### 체험해볼 것들:
1. 드래그해서 이동시키며 `DragGesture` 체험
2. 핀치로 크기 조절하며 `MagnificationGesture` 확인
3. 두 손가락으로 회전시키며 `RotationGesture` 테스트
4. 더블 탭과 롱 프레스로 다른 제스처 조합 체험
5. 애니메이션과 제스처가 자연스럽게 조합되는 과정 관찰

## 학습 체크리스트

각 실습을 완료한 후, 다음을 스스로 확인해보세요:

### ✅ 상태 관리 이해도
- [ ] `@State`가 언제 UI를 업데이트하는지 안다
- [ ] `@Binding`으로 부모-자식 간 데이터를 어떻게 공유하는지 안다
- [ ] computed property가 상태 변화에 어떻게 반응하는지 안다

### ✅ 레이아웃 시스템 이해도
- [ ] `VStack`, `HStack`, `ZStack`의 차이점을 안다
- [ ] `Spacer()`가 레이아웃에 미치는 영향을 안다
- [ ] `GeometryReader`로 공간을 어떻게 측정하는지 안다

### ✅ 애니메이션 시스템 이해도
- [ ] `withAnimation`과 `.animation` modifier의 차이를 안다
- [ ] 어떤 속성이 애니메이션 가능한지 안다
- [ ] 스프링 애니메이션의 매개변수들이 무엇을 의미하는지 안다

### ✅ 제스처 시스템 이해도
- [ ] 기본 제스처들(`tap`, `drag`, `pinch`, `rotation`)을 구현할 수 있다
- [ ] `SimultaneousGesture`로 여러 제스처를 조합할 수 있다
- [ ] 제스처와 애니메이션을 자연스럽게 연결할 수 있다

## 다음 단계를 위한 준비

L1에서 배운 내용들이 실제 앱에서 어떻게 조합되는지 볼 준비가 되었나요?

이제 다음 레벨로 가서 완전한 앱을 만들어봅시다:

→ [L2: 첫 앱 만들기 - 배달 주문 앱](../L2_implementation/00_first_app.md)

---

*"I can't understand anything in general unless I'm carrying along in my mind a specific example and watching it go." - Richard Feynman*

*"일반적인 것은 구체적인 예시를 머릿속에 담고 그것이 어떻게 동작하는지 지켜봐야만 이해할 수 있다." - 리처드 파인만*