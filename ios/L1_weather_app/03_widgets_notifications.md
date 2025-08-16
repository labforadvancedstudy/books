# L1-03: 위젯과 알림 시스템
## 사용자와 24시간 연결되는 앱

---

> **"The best app is the one users don't need to open."**

위젯과 알림으로 사용자가 앱을 열지 않아도 날씨 정보를 받을 수 있게 만들어봅시다.

---

## 🎯 목표

**완성 후 결과물**:
- 홈 화면 위젯 (소형/중형/대형)
- 잠금 화면 위젯
- Live Activities
- 푸시 알림 시스템

---

## 🚀 실습: 위젯 구현

### 1단계: Widget Extension 추가

**File → New → Target → Widget Extension**

```
Product Name: WeatherWidget
Include Live Activity: ✅
Include Configuration Intent: ✅
```

### 2단계: 위젯 Provider 구현

**WeatherWidget/WeatherWidget.swift**:

```swift
import WidgetKit
import SwiftUI
import Intents

// MARK: - Widget Entry
struct WeatherEntry: TimelineEntry {
    let date: Date
    let configuration: ConfigurationIntent
    let weather: WeatherData?
    let forecast: [ForecastData]
    let locationName: String
    let isPlaceholder: Bool
    
    static var placeholder: WeatherEntry {
        WeatherEntry(
            date: Date(),
            configuration: ConfigurationIntent(),
            weather: WeatherData.sample,
            forecast: [],
            locationName: "서울",
            isPlaceholder: true
        )
    }
    
    static var snapshot: WeatherEntry {
        WeatherEntry(
            date: Date(),
            configuration: ConfigurationIntent(),
            weather: WeatherData.sample,
            forecast: [
                ForecastData.sample,
                ForecastData.sample,
                ForecastData.sample
            ],
            locationName: "서울",
            isPlaceholder: false
        )
    }
}

// MARK: - Timeline Provider
struct WeatherProvider: IntentTimelineProvider {
    func placeholder(in context: Context) -> WeatherEntry {
        WeatherEntry.placeholder
    }
    
    func getSnapshot(for configuration: ConfigurationIntent, in context: Context, completion: @escaping (WeatherEntry) -> ()) {
        if context.isPreview {
            completion(WeatherEntry.snapshot)
        } else {
            Task {
                let entry = await fetchWeatherEntry(for: configuration)
                completion(entry)
            }
        }
    }
    
    func getTimeline(for configuration: ConfigurationIntent, in context: Context, completion: @escaping (Timeline<WeatherEntry>) -> ()) {
        Task {
            let entry = await fetchWeatherEntry(for: configuration)
            
            // 30분마다 업데이트
            let nextUpdate = Calendar.current.date(byAdding: .minute, value: 30, to: Date())!
            let timeline = Timeline(entries: [entry], policy: .after(nextUpdate))
            
            completion(timeline)
        }
    }
    
    private func fetchWeatherEntry(for configuration: ConfigurationIntent) async -> WeatherEntry {
        // 위치 설정 확인
        let locationName = configuration.location?.displayString ?? "현재 위치"
        
        // API 호출
        guard let url = APIConfig.Endpoint.currentWeather(city: locationName).url else {
            return WeatherEntry(
                date: Date(),
                configuration: configuration,
                weather: nil,
                forecast: [],
                locationName: locationName,
                isPlaceholder: false
            )
        }
        
        do {
            let weatherResponse = try await HTTPClient.shared.fetch(WeatherResponse.self, from: url)
            let weatherData = WeatherData(from: weatherResponse)
            
            // 예보 데이터도 가져오기
            var forecast: [ForecastData] = []
            if let forecastURL = APIConfig.Endpoint.forecast(city: locationName).url {
                let forecastResponse = try await HTTPClient.shared.fetch(ForecastResponse.self, from: forecastURL)
                forecast = forecastResponse.list.prefix(5).map { ForecastData(from: $0) }
            }
            
            return WeatherEntry(
                date: Date(),
                configuration: configuration,
                weather: weatherData,
                forecast: forecast,
                locationName: locationName,
                isPlaceholder: false
            )
        } catch {
            print("위젯 데이터 로드 실패: \(error)")
            return WeatherEntry(
                date: Date(),
                configuration: configuration,
                weather: nil,
                forecast: [],
                locationName: locationName,
                isPlaceholder: false
            )
        }
    }
}

// MARK: - Widget Views

struct WeatherWidgetEntryView: View {
    var entry: WeatherProvider.Entry
    @Environment(\.widgetFamily) var family
    
    var body: some View {
        switch family {
        case .systemSmall:
            SmallWeatherWidget(entry: entry)
        case .systemMedium:
            MediumWeatherWidget(entry: entry)
        case .systemLarge:
            LargeWeatherWidget(entry: entry)
        case .systemExtraLarge:
            ExtraLargeWeatherWidget(entry: entry)
        case .accessoryCircular:
            CircularWeatherWidget(entry: entry)
        case .accessoryRectangular:
            RectangularWeatherWidget(entry: entry)
        case .accessoryInline:
            InlineWeatherWidget(entry: entry)
        @unknown default:
            SmallWeatherWidget(entry: entry)
        }
    }
}

// MARK: - Small Widget
struct SmallWeatherWidget: View {
    let entry: WeatherProvider.Entry
    
    var body: some View {
        if let weather = entry.weather {
            ZStack {
                // 배경 그라데이션
                LinearGradient(
                    colors: weatherGradientColors(for: weather.condition),
                    startPoint: .top,
                    endPoint: .bottom
                )
                
                VStack(alignment: .leading, spacing: 8) {
                    // 위치
                    HStack {
                        Image(systemName: "location.fill")
                            .font(.caption2)
                        Text(entry.locationName)
                            .font(.caption)
                            .lineLimit(1)
                    }
                    .foregroundColor(.white.opacity(0.8))
                    
                    Spacer()
                    
                    // 온도
                    Text("\(weather.temperature)°")
                        .font(.system(size: 44, weight: .light))
                        .foregroundColor(.white)
                    
                    // 날씨 상태
                    HStack {
                        Image(systemName: weather.icon)
                            .font(.caption)
                        Text(weather.conditionDescription)
                            .font(.caption)
                            .lineLimit(1)
                    }
                    .foregroundColor(.white.opacity(0.9))
                    
                    // 최고/최저
                    HStack(spacing: 8) {
                        Text("H:\(weather.maxTemperature)°")
                        Text("L:\(weather.minTemperature)°")
                    }
                    .font(.caption2)
                    .foregroundColor(.white.opacity(0.8))
                }
                .padding()
            }
            .widgetURL(URL(string: "weathernow://weather?city=\(entry.locationName)"))
        } else {
            EmptyWeatherWidget()
        }
    }
    
    private func weatherGradientColors(for condition: String) -> [Color] {
        switch condition.lowercased() {
        case "clear":
            return [Color(hex: "4FC3F7"), Color(hex: "29B6F6")]
        case "clouds":
            return [Color(hex: "90A4AE"), Color(hex: "607D8B")]
        case "rain", "drizzle":
            return [Color(hex: "5C6BC0"), Color(hex: "3F51B5")]
        case "snow":
            return [Color(hex: "E1F5FE"), Color(hex: "B3E5FC")]
        case "thunderstorm":
            return [Color(hex: "7E57C2"), Color(hex: "5E35B1")]
        default:
            return [Color(hex: "42A5F5"), Color(hex: "1E88E5")]
        }
    }
}

// MARK: - Medium Widget
struct MediumWeatherWidget: View {
    let entry: WeatherProvider.Entry
    
    var body: some View {
        if let weather = entry.weather {
            ZStack {
                LinearGradient(
                    colors: [Color.blue.opacity(0.6), Color.blue.opacity(0.2)],
                    startPoint: .topLeading,
                    endPoint: .bottomTrailing
                )
                
                HStack(spacing: 20) {
                    // 왼쪽: 현재 날씨
                    VStack(alignment: .leading, spacing: 8) {
                        HStack {
                            Image(systemName: "location.fill")
                                .font(.caption2)
                            Text(entry.locationName)
                                .font(.caption)
                        }
                        .foregroundColor(.white.opacity(0.8))
                        
                        Spacer()
                        
                        Image(systemName: weather.icon)
                            .font(.title)
                            .foregroundColor(.white)
                            .symbolRenderingMode(.multicolor)
                        
                        Text("\(weather.temperature)°")
                            .font(.system(size: 36, weight: .medium))
                            .foregroundColor(.white)
                        
                        Text(weather.conditionDescription)
                            .font(.caption)
                            .foregroundColor(.white.opacity(0.9))
                    }
                    .frame(maxWidth: .infinity, alignment: .leading)
                    
                    Divider()
                        .background(Color.white.opacity(0.3))
                    
                    // 오른쪽: 시간별 예보
                    VStack(alignment: .leading, spacing: 4) {
                        Text("시간별")
                            .font(.caption2)
                            .foregroundColor(.white.opacity(0.8))
                        
                        ForEach(entry.forecast.prefix(4)) { forecast in
                            HStack {
                                Text(forecast.date, format: .dateTime.hour())
                                    .font(.caption2)
                                    .frame(width: 30, alignment: .leading)
                                
                                Image(systemName: forecast.icon)
                                    .font(.caption)
                                    .frame(width: 20)
                                
                                Text("\(forecast.temperature)°")
                                    .font(.caption)
                                    .frame(width: 30, alignment: .trailing)
                            }
                            .foregroundColor(.white)
                        }
                    }
                    .frame(maxWidth: .infinity, alignment: .leading)
                }
                .padding()
            }
            .widgetURL(URL(string: "weathernow://forecast"))
        } else {
            EmptyWeatherWidget()
        }
    }
}

// MARK: - Large Widget
struct LargeWeatherWidget: View {
    let entry: WeatherProvider.Entry
    
    var body: some View {
        if let weather = entry.weather {
            ZStack {
                LinearGradient(
                    colors: [Color.blue.opacity(0.5), Color.cyan.opacity(0.2)],
                    startPoint: .top,
                    endPoint: .bottom
                )
                
                VStack(spacing: 16) {
                    // 헤더
                    HStack {
                        VStack(alignment: .leading) {
                            Text(entry.locationName)
                                .font(.headline)
                            Text(Date(), style: .date)
                                .font(.caption)
                                .foregroundColor(.white.opacity(0.8))
                        }
                        Spacer()
                        Image(systemName: weather.icon)
                            .font(.largeTitle)
                            .symbolRenderingMode(.multicolor)
                    }
                    .foregroundColor(.white)
                    
                    // 현재 날씨
                    HStack(alignment: .top) {
                        Text("\(weather.temperature)°")
                            .font(.system(size: 72, weight: .thin))
                        
                        VStack(alignment: .leading) {
                            Text(weather.conditionDescription)
                                .font(.title3)
                            HStack {
                                Text("H:\(weather.maxTemperature)°")
                                Text("L:\(weather.minTemperature)°")
                            }
                            .font(.caption)
                        }
                        .padding(.top, 8)
                        
                        Spacer()
                    }
                    .foregroundColor(.white)
                    
                    // 상세 정보
                    LazyVGrid(columns: [GridItem(.flexible()), GridItem(.flexible()), GridItem(.flexible())]) {
                        WeatherDetailItem(icon: "humidity", title: "습도", value: "\(weather.humidity)%")
                        WeatherDetailItem(icon: "wind", title: "풍속", value: String(format: "%.1fm/s", weather.windSpeed))
                        WeatherDetailItem(icon: "gauge", title: "기압", value: "\(weather.pressure)hPa")
                        WeatherDetailItem(icon: "eye", title: "가시거리", value: "\(weather.visibility/1000)km")
                        WeatherDetailItem(icon: "sunrise", title: "일출", value: weather.sunrise.formatted(date: .omitted, time: .shortened))
                        WeatherDetailItem(icon: "sunset", title: "일몰", value: weather.sunset.formatted(date: .omitted, time: .shortened))
                    }
                    
                    Divider()
                        .background(Color.white.opacity(0.3))
                    
                    // 5일 예보
                    VStack(alignment: .leading, spacing: 8) {
                        Text("5일 예보")
                            .font(.caption)
                            .foregroundColor(.white.opacity(0.8))
                        
                        ForEach(entry.forecast.prefix(5)) { forecast in
                            HStack {
                                Text(forecast.date, format: .dateTime.weekday(.abbreviated))
                                    .frame(width: 40, alignment: .leading)
                                
                                Image(systemName: forecast.icon)
                                    .frame(width: 30)
                                    .symbolRenderingMode(.multicolor)
                                
                                Spacer()
                                
                                if forecast.precipitationProbability > 0 {
                                    HStack(spacing: 2) {
                                        Image(systemName: "drop.fill")
                                            .font(.caption2)
                                            .foregroundColor(.blue)
                                        Text("\(forecast.precipitationProbability)%")
                                            .font(.caption2)
                                    }
                                }
                                
                                Text("\(forecast.minTemperature)° / \(forecast.maxTemperature)°")
                                    .frame(width: 70, alignment: .trailing)
                            }
                            .font(.caption)
                            .foregroundColor(.white)
                        }
                    }
                }
                .padding()
            }
            .widgetURL(URL(string: "weathernow://home"))
        } else {
            EmptyWeatherWidget()
        }
    }
}

// MARK: - Lock Screen Widgets

struct CircularWeatherWidget: View {
    let entry: WeatherProvider.Entry
    
    var body: some View {
        if let weather = entry.weather {
            ZStack {
                AccessoryWidgetBackground()
                VStack(spacing: 2) {
                    Image(systemName: weather.icon)
                        .font(.title3)
                    Text("\(weather.temperature)°")
                        .font(.caption)
                        .fontWeight(.semibold)
                }
            }
        } else {
            ZStack {
                AccessoryWidgetBackground()
                Image(systemName: "cloud.fill")
            }
        }
    }
}

struct RectangularWeatherWidget: View {
    let entry: WeatherProvider.Entry
    
    var body: some View {
        if let weather = entry.weather {
            VStack(alignment: .leading, spacing: 4) {
                HStack {
                    Image(systemName: weather.icon)
                    Text("\(weather.temperature)°")
                        .font(.headline)
                }
                Text(weather.conditionDescription)
                    .font(.caption)
                Text("H:\(weather.maxTemperature)° L:\(weather.minTemperature)°")
                    .font(.caption2)
            }
        } else {
            Text("날씨 정보 없음")
                .font(.caption)
        }
    }
}

struct InlineWeatherWidget: View {
    let entry: WeatherProvider.Entry
    
    var body: some View {
        if let weather = entry.weather {
            Text("\(weather.temperature)° \(weather.conditionDescription)")
        } else {
            Text("날씨 정보를 불러오는 중...")
        }
    }
}

// MARK: - Supporting Views

struct WeatherDetailItem: View {
    let icon: String
    let title: String
    let value: String
    
    var body: some View {
        VStack(spacing: 4) {
            Image(systemName: icon)
                .font(.caption)
            Text(title)
                .font(.caption2)
            Text(value)
                .font(.caption)
                .fontWeight(.semibold)
        }
        .foregroundColor(.white)
    }
}

struct EmptyWeatherWidget: View {
    var body: some View {
        ZStack {
            LinearGradient(
                colors: [Color.gray.opacity(0.3), Color.gray.opacity(0.1)],
                startPoint: .top,
                endPoint: .bottom
            )
            
            VStack {
                Image(systemName: "cloud.fill")
                    .font(.largeTitle)
                    .foregroundColor(.white.opacity(0.5))
                Text("날씨 정보 없음")
                    .font(.caption)
                    .foregroundColor(.white.opacity(0.5))
            }
        }
    }
}

// MARK: - Widget Configuration

@main
struct WeatherWidgetBundle: WidgetBundle {
    var body: some Widget {
        WeatherWidget()
        WeatherLiveActivity()
    }
}

struct WeatherWidget: Widget {
    let kind: String = "WeatherWidget"
    
    var body: some WidgetConfiguration {
        IntentConfiguration(
            kind: kind,
            intent: ConfigurationIntent.self,
            provider: WeatherProvider()
        ) { entry in
            WeatherWidgetEntryView(entry: entry)
        }
        .configurationDisplayName("날씨 위젯")
        .description("현재 날씨와 예보를 한눈에 확인하세요")
        .supportedFamilies([
            .systemSmall,
            .systemMedium,
            .systemLarge,
            .systemExtraLarge,
            .accessoryCircular,
            .accessoryRectangular,
            .accessoryInline
        ])
    }
}
```

