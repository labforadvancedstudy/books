# 다국어 지원 (Localization)

## 개념
VoIP 앱을 여러 언어와 지역에 맞게 현지화하여 글로벌 사용자 경험 제공.

## 문자열 현지화
```swift
// Localizable.strings (English)
"call.incoming.title" = "Incoming Call";
"call.incoming.subtitle" = "from %@";
"call.button.accept" = "Accept";
"call.button.decline" = "Decline";
"call.duration.format" = "%02d:%02d";
"call.quality.poor" = "Poor connection";
"call.ended.message" = "Call ended - %@";

// Localizable.strings (Korean)
"call.incoming.title" = "수신 전화";
"call.incoming.subtitle" = "%@로부터";
"call.button.accept" = "수락";
"call.button.decline" = "거절";
"call.duration.format" = "%02d:%02d";
"call.quality.poor" = "연결 상태 불량";
"call.ended.message" = "통화 종료 - %@";
```

## Swift 현지화 Helper
```swift
extension String {
    var localized: String {
        return NSLocalizedString(self, comment: "")
    }
    
    func localized(with arguments: CVarArg...) -> String {
        return String(format: self.localized, arguments: arguments)
    }
    
    func localizedPlural(count: Int) -> String {
        return String.localizedStringWithFormat(
            NSLocalizedString(self, comment: ""),
            count
        )
    }
}

// 사용 예시
let title = "call.incoming.title".localized
let subtitle = "call.incoming.subtitle".localized(with: contactName)
let message = "call.missed.count".localizedPlural(count: 3)
```

## 날짜/시간 형식
```swift
class DateTimeFormatter {
    static let callDateFormatter: DateFormatter = {
        let formatter = DateFormatter()
        formatter.dateStyle = .medium
        formatter.timeStyle = .short
        formatter.doesRelativeDateFormatting = true
        return formatter
    }()
    
    static let durationFormatter: DateComponentsFormatter = {
        let formatter = DateComponentsFormatter()
        formatter.allowedUnits = [.hour, .minute, .second]
        formatter.unitsStyle = .positional
        formatter.zeroFormattingBehavior = .pad
        return formatter
    }()
    
    static func formatCallTime(_ date: Date) -> String {
        // 사용자 로케일에 맞게 자동 포맷팅
        return callDateFormatter.string(from: date)
    }
    
    static func formatDuration(_ duration: TimeInterval) -> String {
        return durationFormatter.string(from: duration) ?? "00:00"
    }
    
    static func formatRelativeTime(_ date: Date) -> String {
        let formatter = RelativeDateTimeFormatter()
        formatter.unitsStyle = .full
        return formatter.localizedString(for: date, relativeTo: Date())
    }
}
```

## 숫자 포맷팅
```swift
class NumberFormatter {
    // 전화번호 포맷팅
    static func formatPhoneNumber(_ number: String, for locale: Locale = .current) -> String {
        let phoneUtil = NBPhoneNumberUtil()
        
        do {
            let phoneNumber = try phoneUtil.parse(number, defaultRegion: locale.regionCode)
            
            // 국제 표준 형식
            if phoneNumber.countryCode != locale.countryCode {
                return try phoneUtil.format(phoneNumber, numberFormat: .INTERNATIONAL)
            } else {
                // 국내 형식
                return try phoneUtil.format(phoneNumber, numberFormat: .NATIONAL)
            }
        } catch {
            return number
        }
    }
    
    // 통화 요금 포맷팅
    static func formatCurrency(_ amount: Decimal, currencyCode: String? = nil) -> String {
        let formatter = Foundation.NumberFormatter()
        formatter.numberStyle = .currency
        formatter.currencyCode = currencyCode ?? Locale.current.currencyCode
        formatter.maximumFractionDigits = 2
        
        return formatter.string(from: amount as NSNumber) ?? "$0.00"
    }
    
    // 데이터 사용량 포맷팅
    static func formatDataUsage(_ bytes: Int64) -> String {
        let formatter = ByteCountFormatter()
        formatter.countStyle = .decimal
        formatter.allowedUnits = [.useKB, .useMB, .useGB]
        
        return formatter.string(fromByteCount: bytes)
    }
}
```

## UI 현지화
```swift
class LocalizedUI {
    static func setupLocalizedUI(for viewController: UIViewController) {
        // 버튼 텍스트
        if let callButton = viewController.view.viewWithTag(100) as? UIButton {
            callButton.setTitle("call.button.start".localized, for: .normal)
        }
        
        // 네비게이션 타이틀
        viewController.navigationItem.title = "app.title".localized
        
        // 탭 바 아이템
        viewController.tabBarItem.title = "tab.calls".localized
    }
    
    static func localizedImage(named: String) -> UIImage? {
        // 언어별 이미지 리소스
        let languageCode = Locale.current.languageCode ?? "en"
        let localizedName = "\(named)_\(languageCode)"
        
        return UIImage(named: localizedName) ?? UIImage(named: named)
    }
}
```

## 동적 언어 전환
```swift
class LanguageManager {
    static var currentLanguage: String {
        get { UserDefaults.standard.string(forKey: "AppLanguage") ?? "en" }
        set {
            UserDefaults.standard.set(newValue, forKey: "AppLanguage")
            applyLanguage(newValue)
        }
    }
    
    static func applyLanguage(_ languageCode: String) {
        // Bundle 오버라이드
        Bundle.setLanguage(languageCode)
        
        // UI 재시작
        restartUI()
        
        // 알림
        NotificationCenter.default.post(
            name: .languageChanged,
            object: nil,
            userInfo: ["language": languageCode]
        )
    }
    
    private static func restartUI() {
        let storyboard = UIStoryboard(name: "Main", bundle: nil)
        let window = UIApplication.shared.windows.first
        window?.rootViewController = storyboard.instantiateInitialViewController()
        
        UIView.transition(
            with: window!,
            duration: 0.3,
            options: .transitionCrossDissolve,
            animations: nil
        )
    }
}
```

## 연관 개녕
- [[rtl_support]]
- [[timezone_handling]]
- [[country_regulations]]

## 태그
#localization #i18n #internationalization #translation #multilingual