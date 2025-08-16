# L1-01: 날씨 API 완벽 연동
## 실제 데이터로 살아있는 앱 만들기

---

> **"Data is the new oil, but APIs are the refineries."**

이제 실제 OpenWeatherMap API를 완벽히 연동하여 실시간 날씨 데이터를 보여주는 앱을 완성해봅시다.

---

## 🎯 목표

**1시간 후 결과물**:
- 실시간 날씨 데이터 표시
- 에러 처리 및 재시도 로직
- 오프라인 캐싱 시스템

**배울 것**:
- REST API 완벽 연동
- 실전 에러 처리
- 캐싱 전략
- 사용자 경험 최적화

---

## 📋 사전 준비

### API 키 안전하게 설정하기

1. **Xcode에서 Info.plist 열기**
2. **새 키 추가**:
   ```
   Key: OPENWEATHER_API_KEY
   Type: String  
   Value: [발급받은 API 키]
   ```

3. **환경별 설정** (선택사항):
   ```
   OPENWEATHER_API_KEY_DEV: 개발용 키
   OPENWEATHER_API_KEY_PROD: 프로덕션용 키
   ```

⚠️ **보안 주의**: API 키는 절대 코드에 하드코딩하지 마세요!

---

## 🚀 실습: 완벽한 API 연동

### 1단계: 날씨 데이터 모델 정의

**Features/Weather/Models/WeatherModels.swift** 생성:

