# L0-01: 첫 화면 만들기
## 정적 UI에서 상호작용하는 앱으로

---

> **"Static screens are for portfolios. Interactive apps are for users."**

이전 단계에서 기본 UI를 만들었습니다. 이제 사용자와 실제로 상호작용하는 앱으로 발전시켜봅시다.

---

## 🎯 목표

**30분 후 결과물**:
- 도시 선택 기능
- 탭 네비게이션
- 실제 상태 변화하는 UI

**배울 것**:
- @State 속성 래퍼
- 사용자 입력 처리
- 조건부 UI 렌더링

---

## 🚀 실습: 상호작용 추가하기

### 1단계: 상태 관리 도입

`ContentView.swift`를 다음과 같이 수정:

```swift
import SwiftUI

// MARK: - 데이터 모델
struct WeatherData {
    let city: String
    let temperature: Int
    let condition: String
    let high: Int
    let low: Int
    let humidity: String
    let wind: String
    let feelsLike: String
    let icon: String
}

// MARK: - 메인 앱 구조
struct ContentView: View {
    // 상태 관리
    @State private var selectedCity = "서울시"
    @State private var isRefreshing = false
    @State private var lastRefreshTime = Date()
    
    // 더미 데이터 (나중에 실제 API로 교체)
    private let weatherData: [String: WeatherData] = [
        "서울시": WeatherData(
            city: "서울시",
            temperature: 23,
            condition: "맑음",
            high: 28,
            low: 18,
            humidity: "65%",
            wind: "2m/s",
            feelsLike: "25°C",
            icon: "sun.max.fill"
        ),
        "부산시": WeatherData(
            city: "부산시",
            temperature: 26,
            condition: "구름많음",
            high: 30,
            low: 22,
            humidity: "70%",
            wind: "3m/s",
            feelsLike: "28°C",
            icon: "cloud.sun.fill"
        ),
        "대구시": WeatherData(
            city: "대구시",
            temperature: 28,
            condition: "흐림",
            high: 32,
            low: 24,
            humidity: "75%",
            wind: "1m/s",
            feelsLike: "30°C",
            icon: "cloud.fill"
        )
    ]
    
    var currentWeather: WeatherData {
        weatherData[selectedCity] ?? weatherData["서울시"]!
    }
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 20) {
                // 도시 선택기
                CityPickerView(
                    selectedCity: $selectedCity,
                    cities: Array(weatherData.keys)
                )
                
                // 메인 날씨 카드
                WeatherCardView(weather: currentWeather)
                
                // 새로고침 시간
                LastRefreshView(time: lastRefreshTime)
                
                Spacer()
                
                // 새로고침 버튼
                RefreshButton(
                    isRefreshing: $isRefreshing,
                    onRefresh: refreshWeather
                )
            }
            .padding()
            .navigationTitle("WeatherNow")
            .navigationBarTitleDisplayMode(.large)
        }
    }
    
    // MARK: - Actions
    private func refreshWeather() {
        isRefreshing = true
        
        // 실제 API 호출 시뮬레이션
        DispatchQueue.main.asyncAfter(deadline: .now() + 1.5) {
            lastRefreshTime = Date()
            isRefreshing = false
        }
    }
}

// MARK: - 새로운 컴포넌트들
struct CityPickerView: View {
    @Binding var selectedCity: String
    let cities: [String]
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text("지역 선택")
                .font(.headline)
                .foregroundColor(.primary)
            
            ScrollView(.horizontal, showsIndicators: false) {
                HStack(spacing: 12) {
                    ForEach(cities.sorted(), id: \.self) { city in
                        CityButton(
                            city: city,
                            isSelected: city == selectedCity
                        ) {
                            selectedCity = city
                        }
                    }
                }
                .padding(.horizontal, 4)
            }
        }
    }
}

struct CityButton: View {
    let city: String
    let isSelected: Bool
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            Text(city)
                .font(.subheadline)
                .fontWeight(.medium)
                .padding(.horizontal, 16)
                .padding(.vertical, 8)
                .background(
                    RoundedRectangle(cornerRadius: 20)
                        .fill(isSelected ? .blue : .gray.opacity(0.2))
                )
                .foregroundColor(isSelected ? .white : .primary)
        }
        .animation(.easeInOut(duration: 0.2), value: isSelected)
    }
}

struct WeatherCardView: View {
    let weather: WeatherData
    
    var body: some View {
        VStack(spacing: 15) {
            // 헤더
            HStack {
                VStack(alignment: .leading) {
                    Text(weather.city)
                        .font(.title2)
                        .fontWeight(.semibold)
                    Text(weather.condition)
                        .font(.subheadline)
                        .foregroundColor(.secondary)
                }
                
                Spacer()
                
                Image(systemName: weather.icon)
                    .font(.system(size: 40))
                    .foregroundColor(.orange)
            }
            
            // 온도
            HStack {
                Text("\(weather.temperature)°C")
                    .font(.system(size: 64, weight: .thin))
                
                Spacer()
                
                VStack(alignment: .trailing, spacing: 4) {
                    HStack {
                        Image(systemName: "thermometer.sun")
                            .foregroundColor(.red)
                        Text("최고: \(weather.high)°C")
                    }
                    HStack {
                        Image(systemName: "thermometer.snowflake")
                            .foregroundColor(.blue)
                        Text("최저: \(weather.low)°C")
                    }
                }
                .font(.caption)
                .foregroundColor(.secondary)
            }
            
            Divider()
            
            // 상세 정보
            HStack {
                WeatherDetailItem(
                    icon: "humidity",
                    title: "습도",
                    value: weather.humidity
                )
                Spacer()
                WeatherDetailItem(
                    icon: "wind",
                    title: "바람",
                    value: weather.wind
                )
                Spacer()
                WeatherDetailItem(
                    icon: "thermometer",
                    title: "체감",
                    value: weather.feelsLike
                )
            }
        }
        .padding(20)
        .background(
            RoundedRectangle(cornerRadius: 20)
                .fill(.regularMaterial)
                .shadow(color: .black.opacity(0.1), radius: 10, x: 0, y: 5)
        )
        .animation(.easeInOut, value: weather.city)
    }
}

struct WeatherDetailItem: View {
    let icon: String
    let title: String
    let value: String
    
    var body: some View {
        VStack(spacing: 4) {
            Image(systemName: icon)
                .font(.title3)
                .foregroundColor(.blue)
            
            Text(title)
                .font(.caption2)
                .foregroundColor(.secondary)
                .textCase(.uppercase)
            
            Text(value)
                .font(.caption)
                .fontWeight(.semibold)
        }
    }
}

struct LastRefreshView: View {
    let time: Date
    
    private var timeFormatter: DateFormatter {
        let formatter = DateFormatter()
        formatter.dateFormat = "HH:mm:ss"
        return formatter
    }
    
    var body: some View {
        HStack {
            Image(systemName: "clock")
                .foregroundColor(.secondary)
            Text("마지막 업데이트: \(timeFormatter.string(from: time))")
                .font(.caption)
                .foregroundColor(.secondary)
        }
    }
}

struct RefreshButton: View {
    @Binding var isRefreshing: Bool
    let onRefresh: () -> Void
    
    var body: some View {
        Button(action: onRefresh) {
            HStack(spacing: 8) {
                if isRefreshing {
                    ProgressView()
                        .scaleEffect(0.8)
                        .tint(.white)
                } else {
                    Image(systemName: "arrow.clockwise")
                        .font(.headline)
                }
                
                Text(isRefreshing ? "업데이트 중..." : "새로고침")
                    .font(.headline)
            }
            .foregroundColor(.white)
            .frame(maxWidth: .infinity)
            .padding(.vertical, 16)
            .background(
                RoundedRectangle(cornerRadius: 12)
                    .fill(isRefreshing ? .gray : .blue)
            )
        }
        .disabled(isRefreshing)
        .animation(.easeInOut, value: isRefreshing)
    }
}

// MARK: - 프리뷰
#Preview {
    ContentView()
}
```

