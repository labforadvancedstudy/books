# L1: SwiftUI 기초 - 선언하면 나타난다

## 첫 번째 진실: UI는 상태의 함수다

```swift
UI = f(State)
```

이 한 줄이 SwiftUI의 모든 것이다.

전통적인 방식 (UIKit):
```swift
// 명령형: "어떻게" 그릴지 설명
let label = UILabel()
label.text = "Hello"
label.textColor = .black
view.addSubview(label)

// 상태가 바뀌면?
label.text = "World"  // 직접 변경
```

SwiftUI 방식:
```swift
// 선언형: "무엇"을 그릴지 선언
struct ContentView: View {
    @State var message = "Hello"
    
    var body: some View {
        Text(message)
            .foregroundColor(.black)
    }
}

// 상태가 바뀌면?
message = "World"  // 자동으로 UI 업데이트
```

## View: 모든 것의 시작

```swift
protocol View {
    associatedtype Body: View
    var body: Self.Body { get }
}
```

View는 프로토콜이다. 약속이다. "나는 그려질 수 있다"는 선언이다.

### 가장 단순한 View

```swift
struct MyView: View {
    var body: some View {
        Text("나는 존재한다")
    }
}
```

이것이 전부다. 복잡한 설정 없이, 이것만으로 화면에 나타난다.

## 기본 컴포넌트들

### Text - 생각을 표현하다
```swift
Text("Hello, World!")
    .font(.title)
    .foregroundColor(.blue)
    .bold()
    .italic()
    .underline()
```

### Image - 천 마디 말
```swift
Image(systemName: "star.fill")
    .resizable()
    .frame(width: 50, height: 50)
    .foregroundColor(.yellow)

Image("myPhoto")
    .resizable()
    .aspectRatio(contentMode: .fit)
```

### Button - 의도의 시작
```swift
Button("탭 해주세요") {
    print("탭 되었습니다!")
}

// 더 복잡한 버튼
Button(action: {
    // 액션
}) {
    HStack {
        Image(systemName: "hand.tap")
        Text("커스텀 버튼")
    }
}
.buttonStyle(.borderedProminent)
```

### TextField - 생각을 입력하다
```swift
struct FormView: View {
    @State private var name = ""
    
    var body: some View {
        TextField("이름을 입력하세요", text: $name)
            .textFieldStyle(.roundedBorder)
            .onSubmit {
                print("입력 완료: \(name)")
            }
    }
}
```

## State: 변화의 원천

### @State - 뷰가 소유하는 진실
```swift
struct CounterView: View {
    @State private var count = 0
    
    var body: some View {
        VStack {
            Text("카운트: \(count)")
            Button("증가") {
                count += 1  // 이것만으로 UI가 업데이트된다!
            }
        }
    }
}
```

왜 마법처럼 동작하는가?
1. `@State`는 값을 감시한다
2. 값이 바뀌면 SwiftUI에게 알린다
3. SwiftUI는 body를 다시 계산한다
4. 변경된 부분만 효율적으로 업데이트한다

### @Binding - 진실의 참조
```swift
struct ParentView: View {
    @State private var isOn = false
    
    var body: some View {
        ChildView(isOn: $isOn)  // $ = binding 전달
    }
}

struct ChildView: View {
    @Binding var isOn: Bool  // 부모의 상태를 참조
    
    var body: some View {
        Toggle("스위치", isOn: $isOn)
    }
}
```

### @StateObject & @ObservedObject - 복잡한 상태
```swift
class UserModel: ObservableObject {
    @Published var name = "익명"
    @Published var age = 0
}

struct ProfileView: View {
    @StateObject private var user = UserModel()  // 뷰가 소유
    
    var body: some View {
        VStack {
            Text("이름: \(user.name)")
            Text("나이: \(user.age)")
        }
    }
}
```

## Layout: 공간의 조화

### Stack들 - 쌓기의 예술
```swift
// 수직 쌓기
VStack(alignment: .leading, spacing: 10) {
    Text("첫 번째")
    Text("두 번째")
    Text("세 번째")
}

// 수평 쌓기
HStack {
    Image(systemName: "star")
    Text("별")
    Spacer()  // 남은 공간 차지
    Text("끝")
}

// 깊이 쌓기
ZStack {
    Color.blue  // 배경
    Text("앞")   // 전경
}
```

