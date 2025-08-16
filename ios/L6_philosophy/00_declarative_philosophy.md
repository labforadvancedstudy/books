# L6: 선언적 UI의 철학 - SwiftUI가 바꾼 사고방식

## 패러다임의 전환

```
명령형: "어떻게" 그릴 것인가?
선언형: "무엇"이 되어야 하는가?

이것은 단순한 문법의 차이가 아니다.
사고방식의 혁명이다.
```

## 플라톤의 이데아와 SwiftUI

### 동굴의 비유

```swift
// 명령형 - 그림자를 직접 조작
class ShadowManipulator {
    func drawCircle() {
        moveHand(to: startPoint)
        for angle in 0..<360 {
            moveHand(alongCircle: angle)
            castShadow()
        }
    }
}

// 선언형 - 이데아를 선언
struct Circle: View {
    var body: some View {
        // 원의 본질만 선언
        // 실제 그리기는 시스템이
    }
}
```

플라톤은 현실 세계가 이데아의 그림자라고 했다.
SwiftUI에서 View는 이데아고, 화면의 픽셀은 그림자다.

## 시간과 상태: 헤라클리토스의 강

> "같은 강물에 두 번 발을 담글 수 없다" - 헤라클리토스

```swift
// 전통적 사고: 상태를 변경
var river = River()
river.flow()  // 강을 흐르게 한다
river.temperature = 15  // 온도를 바꾼다

// SwiftUI 사고: 상태가 곧 진실
struct RiverView: View {
    @State private var temperature = 15
    
    var body: some View {
        // temperature가 15일 때의 강
        // temperature가 20일 때는 다른 강
        // 매 순간 새로운 강
    }
}
```

UI는 상태의 스냅샷이다. 상태가 바뀌면 새로운 UI가 된다.
변경하는 것이 아니라, 새롭게 존재하는 것이다.

## 함수형 프로그래밍의 순수성

### 부작용 없는 세계

```swift
// 부작용이 있는 명령형
func updateView() {
    label.text = "Hello"  // 외부 상태 변경
    button.isEnabled = false  // 또 다른 변경
    NetworkManager.shared.fetch()  // 부작용의 연쇄
}

// 순수한 선언형
struct PureView: View {
    let name: String  // 입력
    
    var body: some View {  // 출력
        Text("Hello, \(name)")
        // 같은 입력 → 항상 같은 출력
        // 부작용 없음
    }
}
```

### 불변성의 힘

```swift
// 가변 상태의 위험
class MutableModel {
    var items = [Item]()  // 누가 언제 바꿀지 모름
    
    func addItem(_ item: Item) {
        items.append(item)  // 직접 변경
        // 다른 곳에서 items를 보고 있다면?
    }
}

// 불변성과 명확한 변경
@Observable
class ImmutableModel {
    private(set) var items = [Item]()  // 외부에서 직접 변경 불가
    
    func addItem(_ item: Item) {
        items = items + [item]  // 새로운 배열 생성
        // SwiftUI가 변경을 감지하고 다시 그림
    }
}
```

## 반응형 프로그래밍의 본질

### 데이터 흐름의 방향

```
┌─────────────────────────────────┐
│          Source of Truth         │
│            (State)               │
└────────────┬────────────────────┘
             │
             ▼ (단방향)
┌─────────────────────────────────┐
│        Derived Values            │
│     (Computed Properties)        │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│            View                  │
│         (Presentation)           │
└────────────┬────────────────────┘
             │
             ▼ (User Action)
┌─────────────────────────────────┐
│           Intent                 │
│    (Changes Source of Truth)     │
└─────────────────────────────────┘
```

### 진실의 단일 소스

```swift
// 문제: 여러 진실
struct BadView: View {
    @State var displayText = "Hello"
    @State var dataText = "Hello"
    
    // displayText와 dataText 중 뭐가 진짜?
}

// 해결: 하나의 진실
struct GoodView: View {
    @State var text = "Hello"  // 유일한 진실
    
    var displayText: String {  // 파생된 값
        text.uppercased()
    }
}
```

