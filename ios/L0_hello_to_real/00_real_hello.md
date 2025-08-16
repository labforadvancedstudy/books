# L0-00: 진짜 Hello World
## 90%의 책이 멈추는 지점을 넘어서

---

> **"Most tutorials stop at Hello World. We start there."**

대부분의 iOS 개발서는 `Text("Hello, World!")`로 시작해서 거기서 멈춥니다. 우리는 그것을 **시작점**으로 사용해 실제 앱을 만들어봅니다.

---

## 🎯 목표

**30분 후 결과물**:
- Xcode 프로젝트 생성부터 실행까지
- Hello World가 아닌 실제 앱 구조
- 시뮬레이터에서 작동하는 앱

**배울 것**:
- Xcode 프로젝트 설정
- SwiftUI App 구조 이해
- 첫 실제 UI 구현

---

## 📋 사전 준비

### 확인사항
```bash
# 터미널에서 실행
xcodebuild -version
# 출력: Xcode 16.0 이상이어야 함

swift --version  
# 출력: Swift 6.0 이상이어야 함
```

### 필요한 지식
```swift
// 이 코드가 이해된다면 시작 가능
struct MyView: View {
    var body: some View {
        Text("Ready to start!")
    }
}
```

이해 안 된다면? → [Swift 기본서](https://docs.swift.org/swift-book/) 먼저 공부

---

## 🚀 실습: 첫 번째 실제 앱

### 1단계: 프로젝트 생성

**Xcode 열기 → Create New Project**

```
Platform: iOS
Template: App
Product Name: WeatherNow
Interface: SwiftUI
Language: Swift
Bundle Identifier: com.yourname.weathernow
```

⚠️ **중요**: Bundle Identifier는 실제 출시할 때 필요하므로 신중히 선택

### 2단계: 기본 구조 이해

생성된 `ContentView.swift`를 보면:

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack {
            Image(systemName: "globe")
                .imageScale(.large)
                .foregroundStyle(.tint)
            Text("Hello, world!")
        }
        .padding()
    }
}

#Preview {
    ContentView()
}
```

**90%의 책이 여기서 멈춥니다.** 우리는 이것을 실제 앱으로 만들어봅시다.

### 3단계: 진짜 앱 구조로 변경

`ContentView.swift`를 다음과 같이 완전히 교체:

```swift
import SwiftUI

// MARK: - 메인 앱 구조
struct ContentView: View {
    var body: some View {
        NavigationStack {
            VStack(spacing: 20) {
                // 헤더
                HeaderView()
                
                // 메인 컨텐츠
                WeatherCardView()
                
                Spacer()
                
                // 하단 버튼
                RefreshButton()
            }
            .padding()
            .navigationTitle("WeatherNow")
            .navigationBarTitleDisplayMode(.large)
        }
    }
}

// MARK: - 컴포넌트들
struct HeaderView: View {
    var body: some View {
        VStack {
            Image(systemName: "sun.max.fill")
                .font(.system(size: 50))
                .foregroundColor(.orange)
            
            Text("오늘의 날씨")
                .font(.title2)
                .fontWeight(.medium)
        }
    }
}

struct WeatherCardView: View {
    var body: some View {
        VStack(spacing: 15) {
            HStack {
                Text("서울시")
                    .font(.headline)
                Spacer()
                Text("맑음")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
            
            HStack {
                Text("23°C")
                    .font(.system(size: 48, weight: .thin))
                Spacer()
                VStack(alignment: .trailing) {
                    Text("최고: 28°C")
                    Text("최저: 18°C")
                }
                .font(.caption)
                .foregroundColor(.secondary)
            }
            
            HStack {
                WeatherDetailItem(title: "습도", value: "65%")
                Spacer()
                WeatherDetailItem(title: "바람", value: "2m/s")
                Spacer()
                WeatherDetailItem(title: "체감", value: "25°C")
            }
        }
        .padding()
        .background(
            RoundedRectangle(cornerRadius: 15)
                .fill(.regularMaterial)
        )
    }
}

struct WeatherDetailItem: View {
    let title: String
    let value: String
    