```swift
import Foundation

// MARK: - API Response Models
struct WeatherResponse: Codable {
    let coord: Coordinates
    let weather: [WeatherCondition]
    let base: String
    let main: MainWeatherData
    let visibility: Int
    let wind: WindData
    let clouds: CloudData
    let dt: Int
    let sys: SystemData
    let timezone: Int
    let id: Int
    let name: String
    let cod: Int
    
    struct Coordinates: Codable {
        let lon: Double
        let lat: Double
    }
    
    struct WeatherCondition: Codable {
        let id: Int
        let main: String
        let description: String
        let icon: String
    }
    
    struct MainWeatherData: Codable {
        let temp: Double
        let feelsLike: Double
        let tempMin: Double
        let tempMax: Double
        let pressure: Int
        let humidity: Int
        let seaLevel: Int?
        let grndLevel: Int?
        
        enum CodingKeys: String, CodingKey {
            case temp, pressure, humidity
            case feelsLike = "feels_like"
            case tempMin = "temp_min"
            case tempMax = "temp_max"
            case seaLevel = "sea_level"
            case grndLevel = "grnd_level"
        }
    }
    
    struct WindData: Codable {
        let speed: Double
        let deg: Int?
        let gust: Double?
    }
    
    struct CloudData: Codable {
        let all: Int
    }
    
    struct SystemData: Codable {
        let type: Int?
        let id: Int?
        let country: String
        let sunrise: Int
        let sunset: Int
    }
}

// MARK: - 5일 예보 Response
struct ForecastResponse: Codable {
    let cod: String
    let message: Int
    let cnt: Int
    let list: [ForecastItem]
    let city: CityInfo
    
    struct ForecastItem: Codable {
        let dt: Int
        let main: WeatherResponse.MainWeatherData
        let weather: [WeatherResponse.WeatherCondition]
        let clouds: WeatherResponse.CloudData
        let wind: WeatherResponse.WindData
        let visibility: Int
        let pop: Double
        let sys: Pod
        let dtTxt: String
        
        struct Pod: Codable {
            let pod: String
        }
        
        enum CodingKeys: String, CodingKey {
            case dt, main, weather, clouds, wind, visibility, pop, sys
            case dtTxt = "dt_txt"
        }
    }
    
    struct CityInfo: Codable {
        let id: Int
        let name: String
        let coord: WeatherResponse.Coordinates
        let country: String
        let population: Int
        let timezone: Int
        let sunrise: Int
        let sunset: Int
    }
}

// MARK: - 앱 내부 데이터 모델
struct WeatherData: Codable, Identifiable, Equatable {
    let id = UUID()
    let cityName: String
    let country: String
    let coordinates: Coordinates
    let temperature: Int
    let feelsLike: Int
    let condition: String
    let conditionDescription: String
    let icon: String
    let humidity: Int
    let pressure: Int
    let windSpeed: Double
    let windDirection: Int?
    let visibility: Int
    let cloudiness: Int
    let sunrise: Date
    let sunset: Date
    let timezone: Int
    let lastUpdated: Date
    
    // 온도 범위
    let minTemperature: Int
    let maxTemperature: Int
    
    struct Coordinates: Codable, Equatable {
        let latitude: Double
        let longitude: Double
    }
    
    // API Response로부터 생성
    init(from response: WeatherResponse) {
        self.cityName = response.name
        self.country = response.sys.country
        self.coordinates = Coordinates(
            latitude: response.coord.lat,
            longitude: response.coord.lon
        )
        self.temperature = Int(response.main.temp.rounded())
        self.feelsLike = Int(response.main.feelsLike.rounded())
        self.condition = response.weather.first?.main ?? "Unknown"
        self.conditionDescription = response.weather.first?.description.capitalized ?? "알 수 없음"
        self.icon = WeatherIconMapper.mapToSFSymbol(response.weather.first?.icon ?? "01d")
        self.humidity = response.main.humidity
        self.pressure = response.main.pressure
        self.windSpeed = response.wind.speed
        self.windDirection = response.wind.deg
        self.visibility = response.visibility
        self.cloudiness = response.clouds.all
        self.sunrise = Date(timeIntervalSince1970: TimeInterval(response.sys.sunrise))
        self.sunset = Date(timeIntervalSince1970: TimeInterval(response.sys.sunset))
        self.timezone = response.timezone
        self.lastUpdated = Date()
        self.minTemperature = Int(response.main.tempMin.rounded())
        self.maxTemperature = Int(response.main.tempMax.rounded())
    }
    
    // 더미 데이터 (테스트용)
    static let sample = WeatherData(
        cityName: "서울",
        country: "KR",
        coordinates: Coordinates(latitude: 37.5665, longitude: 126.9780),
        temperature: 23,
        feelsLike: 25,
        condition: "Clear",
        conditionDescription: "맑음",
        icon: "sun.max.fill",
        humidity: 65,
        pressure: 1013,
        windSpeed: 2.5,
        windDirection: 180,
        visibility: 10000,
        cloudiness: 0,
        sunrise: Date(),
        sunset: Date().addingTimeInterval(12 * 3600),
        timezone: 32400,
        lastUpdated: Date(),
        minTemperature: 18,
        maxTemperature: 28
    )
}

// MARK: - 날씨 아이콘 매퍼
struct WeatherIconMapper {
    static func mapToSFSymbol(_ openWeatherIcon: String) -> String {
        switch openWeatherIcon {
        // 맑음
        case "01d": return "sun.max.fill"
        case "01n": return "moon.stars.fill"
        
        // 약간 흐림
        case "02d": return "cloud.sun.fill"
        case "02n": return "cloud.moon.fill"
        
        // 흐림
        case "03d", "03n": return "cloud.fill"
        case "04d", "04n": return "smoke.fill"
        
        // 소나기
        case "09d", "09n": return "cloud.drizzle.fill"
        
        // 비
        case "10d": return "cloud.sun.rain.fill"
        case "10n": return "cloud.moon.rain.fill"
        
        // 천둥번개
        case "11d", "11n": return "cloud.bolt.rain.fill"
        
        // 눈
        case "13d", "13n": return "cloud.snow.fill"
        
        // 안개
        case "50d", "50n": return "cloud.fog.fill"
        
        default: return "questionmark.circle.fill"
        }
    }
    
    static func getWeatherColor(_ condition: String) -> String {
        switch condition.lowercased() {
        case "clear": return "systemYellow"
        case "clouds": return "systemGray"
        case "rain", "drizzle": return "systemBlue"
        case "thunderstorm": return "systemPurple"
        case "snow": return "systemCyan"
        case "mist", "fog", "haze": return "systemGray2"
        default: return "systemBlue"
        }
    }
}

// MARK: - 예보 데이터
struct ForecastData: Codable, Identifiable {
    let id = UUID()
    let date: Date
    let temperature: Int
    let minTemperature: Int
    let maxTemperature: Int
    let condition: String
    let icon: String
    let humidity: Int
    let windSpeed: Double
    let precipitationProbability: Int
    
    init(from item: ForecastResponse.ForecastItem) {
        self.date = Date(timeIntervalSince1970: TimeInterval(item.dt))
        self.temperature = Int(item.main.temp.rounded())
        self.minTemperature = Int(item.main.tempMin.rounded())
        self.maxTemperature = Int(item.main.tempMax.rounded())
        self.condition = item.weather.first?.main ?? "Unknown"
        self.icon = WeatherIconMapper.mapToSFSymbol(item.weather.first?.icon ?? "01d")
        self.humidity = item.main.humidity
        self.windSpeed = item.wind.speed
        self.precipitationProbability = Int((item.pop * 100).rounded())
    }
}
```