## 컴포지션의 미학

### 레고 블록 철학

```swift
// 원자적 컴포넌트
struct Atom: View {
    var body: some View {
        Circle()
    }
}

// 분자
struct Molecule: View {
    var body: some View {
        HStack {
            Atom()
            Atom()
        }
    }
}

// 유기체
struct Organism: View {
    var body: some View {
        VStack {
            Molecule()
            Molecule()
        }
    }
}

// 생태계
struct Ecosystem: View {
    var body: some View {
        ScrollView {
            ForEach(0..<10) { _ in
                Organism()
            }
        }
    }
}
```

작은 것들이 모여 큰 것을 만든다.
각 부분은 독립적이면서도 조화롭다.

## 추상화의 계층

### View Protocol의 재귀적 본질

```swift
protocol View {
    associatedtype Body: View  // View를 반환하는 View
    var body: Self.Body { get }
}

// 무한 재귀? 아니다.
// 어딘가에 기본 View가 있다 (Text, Image 등)
// 거북이 위에 거북이, 그 끝에는 SwiftUI
```

### 추상화 레벨

```swift
// L0: 픽셀
// Metal이 그리는 실제 픽셀

// L1: 기본 도형
Path { path in
    path.addEllipse(in: rect)
}

// L2: 시스템 컴포넌트
Circle()

// L3: 스타일 적용
Circle()
    .fill(Color.blue)
    .frame(width: 100)

// L4: 의미 부여
struct ProfileImage: View {
    var body: some View {
        Circle()
            .fill(Color.blue)
            .frame(width: 100)
    }
}

// L5: 비즈니스 로직
struct UserProfile: View {
    let user: User
    
    var body: some View {
        ProfileImage(url: user.imageURL)
    }
}
```

## Identity와 Lifetime

### 구조적 Identity

```swift
// SwiftUI는 어떻게 View를 구분할까?
struct ListView: View {
    let items: [Item]
    
    var body: some View {
        ForEach(items) { item in  // id로 구분
            ItemView(item: item)
        }
    }
}

// 명시적 Identity
ForEach(items, id: \.self) { item in
    ItemView(item: item)
}

// 조건부 Identity
if isLoggedIn {
    ProfileView()  // 조건이 바뀌면 다른 View
} else {
    LoginView()
}
```

### View의 생명주기

```swift
// 1. 생성 (Initialization)
struct LifecycleView: View {
    init() {
        print("View 생성")  // 여러 번 호출될 수 있음!
    }
    
    // 2. 나타남 (Appearance)
    var body: some View {
        Text("Hello")
            .onAppear {
                print("View 나타남")
            }
            // 3. 사라짐 (Disappearance)
            .onDisappear {
                print("View 사라짐")
            }
    }
    
    // View는 값 타입 - 소멸자 없음
    // 상태는 별도로 관리됨
}
```

## 애니메이션의 철학

### 시간의 함수

```swift
// 애니메이션은 시간에 따른 상태 변화
Animation = f(State, Time)

// 선형 보간
func linear(from: Double, to: Double, progress: Double) -> Double {
    from + (to - from) * progress
}

// 스프링 물리학
func spring(stiffness: Double, damping: Double) -> Animation {
    // 후크의 법칙: F = -kx
    // 감쇠 진동: x(t) = Ae^(-γt)cos(ωt + φ)
}
```

### 암시적 vs 명시적

```swift
// 암시적: "이 값이 변할 때 애니메이션"
@State var scale = 1.0

Circle()
    .scaleEffect(scale)
    .animation(.spring(), value: scale)

// 명시적: "이 시점에 애니메이션"
withAnimation(.spring()) {
    scale = 2.0
}

// 철학적 차이:
// 암시적 = 속성 중심 (무엇이)
// 명시적 = 시점 중심 (언제)
```

## Environment의 맥락

### 암묵적 의존성 주입