### 2단계: 실행하고 테스트

**⌘ + R**로 앱 실행 후:

1. **도시 버튼 클릭** → 날씨 데이터가 변경되는지 확인
2. **새로고침 버튼 클릭** → 로딩 상태와 시간 업데이트 확인
3. **다크 모드 전환** → UI가 자동 적응하는지 확인

---

## 🎯 여기서 배운 것

### 1. **상태 관리 (@State)**
```swift
@State private var selectedCity = "서울시"
@State private var isRefreshing = false
@State private var lastRefreshTime = Date()
```

- `@State`: 뷰의 상태를 관리하는 속성 래퍼
- `private`: 외부에서 직접 접근 방지
- 상태가 변경되면 자동으로 UI 재렌더링

### 2. **바인딩 (@Binding)**
```swift
struct CityPickerView: View {
    @Binding var selectedCity: String  // 양방향 데이터 바인딩
    
    // 사용법
    CityPickerView(selectedCity: $selectedCity)
}
```

- `@Binding`: 부모 뷰의 상태를 자식 뷰에서 수정 가능
- `$`: 바인딩을 전달할 때 사용하는 projection

### 3. **조건부 렌더링**
```swift
// 상태에 따른 다른 UI 표시
if isRefreshing {
    ProgressView()
} else {
    Image(systemName: "arrow.clockwise")
}
```

### 4. **애니메이션**
```swift
.animation(.easeInOut(duration: 0.2), value: isSelected)
.animation(.easeInOut, value: weather.city)
```

---

## 🔍 핵심 개념 깊이 이해

### @State vs @Binding