### 2단계: 날씨 서비스 구현

**Features/Weather/Services/WeatherService.swift** 생성:

```swift
import Foundation
import CoreData

// MARK: - Weather Service
@MainActor
class WeatherService: ObservableObject {
    // Published Properties
    @Published var currentWeather: WeatherData?
    @Published var forecast: [ForecastData] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    @Published var lastUpdateTime: Date?
    
    // Private Properties
    private let httpClient = HTTPClient.shared
    private let cacheManager = WeatherCacheManager.shared
    private let locationService = LocationService.shared
    
    // Configuration
    private let cacheValidityDuration: TimeInterval = 10 * 60 // 10분
    
    // MARK: - Public Methods
    
    /// 도시명으로 현재 날씨 조회
    func fetchCurrentWeather(for city: String, forceRefresh: Bool = false) async {
        await performWeatherFetch {
            try await self.fetchWeatherFromAPI(city: city, forceRefresh: forceRefresh)
        }
    }
    
    /// 좌표로 현재 날씨 조회
    func fetchCurrentWeather(latitude: Double, longitude: Double, forceRefresh: Bool = false) async {
        await performWeatherFetch {
            try await self.fetchWeatherFromAPI(
                latitude: latitude,
                longitude: longitude,
                forceRefresh: forceRefresh
            )
        }
    }
    
    /// 현재 위치 날씨 조회
    func fetchCurrentLocationWeather() async {
        guard let location = await locationService.getCurrentLocation() else {
            errorMessage = "위치 정보를 가져올 수 없습니다."
            return
        }
        
        await fetchCurrentWeather(
            latitude: location.coordinate.latitude,
            longitude: location.coordinate.longitude
        )
    }
    
    /// 5일 예보 조회
    func fetchForecast(for city: String) async {
        do {
            guard let url = APIConfig.Endpoint.forecast(city: city).url else {
                throw NetworkError.invalidURL
            }
            
            let response = try await httpClient.fetch(ForecastResponse.self, from: url)
            let forecastData = response.list.map { ForecastData(from: $0) }
            
            self.forecast = forecastData
            cacheManager.cacheForecast(forecastData, for: city)
            
        } catch {
            handleError(error)
        }
    }
    
    /// 캐시 새로고침
    func refreshWeather() async {
        guard let weather = currentWeather else { return }
        await fetchCurrentWeather(for: weather.cityName, forceRefresh: true)
    }
    
    /// 오프라인 데이터 로드
    func loadCachedWeather(for city: String) {
        if let cachedWeather = cacheManager.getCachedWeather(for: city),
           Date().timeIntervalSince(cachedWeather.lastUpdated) < cacheValidityDuration {
            currentWeather = cachedWeather
            lastUpdateTime = cachedWeather.lastUpdated
        }
    }
    
    // MARK: - Private Methods
    
    private func performWeatherFetch(operation: @escaping () async throws -> WeatherData) async {
        isLoading = true
        errorMessage = nil
        
        do {
            let weatherData = try await operation()
            currentWeather = weatherData
            lastUpdateTime = Date()
            
            // 해당 도시의 예보도 자동으로 가져오기
            await fetchForecast(for: weatherData.cityName)
            
        } catch {
            handleError(error)
        }
        
        isLoading = false
    }
    
    private func fetchWeatherFromAPI(city: String, forceRefresh: Bool) async throws -> WeatherData {
        // 캐시 확인 (강제 새로고침이 아닌 경우)
        if !forceRefresh,
           let cachedWeather = cacheManager.getCachedWeather(for: city),
           Date().timeIntervalSince(cachedWeather.lastUpdated) < cacheValidityDuration {
            return cachedWeather
        }
        
        // API 호출
        guard let url = APIConfig.Endpoint.currentWeather(city: city).url else {
            throw NetworkError.invalidURL
        }
        
        let response = try await httpClient.fetch(WeatherResponse.self, from: url)
        let weatherData = WeatherData(from: response)
        
        // 캐시 저장
        cacheManager.cacheWeather(weatherData)
        
        return weatherData
    }
    
    private func fetchWeatherFromAPI(latitude: Double, longitude: Double, forceRefresh: Bool) async throws -> WeatherData {
        // 좌표 기반 캐시 키 생성
        let cacheKey = "\(latitude),\(longitude)"
        
        if !forceRefresh,
           let cachedWeather = cacheManager.getCachedWeather(for: cacheKey),
           Date().timeIntervalSince(cachedWeather.lastUpdated) < cacheValidityDuration {
            return cachedWeather
        }
        
        guard let url = APIConfig.Endpoint.weatherByCoordinates(lat: latitude, lon: longitude).url else {
            throw NetworkError.invalidURL
        }
        
        let response = try await httpClient.fetch(WeatherResponse.self, from: url)
        let weatherData = WeatherData(from: response)
        
        cacheManager.cacheWeather(weatherData, key: cacheKey)
        
        return weatherData
    }
    
    private func handleError(_ error: Error) {
        if let networkError = error as? NetworkError {
            errorMessage = networkError.localizedDescription
        } else {
            errorMessage = "예상치 못한 오류가 발생했습니다: \(error.localizedDescription)"
        }
        
        // 오프라인인 경우 캐시된 데이터 로드 시도
        if case .networkError = error as? NetworkError {
            // 마지막으로 조회한 도시의 캐시 데이터 로드
            if let lastCity = UserDefaults.standard.string(forKey: "lastSearchedCity") {
                loadCachedWeather(for: lastCity)
            }
        }
    }
}

// MARK: - Weather Cache Manager
class WeatherCacheManager {
    static let shared = WeatherCacheManager()
    
    private let userDefaults = UserDefaults.standard
    private let weatherCacheKey = "cachedWeatherData"
    private let forecastCacheKey = "cachedForecastData"
    
    private init() {}
    
    func cacheWeather(_ weather: WeatherData, key: String? = nil) {
        let cacheKey = key ?? weather.cityName
        
        do {
            let data = try JSONEncoder().encode(weather)
            userDefaults.set(data, forKey: "\(weatherCacheKey)_\(cacheKey)")
            userDefaults.set(cacheKey, forKey: "lastSearchedCity")
        } catch {
            print("날씨 데이터 캐싱 실패: \(error)")
        }
    }
    
    func getCachedWeather(for key: String) -> WeatherData? {
        guard let data = userDefaults.data(forKey: "\(weatherCacheKey)_\(key)") else {
            return nil
        }
        
        do {
            return try JSONDecoder().decode(WeatherData.self, from: data)
        } catch {
            print("캐시된 날씨 데이터 로드 실패: \(error)")
            return nil
        }
    }
    
    func cacheForecast(_ forecast: [ForecastData], for city: String) {
        do {
            let data = try JSONEncoder().encode(forecast)
            userDefaults.set(data, forKey: "\(forecastCacheKey)_\(city)")
        } catch {
            print("예보 데이터 캐싱 실패: \(error)")
        }
    }
    
    func getCachedForecast(for city: String) -> [ForecastData]? {
        guard let data = userDefaults.data(forKey: "\(forecastCacheKey)_\(city)") else {
            return nil
        }
        
        do {
            return try JSONDecoder().decode([ForecastData].self, from: data)
        } catch {
            print("캐시된 예보 데이터 로드 실패: \(error)")
            return nil
        }
    }
    
    func clearCache() {
        let keys = userDefaults.dictionaryRepresentation().keys
        for key in keys {
            if key.contains(weatherCacheKey) || key.contains(forecastCacheKey) {
                userDefaults.removeObject(forKey: key)
            }
        }
    }
}

// MARK: - Location Service
import CoreLocation

class LocationService: NSObject, ObservableObject {
    static let shared = LocationService()
    
    @Published var authorizationStatus: CLAuthorizationStatus = .notDetermined
    @Published var currentLocation: CLLocation?
    
    private let locationManager = CLLocationManager()
    
    override init() {
        super.init()
        setupLocationManager()
    }
    
    private func setupLocationManager() {
        locationManager.delegate = self
        locationManager.desiredAccuracy = kCLLocationAccuracyBest
        locationManager.distanceFilter = 1000 // 1km
    }
    
    func requestLocationPermission() {
        locationManager.requestWhenInUseAuthorization()
    }
    
    func getCurrentLocation() async -> CLLocation? {
        guard authorizationStatus == .authorizedWhenInUse || authorizationStatus == .authorizedAlways else {
            return nil
        }
        
        return await withCheckedContinuation { continuation in
            locationManager.requestLocation()
            
            DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
                continuation.resume(returning: self.currentLocation)
            }
        }
    }
}

// MARK: - CLLocationManagerDelegate
extension LocationService: CLLocationManagerDelegate {
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        currentLocation = locations.last
    }
    
    func locationManager(_ manager: CLLocationManager, didFailWithError error: Error) {
        print("위치 조회 실패: \(error)")
    }
    
    func locationManagerDidChangeAuthorization(_ manager: CLLocationManager) {
        DispatchQueue.main.async {
            self.authorizationStatus = manager.authorizationStatus
        }
    }
}
```