    var body: some View {
        VStack {
            Text(title)
                .font(.caption)
                .foregroundColor(.secondary)
            Text(value)
                .font(.caption)
                .fontWeight(.medium)
        }
    }
}

struct RefreshButton: View {
    var body: some View {
        Button(action: {
            // 나중에 실제 기능 구현
            print("새로고침 버튼 클릭")
        }) {
            HStack {
                Image(systemName: "arrow.clockwise")
                Text("새로고침")
            }
            .font(.headline)
            .foregroundColor(.white)
            .frame(maxWidth: .infinity)
            .padding()
            .background(
                RoundedRectangle(cornerRadius: 10)
                    .fill(.blue)
            )
        }
    }
}

// MARK: - 프리뷰
#Preview {
    ContentView()
}
```

### 4단계: 실행하고 확인

**⌘ + R** 또는 Play 버튼 클릭

**기대 결과**:
- ✅ 앱이 시뮬레이터에서 실행됨
- ✅ "WeatherNow" 타이틀 표시
- ✅ 날씨 카드 UI 표시
- ✅ 새로고침 버튼 클릭 시 콘솔에 메시지 출력

---

## 🎯 여기서 배운 것

### 1. **실제 앱 구조**
```swift
NavigationStack {           // 네비게이션 컨테이너
    VStack {               // 세로 레이아웃
        HeaderView()       // 컴포넌트 분리
        WeatherCardView()  // 재사용 가능한 뷰
        RefreshButton()    // 상호작용 요소
    }
}
```

### 2. **컴포넌트 기반 설계**
```swift
// 각 기능을 별도 View로 분리
struct HeaderView: View { ... }
struct WeatherCardView: View { ... }
struct RefreshButton: View { ... }
```

### 3. **Material Design 시스템**
```swift
.background(.regularMaterial)  // iOS 기본 디자인 시스템 활용
.foregroundColor(.secondary)   // 시스템 색상 사용
```

---

## 🔍 코드 분석

### NavigationStack vs NavigationView
```swift
// ✅ iOS 16+ 권장 (우리가 사용)
NavigationStack { ... }

// ❌ 구버전 (deprecated)
NavigationView { ... }
```

### VStack, HStack 활용
```swift
VStack(spacing: 20) {    // 세로 정렬, 20pt 간격
    HeaderView()
    WeatherCardView()
    Spacer()             // 남은 공간 채우기
}

HStack {                 // 가로 정렬
    Text("서울시")
    Spacer()             // 좌우 끝 정렬
    Text("맑음")
}
```

### Material과 RoundedRectangle
```swift
.background(
    RoundedRectangle(cornerRadius: 15)
        .fill(.regularMaterial)    // iOS 블러 효과
)
```

---

## 🚨 자주 하는 실수

### 1. **PreviewProvider 대신 #Preview 사용**
```swift
// ❌ 구버전
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

// ✅ Xcode 15+ 권장
#Preview {
    ContentView()
}
```

### 2. **Body 계산 속성 이해**
```swift
// ❌ 잘못된 이해
var body: some View {
    return Text("Hello")  // return 키워드 불필요
}

// ✅ 올바른 SwiftUI 방식
var body: some View {
    Text("Hello")         // 단일 표현식
}
```

### 3. **Spacer() 활용**
```swift
// ❌ 수동 위치 조정
VStack {
    Text("상단")
    Text("하단").offset(y: 200)  // 하드코딩된 위치
}

// ✅ Spacer 활용
VStack {
    Text("상단")
    Spacer()     // 자동으로 공간 분배
    Text("하단")
}
```

---

## 🎉 성공 확인

**체크리스트**:
- [ ] 앱이 시뮬레이터에서 실행됨
- [ ] NavigationStack 타이틀이 "WeatherNow"로 표시됨
- [ ] 날씨 카드가 Material 배경으로 표시됨
- [ ] 새로고침 버튼 클릭 시 콘솔에 메시지 출력됨
- [ ] 다크 모드 전환 시 UI가 자동 적응됨

**다크 모드 테스트**:
시뮬레이터에서 **Settings → Developer → Dark Appearance** 토글

---

## 🎯 다음 단계

훌륭합니다! 이제 "Hello World"에서 벗어나 실제 앱 구조를 만들었습니다.

**지금까지 만든 것**:
- ✅ 실제 앱 같은 UI 구조
- ✅ 컴포넌트 기반 설계
- ✅ Material Design 시스템 활용

**다음에 할 것**:
→ **[01. 첫 화면 만들기](01_first_screen.md)** - 더 정교한 UI와 상태 관리

---

## 💡 핵심 포인트

### 1. **컴포넌트 분리가 핵심**
큰 뷰를 작은 뷰들로 나누는 것이 SwiftUI의 기본 철학입니다.

### 2. **시스템 디자인 활용**
`.regularMaterial`, `.secondary` 같은 시스템 요소를 사용하면 자동으로 다크 모드와 접근성이 지원됩니다.

### 3. **Preview는 개발 도구**
`#Preview`를 활용하면 시뮬레이터 실행 없이 UI를 즉시 확인할 수 있습니다.

---

**축하합니다! 이제 진짜 앱 개발자의 첫 걸음을 내디뎠습니다.**

→ **[다음: 첫 화면 만들기](01_first_screen.md)**

---

*"Every expert was once a beginner. Every pro was once an amateur."*