```swift
// @State: 소유권이 있는 상태 (이 뷰에서 생성)
@State private var counter = 0

// @Binding: 소유권이 없는 상태 (부모에서 전달받음)
@Binding var counter: Int

// 사용 예시
struct ParentView: View {
    @State private var count = 0  // 소유
    
    var body: some View {
        ChildView(count: $count)   // 바인딩으로 전달
    }
}

struct ChildView: View {
    @Binding var count: Int       // 바인딩으로 받음
    
    var body: some View {
        Button("Increment") {
            count += 1            // 부모의 상태 변경 가능
        }
    }
}
```

### 계산 속성 활용

```swift
var currentWeather: WeatherData {
    weatherData[selectedCity] ?? weatherData["서울시"]!
}
```

- 상태가 변경될 때마다 자동으로 다시 계산
- UI에서 직접 사용 가능
- 실제 데이터 변환 로직을 캡슐화

### DispatchQueue 비동기 처리

```swift
private func refreshWeather() {
    isRefreshing = true
    
    // 메인 큐에서 1.5초 후 실행
    DispatchQueue.main.asyncAfter(deadline: .now() + 1.5) {
        lastRefreshTime = Date()
        isRefreshing = false
    }
}
```

---

## 🚨 자주하는 실수와 해결법

### 1. **Binding 전달 실수**
```swift
// ❌ 잘못된 방법
CityPickerView(selectedCity: selectedCity)

// ✅ 올바른 방법
CityPickerView(selectedCity: $selectedCity)  // $ 사용
```

### 2. **Animation 대상 지정**
```swift
// ❌ 모든 변화에 애니메이션
.animation(.easeInOut)

// ✅ 특정 값 변화에만 애니메이션
.animation(.easeInOut, value: isSelected)
```

### 3. **ForEach에서 id 지정**
```swift
// ❌ 고유 식별자 없음
ForEach(cities) { city in
    // 컴파일 에러
}

// ✅ 고유 식별자 지정
ForEach(cities, id: \.self) { city in
    // 정상 작동
}
```

### 4. **Button disabled 상태**
```swift
Button(action: onRefresh) {
    Text("새로고침")
}
.disabled(isRefreshing)  // 로딩 중에는 비활성화
```

---

## 🎯 실습 과제

### 과제 1: 온도 단위 변환 추가
```swift
// Celsius ↔ Fahrenheit 변환 버튼 추가
@State private var isCelsius = true

var displayTemperature: String {
    if isCelsius {
        return "\(weather.temperature)°C"
    } else {
        let fahrenheit = weather.temperature * 9/5 + 32
        return "\(fahrenheit)°F"
    }
}
```

### 과제 2: 즐겨찾기 도시 기능
```swift
@State private var favoriteCities: Set<String> = []

// 도시 버튼에 하트 아이콘 추가
// 탭하면 즐겨찾기 토글
```

### 과제 3: 색상 테마 변경
```swift
// 날씨 상태에 따른 배경색 변경
var backgroundColor: Color {
    switch weather.condition {
    case "맑음": return .orange.opacity(0.3)
    case "흐림": return .gray.opacity(0.3)
    default: return .blue.opacity(0.3)
    }
}
```

---

## 🎉 성공 확인

**체크리스트**:
- [ ] 도시 버튼 클릭 시 날씨 데이터 변경됨
- [ ] 새로고침 버튼 클릭 시 로딩 상태 표시됨
- [ ] 마지막 업데이트 시간이 실시간으로 변경됨
- [ ] 선택된 도시 버튼이 시각적으로 구분됨
- [ ] 애니메이션이 부드럽게 작동함

**고급 테스트**:
- [ ] 빠르게 도시를 변경해도 UI가 깨지지 않음
- [ ] 새로고침 중에 다른 버튼 클릭해도 안전함
- [ ] 가로/세로 모드 전환 시 레이아웃 유지됨

---

## 💡 성능 최적화 팁

### 1. **불필요한 재렌더링 방지**
```swift
// ❌ 매번 새로운 DateFormatter 생성
Text(Date().formatted())

// ✅ 재사용 가능한 DateFormatter
private var timeFormatter: DateFormatter {
    let formatter = DateFormatter()
    formatter.dateFormat = "HH:mm:ss"
    return formatter
}
```

### 2. **계산 속성 최적화**
```swift
// 복잡한 계산은 캐싱 고려
var expensiveCalculation: String {
    // 무거운 연산...
}
```

---

## 🚀 다음 단계

훌륭합니다! 이제 상호작용하는 앱을 만들었습니다.

**지금까지 배운 것**:
- ✅ @State와 @Binding으로 상태 관리
- ✅ 사용자 입력 처리
- ✅ 조건부 UI 렌더링
- ✅ 애니메이션과 전환 효과

**다음에 할 것**:
→ **[02. 데이터와 연결](02_connect_data.md)** - 실제 API에서 데이터 가져오기

---

**축하합니다! 이제 진짜 상호작용하는 앱을 만들 수 있습니다.**

→ **[다음: 데이터와 연결하기](02_connect_data.md)**

---

*"An interface is humane if it is responsive to human needs and considerate of human frailties." - Jef Raskin*