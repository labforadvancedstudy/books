# 다크모드/라이트모드 전환

## 개념
iOS 13+ 다크 모드 지원으로 VoIP 앱 UI가 시스템 테마에 따라 자동 전환. 통화 화면은 야간 사용 시 눈부심 최소화가 중요.

## Dynamic Colors 설정
```swift
// Assets.xcassets에 Color Set 정의
extension UIColor {
    static let voipPrimary = UIColor(named: "VoIPPrimary")!
    static let voipBackground = UIColor(named: "VoIPBackground")!
    static let voipCallActive = UIColor(named: "CallActive")!
    
    // 프로그래매틱 색상 정의
    static let adaptiveCallButton = UIColor { traitCollection in
        switch traitCollection.userInterfaceStyle {
        case .dark:
            return UIColor(red: 0.0, green: 0.8, blue: 0.0, alpha: 1.0)
        default:
            return UIColor(red: 0.0, green: 0.6, blue: 0.0, alpha: 1.0)
        }
    }
}
```

## 통화 화면 테마 처리
```swift
class CallViewController: UIViewController {
    @IBOutlet weak var backgroundBlur: UIVisualEffectView!
    @IBOutlet weak var callerNameLabel: UILabel!
    
    override func traitCollectionDidChange(_ previousTraitCollection: UITraitCollection?) {
        super.traitCollectionDidChange(previousTraitCollection)
        
        if traitCollection.hasDifferentColorAppearance(comparedTo: previousTraitCollection) {
            updateCallUITheme()
        }
    }
    
    private func updateCallUITheme() {
        // 블러 효과 조정
        let style: UIBlurEffect.Style = traitCollection.userInterfaceStyle == .dark ? .dark : .light
        backgroundBlur.effect = UIBlurEffect(style: style)
        
        // 텍스트 가독성 향상
        if traitCollection.userInterfaceStyle == .dark {
            callerNameLabel.textColor = .white
            callerNameLabel.layer.shadowOpacity = 0.8
        } else {
            callerNameLabel.textColor = .black
            callerNameLabel.layer.shadowOpacity = 0.3
        }
    }
}
```

## SF Symbols 활용
```swift
// 시스템 아이콘 자동 테마 적용
let config = UIImage.SymbolConfiguration(
    pointSize: 44,
    weight: .medium,
    scale: .large
)

let phoneIcon = UIImage(
    systemName: "phone.fill",
    withConfiguration: config
)?
.withTintColor(.voipPrimary, renderingMode: .alwaysOriginal)

// 다크모드에서 자동 반전
phoneButton.setImage(phoneIcon, for: .normal)
```

## 커스텀 테마 전환
```swift
class ThemeManager {
    enum Theme: String {
        case light, dark, auto
    }
    
    static func applyTheme(_ theme: Theme) {
        let windows = UIApplication.shared.windows
        
        switch theme {
        case .light:
            windows.forEach { $0.overrideUserInterfaceStyle = .light }
        case .dark:
            windows.forEach { $0.overrideUserInterfaceStyle = .dark }
        case .auto:
            windows.forEach { $0.overrideUserInterfaceStyle = .unspecified }
        }
        
        // UserDefaults에 저장
        UserDefaults.standard.set(theme.rawValue, forKey: "selectedTheme")
    }
}
```

## 연관 개념
- [[accessibility_voiceover]]
- [[dynamic_type]]
- [[haptic_feedback]]

## 태그
#ui #darkmode #theme #accessibility #ios13