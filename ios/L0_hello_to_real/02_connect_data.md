# L0-02: 데이터와 연결
## 정적 데이터에서 실제 API로

---

> **"Real apps use real data. Everything else is just a demo."**

지금까지 하드코딩된 데이터를 사용했습니다. 이제 실제 날씨 API에서 데이터를 가져와서 진짜 앱으로 만들어봅시다.

---

## 🎯 목표

**45분 후 결과물**:
- OpenWeatherMap API 연동
- 실제 날씨 데이터 표시
- 네트워크 에러 처리

**배울 것**:
- HTTP 네트워킹 기초
- JSON 디코딩
- async/await 비동기 처리
- 에러 핸들링

---

## 📋 사전 준비

### OpenWeatherMap API 키 발급

1. **[OpenWeatherMap](https://openweathermap.org/api) 접속**
2. **Sign up** → 무료 계정 생성
3. **API Keys** → API 키 복사 (예: `abcd1234efgh5678`)
4. **5분 정도 기다림** (활성화 시간)

⚠️ **중요**: API 키는 개인정보입니다. Git에 커밋하지 마세요!

---

## 🚀 실습: 실제 API 연동

### 1단계: 네트워킹 모델 구조 설계

새 파일 `WeatherService.swift` 생성:

```swift
import Foundation

// MARK: - API Response 모델
struct WeatherResponse: Codable {
    let name: String
    let main: Main
    let weather: [Weather]
    let wind: Wind
    let sys: Sys
    
    struct Main: Codable {
        let temp: Double
        let feelsLike: Double
        let tempMin: Double
        let tempMax: Double
        let humidity: Int
        
        enum CodingKeys: String, CodingKey {
            case temp, humidity
            case feelsLike = "feels_like"
            case tempMin = "temp_min"
            case tempMax = "temp_max"
        }
    }
    
    struct Weather: Codable {
        let id: Int
        let main: String
        let description: String
        let icon: String
    }
    
    struct Wind: Codable {
        let speed: Double
    }
    
    struct Sys: Codable {
        let country: String
    }
}

// MARK: - 우리 앱의 데이터 모델
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
    let country: String
    
    // API 응답을 우리 모델로 변환
    init(from response: WeatherResponse) {
        self.city = response.name
        self.temperature = Int(response.main.temp.rounded())
        self.condition = response.weather.first?.description.capitalized ?? "알 수 없음"
        self.high = Int(response.main.tempMax.rounded())
        self.low = Int(response.main.tempMin.rounded())
        self.humidity = "\(response.main.humidity)%"
        self.wind = String(format: "%.1fm/s", response.wind.speed)
        self.feelsLike = "\(Int(response.main.feelsLike.rounded()))°C"
        self.icon = Self.mapWeatherIcon(response.weather.first?.icon ?? "01d")
        self.country = response.sys.country
    }
    
    // OpenWeatherMap 아이콘을 SF Symbols로 매핑
    private static func mapWeatherIcon(_ apiIcon: String) -> String {
        switch apiIcon {
        case "01d", "01n": return "sun.max.fill"
        case "02d", "02n": return "cloud.sun.fill"
        case "03d", "03n", "04d", "04n": return "cloud.fill"
        case "09d", "09n": return "cloud.drizzle.fill"
        case "10d", "10n": return "cloud.rain.fill"
        case "11d", "11n": return "cloud.bolt.fill"
        case "13d", "13n": return "cloud.snow.fill"
        case "50d", "50n": return "cloud.fog.fill"
        default: return "questionmark.circle.fill"
        }
    }
}

// MARK: - 네트워킹 에러
enum WeatherError: LocalizedError, CaseIterable {
    case invalidURL
    case noData
    case decodingError
    case networkError
    case apiKeyMissing
    case cityNotFound
    
    var errorDescription: String? {
        switch self {
        case .invalidURL:
            return "잘못된 URL입니다."
        case .noData:
            return "데이터를 받을 수 없습니다."
        case .decodingError:
            return "데이터 형식이 올바르지 않습니다."
        case .networkError:
            return "네트워크 연결을 확인해주세요."
        case .apiKeyMissing:
            return "API 키가 필요합니다."
        case .cityNotFound:
            return "해당 도시를 찾을 수 없습니다."
        }
    }
}

// MARK: - 날씨 서비스
@MainActor
class WeatherService: ObservableObject {
    // ⚠️ 실제 API 키로 교체하세요!
    private let apiKey = "YOUR_API_KEY_HERE"
    private let baseURL = "https://api.openweathermap.org/data/2.5/weather"
    
    @Published var currentWeather: WeatherData?
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    // 도시별 날씨 데이터 가져오기
    func fetchWeather(for city: String) async {
        // API 키 확인
        guard apiKey != "YOUR_API_KEY_HERE" else {
            errorMessage = WeatherError.apiKeyMissing.localizedDescription
            return
        }
        
        isLoading = true
        errorMessage = nil
        
        do {
            let weatherData = try await performWeatherRequest(for: city)
            currentWeather = weatherData
        } catch {
            if let weatherError = error as? WeatherError {
                errorMessage = weatherError.localizedDescription
            } else {
                errorMessage = "알 수 없는 오류가 발생했습니다."
            }
        }
        
        isLoading = false
    }
    
    // 실제 API 요청 수행
    private func performWeatherRequest(for city: String) async throws -> WeatherData {
        // URL 생성
        var components = URLComponents(string: baseURL)
        components?.queryItems = [
            URLQueryItem(name: "q", value: city),
            URLQueryItem(name: "appid", value: apiKey),
            URLQueryItem(name: "units", value: "metric"), // 섭씨 온도
            URLQueryItem(name: "lang", value: "kr")        // 한국어
        ]
        
        guard let url = components?.url else {
            throw WeatherError.invalidURL
        }
        
        // HTTP 요청
        let (data, response) = try await URLSession.shared.data(from: url)
        
        // HTTP 상태 코드 확인
        if let httpResponse = response as? HTTPURLResponse {
            switch httpResponse.statusCode {
            case 200:
                break // 성공
            case 404:
                throw WeatherError.cityNotFound
            default:
                throw WeatherError.networkError
            }
        }
        
        // JSON 디코딩
        do {
            let weatherResponse = try JSONDecoder().decode(WeatherResponse.self, from: data)
            return WeatherData(from: weatherResponse)
        } catch {
            print("디코딩 에러: \(error)")
            throw WeatherError.decodingError
        }
    }
}
```

### 2단계: ContentView와 연동

`ContentView.swift` 수정:

```swift
import SwiftUI

struct ContentView: View {
    // WeatherService 인스턴스
    @StateObject private var weatherService = WeatherService()
    
    // UI 상태
    @State private var selectedCity = "Seoul"
    @State private var lastRefreshTime = Date()
    
    // 도시 목록 (영어명으로 API 호출)
    private let cities = [
        "Seoul": "서울",
        "Busan": "부산", 
        "Daegu": "대구",
        "Incheon": "인천",
        "Gwangju": "광주",
        "Daejeon": "대전",
        "Ulsan": "울산"
    ]
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 20) {
                // 도시 선택기
                CityPickerView(
                    selectedCity: $selectedCity,
                    cities: cities,
                    onCityChanged: { city in
                        Task {
                            await weatherService.fetchWeather(for: city)
                        }
                    }
                )
                
                // 메인 컨텐츠
                if weatherService.isLoading {
                    LoadingView()
                } else if let errorMessage = weatherService.errorMessage {
                    ErrorView(message: errorMessage) {
                        Task {
                            await weatherService.fetchWeather(for: selectedCity)
                        }
                    }
                } else if let weather = weatherService.currentWeather {
                    WeatherCardView(weather: weather)
                    LastRefreshView(time: lastRefreshTime)
                } else {
                    EmptyStateView {
                        Task {
                            await weatherService.fetchWeather(for: selectedCity)
                        }
                    }
                }
                
                Spacer()
                
                // 새로고침 버튼
                RefreshButton(
                    isRefreshing: weatherService.isLoading,
                    onRefresh: refreshWeather
                )
            }
            .padding()
            .navigationTitle("WeatherNow")
            .navigationBarTitleDisplayMode(.large)
            .task {
                // 앱 시작 시 서울 날씨 로드
                await weatherService.fetchWeather(for: selectedCity)
            }
        }
    }
    
    // MARK: - Actions
    private func refreshWeather() {
        Task {
            await weatherService.fetchWeather(for: selectedCity)
            lastRefreshTime = Date()
        }
    }
}

// MARK: - 새로운 UI 컴포넌트들
struct CityPickerView: View {
    @Binding var selectedCity: String
    let cities: [String: String]
    let onCityChanged: (String) -> Void
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text("지역 선택")
                .font(.headline)
                .foregroundColor(.primary)
            
            ScrollView(.horizontal, showsIndicators: false) {
                HStack(spacing: 12) {
                    ForEach(Array(cities.keys).sorted(), id: \.self) { cityKey in
                        CityButton(
                            city: cities[cityKey] ?? cityKey,
                            isSelected: cityKey == selectedCity
                        ) {
                            selectedCity = cityKey
                            onCityChanged(cityKey)
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

struct LoadingView: View {
    var body: some View {
        VStack(spacing: 20) {
            ProgressView()
                .scaleEffect(1.5)
                .tint(.blue)
            
            Text("날씨 정보를 가져오는 중...")
                .font(.subheadline)
                .foregroundColor(.secondary)
        }
        .frame(height: 200)
    }
}

struct ErrorView: View {
    let message: String
    let onRetry: () -> Void
    
    var body: some View {
        VStack(spacing: 20) {
            Image(systemName: "exclamationmark.triangle.fill")
                .font(.system(size: 50))
                .foregroundColor(.red)
            
            Text("오류가 발생했습니다")
                .font(.headline)
            
            Text(message)
                .font(.subheadline)
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)
            
            Button("다시 시도") {
                onRetry()
            }
            .buttonStyle(.borderedProminent)
        }
        .frame(height: 200)
        .padding()
    }
}

struct EmptyStateView: View {
    let onLoad: () -> Void
    
    var body: some View {
        VStack(spacing: 20) {
            Image(systemName: "cloud.sun.fill")
                .font(.system(size: 50))
                .foregroundColor(.blue)
            
            Text("날씨 정보를 불러오세요")
                .font(.headline)
            
            Button("날씨 보기") {
                onLoad()
            }
            .buttonStyle(.borderedProminent)
        }
        .frame(height: 200)
    }
}

struct WeatherCardView: View {
    let weather: WeatherData
    
    var body: some View {
        VStack(spacing: 15) {
            // 헤더
            HStack {
                VStack(alignment: .leading) {
                    HStack {
                        Text(weather.city)
                            .font(.title2)
                            .fontWeight(.semibold)
                        Text(weather.country)
                            .font(.caption)
                            .padding(.horizontal, 8)
                            .padding(.vertical, 2)
                            .background(.gray.opacity(0.2))
                            .clipShape(Capsule())
                    }
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
    let isRefreshing: Bool
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

### 3단계: API 키 설정 및 테스트

1. **WeatherService.swift**에서 `YOUR_API_KEY_HERE`를 실제 API 키로 교체
2. **⌘ + R**로 앱 실행
3. **네트워크 권한 허용** (시뮬레이터에서 자동)

---

## 🎯 여기서 배운 것

### 1. **HTTP 네트워킹**
```swift
// URLComponents로 안전한 URL 생성
var components = URLComponents(string: baseURL)
components?.queryItems = [
    URLQueryItem(name: "q", value: city),
    URLQueryItem(name: "appid", value: apiKey)
]

// URLSession으로 비동기 요청
let (data, response) = try await URLSession.shared.data(from: url)
```

### 2. **JSON 디코딩**
```swift
struct WeatherResponse: Codable {
    let name: String
    let main: Main
    
    // CodingKeys로 이름 매핑
    enum CodingKeys: String, CodingKey {
        case feelsLike = "feels_like"
    }
}

let response = try JSONDecoder().decode(WeatherResponse.self, from: data)
```

### 3. **async/await 비동기 처리**
```swift
// MainActor로 UI 업데이트 보장
@MainActor
class WeatherService: ObservableObject {
    func fetchWeather(for city: String) async {
        // 비동기 작업
    }
}

// Task로 비동기 함수 호출
Button("새로고침") {
    Task {
        await weatherService.fetchWeather(for: city)
    }
}
```

### 4. **에러 처리**
```swift
enum WeatherError: LocalizedError {
    case networkError
    case cityNotFound
    
    var errorDescription: String? {
        switch self {
        case .networkError: return "네트워크 연결을 확인해주세요."
        case .cityNotFound: return "해당 도시를 찾을 수 없습니다."
        }
    }
}
```

---

## 🔍 핵심 개념 깊이 이해

### ObservableObject vs @State

```swift
// @State: 단순한 값 타입
@State private var counter = 0

// ObservableObject: 복잡한 비즈니스 로직
@MainActor
class WeatherService: ObservableObject {
    @Published var currentWeather: WeatherData?  // 자동 UI 업데이트
    @Published var isLoading = false
}

// 사용법
@StateObject private var weatherService = WeatherService()
```

### CodingKeys 활용

```swift
struct APIResponse: Codable {
    let userName: String
    let userAge: Int
    
    // API의 snake_case를 Swift의 camelCase로 변환
    enum CodingKeys: String, CodingKey {
        case userName = "user_name"
        case userAge = "user_age"
    }
}
```

### MainActor와 스레드 안전성

```swift
@MainActor  // 이 클래스의 모든 메서드는 메인 스레드에서 실행
class WeatherService: ObservableObject {
    @Published var data: String = ""
    
    func updateData() {
        // UI 업데이트가 안전하게 메인 스레드에서 실행됨
        data = "새 데이터"
    }
}
```

---

## 🚨 자주하는 실수와 해결법

### 1. **API 키 보안**
```swift
// ❌ 코드에 직접 하드코딩
private let apiKey = "abcd1234efgh5678"

// ✅ 환경 변수나 설정 파일 사용
private let apiKey = Bundle.main.object(forInfoDictionaryKey: "API_KEY") as? String ?? ""
```

### 2. **네트워크 에러 처리**
```swift
// ❌ 모든 에러를 같게 처리
catch {
    print("에러 발생")
}

// ✅ 에러 타입별 세분화 처리
catch let error as WeatherError {
    errorMessage = error.localizedDescription
} catch {
    errorMessage = "알 수 없는 오류: \(error.localizedDescription)"
}
```

### 3. **메모리 누수 방지**
```swift
// ❌ 강한 참조 순환
class ViewController {
    let service = WeatherService()
    
    func setupCallbacks() {
        service.onComplete = {
            self.updateUI()  // 강한 참조
        }
    }
}

// ✅ 약한 참조 사용
service.onComplete = { [weak self] in
    self?.updateUI()
}
```

### 4. **Task 취소 처리**
```swift
struct ContentView: View {
    @State private var task: Task<Void, Never>?
    
    var body: some View {
        VStack { ... }
        .onDisappear {
            task?.cancel()  // 뷰가 사라지면 작업 취소
        }
    }
    
    private func loadData() {
        task = Task {
            await weatherService.fetchWeather(for: city)
        }
    }
}
```

---

## 🎯 실습 과제

### 과제 1: 위치 기반 날씨
```swift
import CoreLocation

// 현재 위치의 날씨 자동 로드
@StateObject private var locationManager = LocationManager()

func fetchCurrentLocationWeather() async {
    // 위도/경도로 날씨 API 호출
}
```

### 과제 2: 오프라인 캐싱
```swift
// UserDefaults에 마지막 날씨 데이터 저장
private func cacheWeatherData(_ data: WeatherData) {
    if let encoded = try? JSONEncoder().encode(data) {
        UserDefaults.standard.set(encoded, forKey: "lastWeatherData")
    }
}
```

### 과제 3: 5일 예보 추가
```swift
// 5일 예보 API 엔드포인트 추가
private let forecastURL = "https://api.openweathermap.org/data/2.5/forecast"

struct ForecastResponse: Codable {
    let list: [ForecastItem]
    
    struct ForecastItem: Codable {
        let dt: Int
        let main: Main
        let weather: [Weather]
    }
}
```

---

## 🎉 성공 확인

**기본 테스트**:
- [ ] 앱 시작 시 서울 날씨가 자동 로드됨
- [ ] 다른 도시 선택 시 해당 도시 날씨 표시됨
- [ ] 새로고침 버튼으로 최신 데이터 업데이트됨
- [ ] 네트워크 오류 시 에러 메시지 표시됨

**고급 테스트**:
- [ ] 비행기 모드에서 적절한 에러 메시지
- [ ] 잘못된 도시명 입력 시 404 에러 처리
- [ ] 빠른 도시 변경 시 이전 요청 취소됨
- [ ] 백그라운드에서 포그라운드 복귀 시 데이터 새로고침

**보안 테스트**:
- [ ] API 키가 코드에 하드코딩되지 않음
- [ ] HTTPS 연결만 허용됨
- [ ] 민감한 정보가 로그에 출력되지 않음

---

## 💡 성능 최적화 팁

### 1. **네트워크 요청 최적화**
```swift
// 중복 요청 방지
private var currentTask: Task<Void, Never>?

func fetchWeather(for city: String) async {
    currentTask?.cancel()  // 이전 요청 취소
    
    currentTask = Task {
        // 새 요청 수행
    }
}
```

### 2. **이미지 캐싱**
```swift
// 날씨 아이콘 캐싱
@StateObject private var imageCache = ImageCache()

AsyncImage(url: iconURL) { image in
    image.resizable()
} placeholder: {
    ProgressView()
}
```

### 3. **JSON 파싱 최적화**
```swift
// 필요한 필드만 디코딩
struct WeatherResponse: Codable {
    let name: String
    let main: Main
    // 불필요한 필드는 제외
}
```

---

## 🚀 다음 단계

축하합니다! 이제 실제 API를 사용하는 앱을 만들었습니다.

**L0에서 배운 것**:
- ✅ Hello World에서 실제 앱 구조로 발전
- ✅ 상태 관리와 사용자 상호작용
- ✅ 실제 API 연동과 데이터 처리
- ✅ 에러 처리와 사용자 경험

**다음 단계**:
→ **[L1: WeatherNow 완성](../L1_weather_app/00_project_setup.md)** - 프로덕션 수준의 완전한 앱

---

## 📈 지금까지의 성장

```swift
// 시작점
Text("Hello, World!")

// 현재 위치
struct WeatherApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(WeatherService())
        }
    }
}
```

**이제 여러분은**:
- 실제 API를 사용하는 앱을 만들 수 있습니다
- 네트워킹과 에러 처리를 할 수 있습니다
- 사용자 경험을 고려한 UI를 설계할 수 있습니다
- SwiftUI의 핵심 개념들을 실전에서 활용할 수 있습니다

---

**축하합니다! L0 완료! 이제 진짜 iOS 개발자입니다.**

→ **[L1: 프로덕션 수준 앱 만들기](../L1_weather_app/00_project_setup.md)**

---

*"The best way to predict the future is to create it." - Peter Drucker*