### 3단계: Live Activities 구현

**WeatherWidget/WeatherLiveActivity.swift**:

```swift
import ActivityKit
import WidgetKit
import SwiftUI

// MARK: - Activity Attributes
struct WeatherActivityAttributes: ActivityAttributes {
    public struct ContentState: Codable, Hashable {
        var temperature: Int
        var condition: String
        var icon: String
        var highTemp: Int
        var lowTemp: Int
        var precipitationChance: Int?
        var alertType: AlertType?
        
        enum AlertType: String, Codable {
            case rain = "비"
            case snow = "눈"
            case storm = "폭풍"
            case heat = "폭염"
            case cold = "한파"
        }
    }
    
    var cityName: String
}

// MARK: - Live Activity Widget
struct WeatherLiveActivity: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: WeatherActivityAttributes.self) { context in
            // Lock Screen / Banner UI (Expanded)
            WeatherLiveActivityView(context: context)
                .activityBackgroundTint(Color.blue.opacity(0.2))
                .activitySystemActionForegroundColor(Color.black)
            
        } dynamicIsland: { context in
            DynamicIsland {
                // Expanded Region
                DynamicIslandExpandedRegion(.leading) {
                    HStack {
                        Image(systemName: context.state.icon)
                            .font(.title2)
                        VStack(alignment: .leading) {
                            Text(context.attributes.cityName)
                                .font(.caption)
                            Text("\(context.state.temperature)°")
                                .font(.title3)
                                .fontWeight(.semibold)
                        }
                    }
                }
                
                DynamicIslandExpandedRegion(.trailing) {
                    VStack(alignment: .trailing) {
                        Text(context.state.condition)
                            .font(.caption)
                        HStack {
                            Text("H:\(context.state.highTemp)°")
                            Text("L:\(context.state.lowTemp)°")
                        }
                        .font(.caption2)
                    }
                }
                
                DynamicIslandExpandedRegion(.center) {
                    if let precipChance = context.state.precipitationChance, precipChance > 0 {
                        HStack {
                            Image(systemName: "drop.fill")
                                .foregroundColor(.blue)
                            Text("\(precipChance)%")
                        }
                        .font(.caption)
                    }
                }
                
                DynamicIslandExpandedRegion(.bottom) {
                    if let alert = context.state.alertType {
                        HStack {
                            Image(systemName: "exclamationmark.triangle.fill")
                                .foregroundColor(.yellow)
                            Text("\(alert.rawValue) 주의보")
                                .font(.caption)
                        }
                        .padding(.vertical, 4)
                        .frame(maxWidth: .infinity)
                        .background(Color.yellow.opacity(0.2))
                        .clipShape(Capsule())
                    }
                }
            } compactLeading: {
                Image(systemName: context.state.icon)
                    .font(.caption)
            } compactTrailing: {
                Text("\(context.state.temperature)°")
                    .font(.caption)
                    .fontWeight(.semibold)
            } minimal: {
                Image(systemName: context.state.icon)
            }
            .widgetURL(URL(string: "weathernow://live"))
            .keylineTint(Color.blue)
        }
    }
}

// MARK: - Live Activity View
struct WeatherLiveActivityView: View {
    let context: ActivityViewContext<WeatherActivityAttributes>
    
    var body: some View {
        HStack {
            // 왼쪽: 현재 날씨
            VStack(alignment: .leading, spacing: 8) {
                Text(context.attributes.cityName)
                    .font(.caption)
                    .foregroundColor(.secondary)
                
                HStack(alignment: .top) {
                    Image(systemName: context.state.icon)
                        .font(.largeTitle)
                        .symbolRenderingMode(.multicolor)
                    
                    VStack(alignment: .leading) {
                        Text("\(context.state.temperature)°")
                            .font(.title)
                            .fontWeight(.semibold)
                        Text(context.state.condition)
                            .font(.caption)
                    }
                }
            }
            
            Spacer()
            
            // 오른쪽: 추가 정보
            VStack(alignment: .trailing, spacing: 8) {
                HStack {
                    Text("H:\(context.state.highTemp)°")
                    Text("L:\(context.state.lowTemp)°")
                }
                .font(.subheadline)
                
                if let precipChance = context.state.precipitationChance, precipChance > 0 {
                    HStack {
                        Image(systemName: "drop.fill")
                            .foregroundColor(.blue)
                        Text("\(precipChance)%")
                    }
                    .font(.caption)
                }
                
                if let alert = context.state.alertType {
                    Label(alert.rawValue, systemImage: "exclamationmark.triangle.fill")
                        .font(.caption)
                        .foregroundColor(.yellow)
                        .padding(.horizontal, 8)
                        .padding(.vertical, 4)
                        .background(Color.yellow.opacity(0.2))
                        .clipShape(Capsule())
                }
            }
        }
        .padding()
    }
}

// MARK: - Activity Manager
class WeatherActivityManager {
    static let shared = WeatherActivityManager()
    
    private var currentActivity: Activity<WeatherActivityAttributes>?
    
    func startActivity(for weather: WeatherData) {
        guard ActivityAuthorizationInfo().areActivitiesEnabled else { return }
        
        let attributes = WeatherActivityAttributes(cityName: weather.cityName)
        let state = WeatherActivityAttributes.ContentState(
            temperature: weather.temperature,
            condition: weather.conditionDescription,
            icon: weather.icon,
            highTemp: weather.maxTemperature,
            lowTemp: weather.minTemperature,
            precipitationChance: nil,
            alertType: nil
        )
        
        do {
            currentActivity = try Activity.request(
                attributes: attributes,
                contentState: state,
                pushType: .token
            )
        } catch {
            print("Live Activity 시작 실패: \(error)")
        }
    }
    
    func updateActivity(with weather: WeatherData) {
        Task {
            let state = WeatherActivityAttributes.ContentState(
                temperature: weather.temperature,
                condition: weather.conditionDescription,
                icon: weather.icon,
                highTemp: weather.maxTemperature,
                lowTemp: weather.minTemperature,
                precipitationChance: nil,
                alertType: checkForWeatherAlert(weather)
            )
            
            await currentActivity?.update(using: state)
        }
    }
    
    func endActivity() {
        Task {
            await currentActivity?.end(dismissalPolicy: .immediate)
        }
    }
    
    private func checkForWeatherAlert(_ weather: WeatherData) -> WeatherActivityAttributes.ContentState.AlertType? {
        switch weather.condition.lowercased() {
        case "rain", "drizzle":
            return .rain
        case "snow":
            return .snow
        case "thunderstorm":
            return .storm
        default:
            if weather.temperature > 35 {
                return .heat
            } else if weather.temperature < -10 {
                return .cold
            }
            return nil
        }
    }
}
```