### GeometryReader - 공간 인식
```swift
GeometryReader { geometry in
    VStack {
        Text("너비: \(geometry.size.width)")
        Text("높이: \(geometry.size.height)")
        
        Rectangle()
            .frame(width: geometry.size.width / 2,
                   height: geometry.size.height / 2)
    }
}
```

### Grid - iOS 16의 선물
```swift
Grid {
    GridRow {
        Text("1,1")
        Text("1,2")
    }
    GridRow {
        Text("2,1")
        Text("2,2")
    }
}
```

## Modifier: 변화의 연쇄

```swift
Text("Hello")
    .font(.title)           // 폰트 변경
    .foregroundColor(.blue) // 색상 변경
    .padding()              // 여백 추가
    .background(Color.gray) // 배경 추가
    .cornerRadius(10)       // 모서리 둥글게
    .shadow(radius: 5)      // 그림자 추가
```

순서가 중요하다:
```swift
// 다른 결과
Text("Hello")
    .background(Color.blue)
    .padding()  // 패딩이 배경 밖에

Text("Hello")
    .padding()
    .background(Color.blue)  // 패딩이 배경 안에
```

## List & ForEach: 반복의 우아함

```swift
struct TodoListView: View {
    @State private var todos = ["공부", "운동", "독서"]
    
    var body: some View {
        List {
            ForEach(todos, id: \.self) { todo in
                HStack {
                    Image(systemName: "circle")
                    Text(todo)
                }
            }
            .onDelete { indexSet in
                todos.remove(atOffsets: indexSet)
            }
        }
    }
}
```

## Navigation: 공간의 이동

### NavigationStack (iOS 16+)
```swift
struct ContentView: View {
    @State private var path = NavigationPath()
    
    var body: some View {
        NavigationStack(path: $path) {
            List {
                NavigationLink("상세 페이지로", value: "detail")
            }
            .navigationDestination(for: String.self) { value in
                DetailView(item: value)
            }
            .navigationTitle("메인")
        }
    }
}
```

## Animation: 생명을 불어넣다

### 암시적 애니메이션
```swift
struct AnimatedView: View {
    @State private var scale = 1.0
    
    var body: some View {
        Image(systemName: "heart.fill")
            .scaleEffect(scale)
            .onTapGesture {
                scale = scale == 1.0 ? 1.5 : 1.0
            }
            .animation(.spring(), value: scale)  // 자동 애니메이션
    }
}
```

### 명시적 애니메이션
```swift
withAnimation(.easeInOut(duration: 0.5)) {
    // 이 블록 안의 상태 변경이 애니메이션된다
    isExpanded.toggle()
}
```

## Gesture: 의도를 읽다

```swift
struct DragView: View {
    @State private var offset = CGSize.zero
    
    var body: some View {
        Circle()
            .fill(Color.blue)
            .frame(width: 100, height: 100)
            .offset(offset)
            .gesture(
                DragGesture()
                    .onChanged { value in
                        offset = value.translation
                    }
                    .onEnded { _ in
                        withAnimation(.spring()) {
                            offset = .zero
                        }
                    }
            )
    }
}
```

## @Observable (Swift 5.9+, iOS 17+)

새로운 방식의 상태 관리 - ObservableObject보다 강력하고 효율적:

```swift
import Observation

// 이전 방식 (ObservableObject)
class OldUserViewModel: ObservableObject {
    @Published var name = "익명"  // 모든 프로퍼티에 @Published 필요
    @Published var age = 0
}

// 새로운 방식 (@Observable)
@Observable
class UserViewModel {
    var name = "익명"  // @Published 불필요!
    var age = 0
    
    // Computed property도 자동 추적
    var displayName: String {
        "\(name) (\(age)세)"
    }
    
    // 추적하지 않을 프로퍼티
    @ObservationIgnored
    private var internalCache = [String: Any]()
}

struct ProfileView: View {
    @State private var viewModel = UserViewModel()  // @StateObject 대신 @State
    
    var body: some View {
        VStack {
            Text(viewModel.displayName)
            TextField("이름", text: $viewModel.name)
            Stepper("나이: \(viewModel.age)", value: $viewModel.age)
        }
    }
}

// 성능 최적화: 필요한 부분만 관찰
struct OptimizedView: View {
    let viewModel: UserViewModel  // @ObservedObject 불필요
    
    var body: some View {
        // withObservationTracking으로 세밀한 제어
        Text(viewModel.name)  // name 변경시에만 업데이트
    }
}
```