### 3단계: 실시간 날씨 UI 구현

**Features/Weather/Views/WeatherView.swift** 생성:

```swift
import SwiftUI

struct WeatherView: View {
    @StateObject private var weatherService = WeatherService()
    @StateObject private var locationService = LocationService.shared
    
    @State private var selectedCity = "Seoul"
    @State private var showingCitySelection = false
    
    // 한국 주요 도시
    private let koreanCities = [
        "Seoul": "서울",
        "Busan": "부산",
        "Daegu": "대구",
        "Incheon": "인천",
        "Gwangju": "광주",
        "Daejeon": "대전",
        "Ulsan": "울산",
        "Sejong": "세종",
        "Suwon": "수원",
        "Goyang": "고양"
    ]
    
    var body: some View {
        NavigationStack {
            ScrollView {
                VStack(spacing: 20) {
                    // 위치 및 도시 선택
                    locationHeader
                    
                    // 메인 날씨 카드
                    if weatherService.isLoading {
                        loadingView
                    } else if let error = weatherService.errorMessage {
                        errorView(error)
                    } else if let weather = weatherService.currentWeather {
                        currentWeatherCard(weather)
                        
                        // 상세 정보
                        weatherDetailsGrid(weather)
                        
                        // 5일 예보
                        if !weatherService.forecast.isEmpty {
                            forecastSection
                        }
                    } else {
                        emptyStateView
                    }
                }
                .padding()
            }
            .navigationTitle("WeatherNow")
            .navigationBarTitleDisplayMode(.large)
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Menu {
                        Button("현재 위치", systemImage: "location") {
                            Task {
                                await weatherService.fetchCurrentLocationWeather()
                            }
                        }
                        
                        Button("도시 선택", systemImage: "list.bullet") {
                            showingCitySelection = true
                        }
                        
                        Button("새로고침", systemImage: "arrow.clockwise") {
                            Task {
                                await weatherService.refreshWeather()
                            }
                        }
                    } label: {
                        Image(systemName: "ellipsis.circle")
                    }
                }
            }
            .sheet(isPresented: $showingCitySelection) {
                CitySelectionView(
                    cities: koreanCities,
                    selectedCity: $selectedCity
                ) { city in
                    Task {
                        await weatherService.fetchCurrentWeather(for: city)
                    }
                }
            }
            .task {
                // 앱 시작 시 위치 권한 요청
                if locationService.authorizationStatus == .notDetermined {
                    locationService.requestLocationPermission()
                }
                
                // 기본 도시 날씨 로드
                await weatherService.fetchCurrentWeather(for: selectedCity)
            }
            .refreshable {
                await weatherService.refreshWeather()
            }
        }
    }
    
    // MARK: - View Components
    
    private var locationHeader: some View {
        HStack {
            VStack(alignment: .leading) {
                Text("현재 위치")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
                
                if let weather = weatherService.currentWeather {
                    HStack {
                        Text(weather.cityName)
                            .font(.title2)
                            .fontWeight(.semibold)
                        
                        Text(weather.country)
                            .font(.caption)
                            .padding(.horizontal, 8)
                            .padding(.vertical, 2)
                            .background(.secondary.opacity(0.2))
                            .clipShape(Capsule())
                    }
                }
            }
            
            Spacer()
            
            if let lastUpdate = weatherService.lastUpdateTime {
                VStack(alignment: .trailing) {
                    Text("업데이트")
                        .font(.caption2)
                        .foregroundColor(.secondary)
                    Text(lastUpdate, style: .time)
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
            }
        }
        .padding()
        .background(.regularMaterial)
        .clipShape(RoundedRectangle(cornerRadius: 12))
    }
    
    private var loadingView: some View {
        VStack(spacing: 20) {
            ProgressView()
                .scaleEffect(1.5)
                .tint(.blue)
            
            Text("날씨 정보 로딩 중...")
                .font(.subheadline)
                .foregroundColor(.secondary)
        }
        .frame(height: 200)
    }
    
    private func errorView(_ message: String) -> some View {
        VStack(spacing: 20) {
            Image(systemName: "exclamationmark.triangle.fill")
                .font(.system(size: 50))
                .foregroundColor(.red)
            
            Text("오류 발생")
                .font(.headline)
            
            Text(message)
                .font(.subheadline)
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)
            
            Button("다시 시도") {
                Task {
                    await weatherService.fetchCurrentWeather(for: selectedCity)
                }
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
        .background(.regularMaterial)
        .clipShape(RoundedRectangle(cornerRadius: 16))
    }
    
    private var emptyStateView: some View {
        VStack(spacing: 20) {
            Image(systemName: "cloud.sun.fill")
                .font(.system(size: 50))
                .foregroundColor(.blue)
            
            Text("날씨 정보를 불러오세요")
                .font(.headline)
            
            Button("날씨 보기") {
                Task {
                    await weatherService.fetchCurrentWeather(for: selectedCity)
                }
            }
            .buttonStyle(.borderedProminent)
        }
        .frame(height: 200)
    }
    
    private func currentWeatherCard(_ weather: WeatherData) -> some View {
        VStack(spacing: 20) {
            // 온도와 아이콘
            HStack {
                VStack(alignment: .leading) {
                    Text("\(weather.temperature)°C")
                        .font(.system(size: 72, weight: .thin))
                    
                    Text(weather.conditionDescription)
                        .font(.title3)
                        .foregroundColor(.secondary)
                }
                
                Spacer()
                
                Image(systemName: weather.icon)
                    .font(.system(size: 60))
                    .foregroundColor(.orange)
                    .symbolRenderingMode(.multicolor)
            }
            
            // 최고/최저 온도
            HStack {
                HStack {
                    Image(systemName: "thermometer.sun")
                        .foregroundColor(.red)
                    Text("최고 \(weather.maxTemperature)°C")
                }
                
                Spacer()
                
                HStack {
                    Image(systemName: "thermometer.snowflake")
                        .foregroundColor(.blue)
                    Text("최저 \(weather.minTemperature)°C")
                }
            }
            .font(.subheadline)
            .foregroundColor(.secondary)
        }
        .padding(24)
        .background(.regularMaterial)
        .clipShape(RoundedRectangle(cornerRadius: 20))
        .shadow(color: .black.opacity(0.1), radius: 10, x: 0, y: 5)
    }
    
    private func weatherDetailsGrid(_ weather: WeatherData) -> some View {
        LazyVGrid(columns: Array(repeating: GridItem(.flexible()), count: 2), spacing: 16) {
            DetailCard(
                icon: "humidity",
                title: "습도",
                value: "\(weather.humidity)%",
                color: .blue
            )
            
            DetailCard(
                icon: "barometer",
                title: "기압",
                value: "\(weather.pressure)hPa",
                color: .purple
            )
            
            DetailCard(
                icon: "wind",
                title: "바람",
                value: String(format: "%.1fm/s", weather.windSpeed),
                color: .green
            )
            
            DetailCard(
                icon: "eye",
                title: "가시거리",
                value: "\(weather.visibility/1000)km",
                color: .orange
            )
            
            DetailCard(
                icon: "sunrise",
                title: "일출",
                value: weather.sunrise.formatted(date: .omitted, time: .shortened),
                color: .yellow
            )
            
            DetailCard(
                icon: "sunset",
                title: "일몰",
                value: weather.sunset.formatted(date: .omitted, time: .shortened),
                color: .red
            )
        }
    }
    
    private var forecastSection: some View {
        VStack(alignment: .leading, spacing: 16) {
            Text("5일 예보")
                .font(.headline)
                .padding(.horizontal)
            
            ScrollView(.horizontal, showsIndicators: false) {
                HStack(spacing: 16) {
                    ForEach(weatherService.forecast.prefix(5)) { forecast in
                        ForecastCard(forecast: forecast)
                    }
                }
                .padding(.horizontal)
            }
        }
    }
}

// MARK: - Supporting Views
struct DetailCard: View {
    let icon: String
    let title: String
    let value: String
    let color: Color
    
    var body: some View {
        VStack(spacing: 8) {
            Image(systemName: icon)
                .font(.title2)
                .foregroundColor(color)
            
            Text(title)
                .font(.caption)
                .foregroundColor(.secondary)
            
            Text(value)
                .font(.subheadline)
                .fontWeight(.semibold)
        }
        .frame(maxWidth: .infinity)
        .padding()
        .background(.regularMaterial)
        .clipShape(RoundedRectangle(cornerRadius: 12))
    }
}

struct ForecastCard: View {
    let forecast: ForecastData
    
    var body: some View {
        VStack(spacing: 8) {
            Text(forecast.date, format: .dateTime.weekday(.abbreviated))
                .font(.caption)
                .foregroundColor(.secondary)
            
            Image(systemName: forecast.icon)
                .font(.title2)
                .foregroundColor(.orange)
                .symbolRenderingMode(.multicolor)
            
            Text("\(forecast.temperature)°")
                .font(.headline)
            
            if forecast.precipitationProbability > 0 {
                Text("\(forecast.precipitationProbability)%")
                    .font(.caption2)
                    .foregroundColor(.blue)
            }
        }
        .frame(width: 80)
        .padding()
        .background(.regularMaterial)
        .clipShape(RoundedRectangle(cornerRadius: 12))
    }
}

struct CitySelectionView: View {
    let cities: [String: String]
    @Binding var selectedCity: String
    let onCitySelected: (String) -> Void
    
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        NavigationStack {
            List {
                ForEach(Array(cities.keys).sorted(), id: \.self) { cityKey in
                    Button {
                        selectedCity = cityKey
                        onCitySelected(cityKey)
                        dismiss()
                    } label: {
                        HStack {
                            Text(cities[cityKey] ?? cityKey)
                                .foregroundColor(.primary)
                            
                            Spacer()
                            
                            if cityKey == selectedCity {
                                Image(systemName: "checkmark")
                                    .foregroundColor(.blue)
                            }
                        }
                    }
                }
            }
            .navigationTitle("도시 선택")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("닫기") {
                        dismiss()
                    }
                }
            }
        }
    }
}

// MARK: - Preview
#Preview {
    WeatherView()
}
```