```swift
// 전통적 DI
class TraditionalView {
    let theme: Theme
    let locale: Locale
    let font: Font
    
    init(theme: Theme, locale: Locale, font: Font) {
        // 명시적 주입
    }
}

// SwiftUI Environment
struct ModernView: View {
    @Environment(\.colorScheme) var colorScheme
    @Environment(\.locale) var locale
    @Environment(\.font) var font
    
    // 암묵적 주입 - 필요한 것만 가져옴
}
```

### 컨텍스트의 전파

```swift
// 부모가 설정한 환경이 자식에게 전파
ContentView()
    .environment(\.locale, Locale(identifier: "ko_KR"))
    .environment(\.layoutDirection, .rightToLeft)
    .environmentObject(UserSettings())

// 깊은 곳의 자식도 접근 가능
// 마치 공기처럼 어디에나 있지만 보이지 않음
```

## 성능의 철학

### 차이점 계산 (Diffing)

```swift
// SwiftUI는 매번 body를 호출하지만
// 실제로는 변경된 부분만 다시 그린다

struct OptimizedView: View {
    @State var counter = 0
    let expensiveData: ExpensiveData  // 이건 다시 계산 안 됨
    
    var body: some View {
        VStack {
            Text("\(counter)")  // 이것만 업데이트
            ExpensiveView(data: expensiveData)  // 이건 그대로
        }
    }
}
```

### Lazy Evaluation

```swift
// 필요할 때만 계산
ScrollView {
    LazyVStack {  // 보이는 것만 렌더링
        ForEach(0..<1000) { i in
            ExpensiveRow(index: i)
        }
    }
}

// 철학: 게으름은 미덕이다
// 필요하지 않은 일은 하지 않는다
```

## 제약과 자유

### 제약이 만드는 창의성

```swift
// SwiftUI의 제약:
// - View는 struct여야 함
// - body는 순수 함수여야 함
// - @State는 View가 소유해야 함

// 이 제약들이 만드는 자유:
// - 예측 가능한 동작
// - 안전한 동시성
// - 자동 최적화
```

### 관습과 혁신

```swift
// UIKit의 관습
viewController.present(nextViewController, animated: true)

// SwiftUI의 혁신
NavigationLink("Next", destination: NextView())

// 표면적으로는 단순해 보이지만
// 내부적으로는 복잡한 상태 관리를 숨김
```

## 미래의 UI 철학

### AI와 선언형 UI

```swift
// 미래: 의도만 선언
struct FutureView: View {
    @Intent var userWants = "배고픔 해결"
    
    var body: some View {
        // AI가 의도를 해석하여 UI 생성
        IntentBasedUI(userWants)
    }
}
```

### 공간 컴퓨팅과 SwiftUI

```swift
// visionOS의 등장
struct SpatialView: View {
    var body: some View {
        RealityView { content in
            // 3D 공간도 선언적으로
        }
        .frame(depth: 500)  // Z축 추가
    }
}
```

## 정리: 선언형 UI의 본질

### 핵심 원리

1. **선언 > 명령**: 무엇을 원하는지 말하면 된다
2. **불변성**: 변경이 아닌 교체
3. **컴포지션**: 작은 것들의 조합
4. **반응성**: 상태 변화에 자동 반응
5. **순수성**: 부작용 최소화

### 철학적 함의

- **단순함의 복잡함**: 단순한 인터페이스 뒤의 복잡한 시스템
- **제어의 양도**: 세부 구현을 시스템에 맡김
- **신뢰의 관계**: 프레임워크를 신뢰해야 함
- **패러다임 전환**: 사고방식을 바꿔야 함

## First Principles: 왜 선언형인가?

인간의 사고는 본래 선언적이다.
"커피 한 잔 주세요" (선언적)
"원두를 갈고, 92도 물을 준비하고..." (명령형)

SwiftUI는 인간의 자연스러운 사고방식에 가깝다.
그래서 처음엔 낯설지만, 익숙해지면 돌아갈 수 없다.

## 다음 레벨로

플랫폼의 철학을 이해할 시간이다.

→ [L7: 플랫폼 철학 - Apple 생태계의 디자인 언어](../L7_platform/00_platform_philosophy.md)

---

*"Design is not just what it looks like and feels like. Design is how it works." - Steve Jobs*