### @Observable vs @ObservableObject 차이점

```swift
// 1. 더 간단한 문법
@Observable class NewWay {
    var value = 0  // 자동 추적
}

class OldWay: ObservableObject {
    @Published var value = 0  // 명시적 선언 필요
}

// 2. 더 나은 성능
struct PerformanceView: View {
    @State private var newModel = NewWay()
    
    var body: some View {
        // 실제 사용되는 프로퍼티만 추적
        Text("\(newModel.value)")  // value 변경시에만 업데이트
    }
}

// 3. 메모리 효율성
@Observable class ViewModel {
    var heavyData = HeavyData()  // 사용될 때만 관찰 시작
    
    @ObservationIgnored
    var cache = Cache()  // 명시적으로 제외
}
```

## Environment: 전역 상태

```swift
struct MyApp: App {
    @StateObject private var userSettings = UserSettings()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(userSettings)  // 주입
        }
    }
}

struct DeepChildView: View {
    @EnvironmentObject var settings: UserSettings  // 어디서든 접근
    
    var body: some View {
        Text("테마: \(settings.theme)")
    }
}
```

## ViewBuilder: 조건부 렌더링

```swift
struct ConditionalView: View {
    let showDetails: Bool
    
    var body: some View {
        VStack {
            Text("항상 보임")
            
            if showDetails {
                Text("상세 정보")
                    .transition(.slide)
            }
        }
    }
}

// ViewBuilder 함수
@ViewBuilder
func makeView(for condition: Bool) -> some View {
    if condition {
        Text("참")
    } else {
        Image(systemName: "xmark")
    }
}
```

## 성능 최적화 기초

### 1. 뷰 아이덴티티
```swift
ForEach(items, id: \.id) { item in  // id가 중요!
    ItemView(item: item)
}
```

### 2. 값 타입 선호
```swift
// 좋음: struct (값 타입)
struct ItemModel {
    let id: UUID
    var name: String
}

// 피하기: class (참조 타입, 필요한 경우만)
```

### 3. 무거운 연산 캐싱
```swift
struct ExpensiveView: View {
    let data: [Int]
    
    // 한 번만 계산
    var processedData: Int {
        data.reduce(0, +)  // 비싼 연산
    }
    
    var body: some View {
        Text("합계: \(processedData)")
    }
}
```

## 디버깅 팁

### 1. 뷰 업데이트 추적
```swift
struct MyView: View {
    var body: some View {
        let _ = print("MyView 업데이트됨")  // 언제 다시 그려지는지 확인
        Text("Hello")
    }
}
```

### 2. 프리뷰 활용
```swift
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
            .previewDevice("iPhone 15 Pro")
            .previewDisplayName("아이폰")
        
        ContentView()
            .previewDevice("iPad Pro (12.9-inch)")
            .previewDisplayName("아이패드")
    }
}
```

## 정리: L1의 핵심 원리

1. **선언적 UI**: "무엇"을 그릴지만 말하면 된다
2. **상태 기반**: State가 바뀌면 UI가 자동으로 바뀐다
3. **컴포지션**: 작은 뷰들을 조합해 큰 뷰를 만든다
4. **단방향 데이터 흐름**: State → UI → Action → State
5. **프로토콜 지향**: View는 프로토콜, 무엇이든 View가 될 수 있다

## First Principles 사고

왜 SwiftUI는 이렇게 설계되었을까?

1. **불변성**: 뷰는 값 타입(struct), 안전하고 예측 가능
2. **함수형**: UI = f(State), 부작용 최소화
3. **반응형**: 상태 변화에 자동 반응
4. **선언형**: 의도를 표현, 구현은 시스템이

이것이 미래다. 복잡함을 숨기고 본질만 남긴다.

## 다음 레벨로

이제 실제로 앱을 만들어볼 시간이다.

→ [L2: 첫 앱 만들기 - 배달 주문 앱](../L2_implementation/00_first_app.md)

---

*"프로그래밍은 글쓰기와 같다. 컴퓨터가 아닌 사람을 위해 쓰는 것이다." - Donald Knuth*