### 4단계: 푸시 알림 구현

**Core/Notifications/NotificationManager.swift**:

```swift
import UserNotifications
import CoreLocation

class NotificationManager: NSObject, ObservableObject {
    static let shared = NotificationManager()
    
    @Published var isAuthorized = false
    private let notificationCenter = UNUserNotificationCenter.current()
    
    override init() {
        super.init()
        notificationCenter.delegate = self
        checkAuthorizationStatus()
    }
    
    // MARK: - Authorization
    
    func requestAuthorization() {
        notificationCenter.requestAuthorization(options: [.alert, .sound, .badge, .providesAppNotificationSettings]) { granted, error in
            DispatchQueue.main.async {
                self.isAuthorized = granted
                if granted {
                    self.registerCategories()
                    self.scheduleWeatherNotifications()
                }
            }
        }
    }
    
    private func checkAuthorizationStatus() {
        notificationCenter.getNotificationSettings { settings in
            DispatchQueue.main.async {
                self.isAuthorized = settings.authorizationStatus == .authorized
            }
        }
    }
    
    // MARK: - Categories
    
    private func registerCategories() {
        // 날씨 알림 카테고리
        let viewAction = UNNotificationAction(
            identifier: "VIEW_WEATHER",
            title: "날씨 보기",
            options: .foreground
        )
        
        let dismissAction = UNNotificationAction(
            identifier: "DISMISS",
            title: "닫기",
            options: .destructive
        )
        
        let weatherCategory = UNNotificationCategory(
            identifier: "WEATHER_ALERT",
            actions: [viewAction, dismissAction],
            intentIdentifiers: [],
            options: .customDismissAction
        )
        
        // 일일 예보 카테고리
        let forecastCategory = UNNotificationCategory(
            identifier: "DAILY_FORECAST",
            actions: [viewAction],
            intentIdentifiers: [],
            options: []
        )
        
        notificationCenter.setNotificationCategories([weatherCategory, forecastCategory])
    }
    
    // MARK: - Weather Notifications
    
    func sendWeatherAlert(for weather: WeatherData) {
        let content = UNMutableNotificationContent()
        content.title = "날씨 경보"
        content.categoryIdentifier = "WEATHER_ALERT"
        content.sound = .default
        content.badge = 1
        
        // 날씨 상태별 알림
        switch weather.condition.lowercased() {
        case "rain", "drizzle":
            content.body = "비가 예상됩니다. 우산을 챙기세요! ☔️"
            content.subtitle = "\(weather.cityName) · 강수 확률 높음"
            
        case "snow":
            content.body = "눈이 예상됩니다. 따뜻하게 입으세요! ❄️"
            content.subtitle = "\(weather.cityName) · 눈 예보"
            
        case "thunderstorm":
            content.body = "천둥번개가 예상됩니다. 실내에 머무르세요! ⛈"
            content.subtitle = "\(weather.cityName) · 폭풍 경고"
            
        default:
            if weather.temperature > 35 {
                content.body = "폭염 주의보! 충분한 수분 섭취를 하세요 🌡"
                content.subtitle = "\(weather.cityName) · 현재 \(weather.temperature)°C"
            } else if weather.temperature < -10 {
                content.body = "한파 주의보! 따뜻하게 입으세요 🧊"
                content.subtitle = "\(weather.cityName) · 현재 \(weather.temperature)°C"
            } else {
                return
            }
        }
        
        // 이미지 첨부
        if let imageURL = createWeatherImage(for: weather) {
            if let attachment = try? UNNotificationAttachment(identifier: "weather", url: imageURL, options: nil) {
                content.attachments = [attachment]
            }
        }
        
        let request = UNNotificationRequest(
            identifier: "weather_alert_\(Date().timeIntervalSince1970)",
            content: content,
            trigger: nil
        )
        
        notificationCenter.add(request)
    }
    
    func scheduleDailyForecast(at time: DateComponents) {
        let content = UNMutableNotificationContent()
        content.title = "오늘의 날씨"
        content.body = "오늘 하루 날씨를 확인하세요"
        content.categoryIdentifier = "DAILY_FORECAST"
        content.sound = .default
        
        let trigger = UNCalendarNotificationTrigger(dateMatching: time, repeats: true)
        
        let request = UNNotificationRequest(
            identifier: "daily_forecast",
            content: content,
            trigger: trigger
        )
        
        notificationCenter.add(request)
    }
    
    func scheduleWeatherNotifications() {
        // 아침 7시 일일 예보
        var morningComponents = DateComponents()
        morningComponents.hour = 7
        morningComponents.minute = 0
        scheduleDailyForecast(at: morningComponents)
        
        // 저녁 6시 내일 날씨
        var eveningComponents = DateComponents()
        eveningComponents.hour = 18
        eveningComponents.minute = 0
        scheduleDailyForecast(at: eveningComponents)
    }
    
    // MARK: - Location-based Notifications
    
    func setupLocationNotification(for location: CLLocation, weather: WeatherData) {
        let content = UNMutableNotificationContent()
        content.title = "위치 날씨 업데이트"
        content.body = "\(weather.cityName)의 현재 날씨: \(weather.temperature)°C, \(weather.conditionDescription)"
        content.sound = .default
        
        let region = CLCircularRegion(
            center: location.coordinate,
            radius: 1000,
            identifier: "weather_location"
        )
        region.notifyOnEntry = true
        region.notifyOnExit = false
        
        let trigger = UNLocationNotificationTrigger(region: region, repeats: true)
        
        let request = UNNotificationRequest(
            identifier: "location_weather",
            content: content,
            trigger: trigger
        )
        
        notificationCenter.add(request)
    }
    
    // MARK: - Helper Methods
    
    private func createWeatherImage(for weather: WeatherData) -> URL? {
        // 날씨 이미지 생성 로직
        return nil
    }
    
    func removeAllNotifications() {
        notificationCenter.removeAllPendingNotificationRequests()
        notificationCenter.removeAllDeliveredNotifications()
    }
    
    func removePendingNotification(with identifier: String) {
        notificationCenter.removePendingNotificationRequests(withIdentifiers: [identifier])
    }
}

// MARK: - UNUserNotificationCenterDelegate
extension NotificationManager: UNUserNotificationCenterDelegate {
    func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        completionHandler([.banner, .sound, .badge])
    }
    
    func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        switch response.actionIdentifier {
        case "VIEW_WEATHER":
            NotificationCenter.default.post(name: .openWeatherDetail, object: nil)
        case UNNotificationDefaultActionIdentifier:
            NotificationCenter.default.post(name: .openApp, object: nil)
        default:
            break
        }
        completionHandler()
    }
}

// MARK: - Notification Names
extension Notification.Name {
    static let openWeatherDetail = Notification.Name("openWeatherDetail")
    static let openApp = Notification.Name("openApp")
}
```

---

## 🎯 여기서 배운 것

### 1. **위젯 시스템**
- Timeline Provider
- 다양한 크기 지원
- 잠금 화면 위젯
- 위젯 업데이트 최적화

### 2. **Live Activities**
- Dynamic Island
- 실시간 업데이트
- 알림 상태 표시

### 3. **푸시 알림**
- 로컬 알림 스케줄링
- 위치 기반 알림
- 알림 액션 처리

---

## 🎉 성공 확인

**위젯 테스트**:
- [ ] 홈 화면 위젯 추가 가능
- [ ] 30분마다 자동 업데이트
- [ ] 모든 크기 정상 표시
- [ ] 잠금 화면 위젯 작동

**알림 테스트**:
- [ ] 날씨 경보 알림 수신
- [ ] 일일 예보 알림 작동
- [ ] Live Activity 표시됨
- [ ] Dynamic Island 애니메이션

---

**완벽합니다! 이제 24시간 사용자와 연결된 날씨 앱이 완성되었습니다.**