---

## 🎯 여기서 배운 것

### 1. **완벽한 API 연동**
- 실시간 데이터 처리
- 에러 처리 및 재시도
- 응답 검증 및 매핑

### 2. **캐싱 시스템**
- 오프라인 지원
- 데이터 유효성 검증
- 성능 최적화

### 3. **위치 기반 서비스**
- Core Location 통합
- 권한 처리
- 좌표 기반 날씨

### 4. **사용자 경험**
- 로딩 상태
- 에러 메시지
- Pull-to-refresh

---

## 🎉 성공 확인

**기본 기능 테스트**:
- [ ] 실시간 날씨 데이터 표시
- [ ] 도시 변경 시 데이터 업데이트
- [ ] 5일 예보 표시
- [ ] 위치 기반 날씨 조회

**고급 기능 테스트**:
- [ ] 오프라인 모드에서 캐시 데이터 표시
- [ ] 에러 상황에서 적절한 메시지
- [ ] Pull-to-refresh로 데이터 갱신
- [ ] 백그라운드 복귀 시 자동 갱신

---

## 🚀 다음 단계

축하합니다! 실제 API와 연동된 완전한 날씨 앱을 만들었습니다.

**완성된 기능**:
- ✅ 실시간 날씨 API 연동
- ✅ 오프라인 캐싱 시스템
- ✅ 위치 기반 서비스
- ✅ 5일 예보
- ✅ 완전한 에러 처리

**다음에 할 것**:
→ **[02. 위치 기반 서비스](02_location_service.md)** - 정밀한 위치 추적과 알림

---

**축하합니다! 이제 프로덕션 수준의 날씨 앱 코어가 완성되었습니다.**

→ **[다음: 위치 기반 서비스 완성](02_location_service.md)**

---

*"The best APIs are invisible to the user, but essential to the experience."*