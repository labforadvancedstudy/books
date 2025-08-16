# Dynamic Type 지원

## 개념
사용자가 시스템 설정에서 선택한 텍스트 크기에 따라 VoIP 앱 UI가 자동 조정. 시력이 약한 사용자의 접근성 향상.

## 기본 구현
```swift
class CallViewController: UIViewController {
    @IBOutlet weak var callerNameLabel: UILabel!
    @IBOutlet weak var callDurationLabel: UILabel!
    @IBOutlet weak var phoneNumberLabel: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupDynamicType()
        
        // 텍스트 크기 변경 감지
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(contentSizeCategoryChanged),
            name: UIContentSizeCategory.didChangeNotification,
            object: nil
        )
    }
    
    private func setupDynamicType() {
        // 시스템 텍스트 스타일 사용
        callerNameLabel.font = .preferredFont(forTextStyle: .largeTitle)
        callerNameLabel.adjustsFontForContentSizeCategory = true
        
        callDurationLabel.font = .preferredFont(forTextStyle: .headline)
        callDurationLabel.adjustsFontForContentSizeCategory = true
        
        phoneNumberLabel.font = .preferredFont(forTextStyle: .body)
        phoneNumberLabel.adjustsFontForContentSizeCategory = true
    }
}
```

## 커스텀 폰트 스케일링
```swift
extension UIFont {
    static func scaledFont(for textStyle: TextStyle, size: CGFloat) -> UIFont {
        let font = UIFont.systemFont(ofSize: size)
        let metrics = UIFontMetrics(forTextStyle: textStyle)
        return metrics.scaledFont(for: font)
    }
    
    // VoIP 앱 전용 폰트
    static var voipCallerName: UIFont {
        scaledFont(for: .largeTitle, size: 34)
    }
    
    static var voipButtonLabel: UIFont {
        scaledFont(for: .body, size: 17)
    }
    
    static var voipStatusText: UIFont {
        scaledFont(for: .caption1, size: 12)
    }
}
```

## 레이아웃 자동 조정
```swift
class AdaptiveCallControls: UIView {
    private var regularConstraints: [NSLayoutConstraint] = []
    private var largeTextConstraints: [NSLayoutConstraint] = []
    
    @objc private func contentSizeCategoryChanged() {
        let isAccessibilityCategory = traitCollection.preferredContentSizeCategory.isAccessibilityCategory
        
        if isAccessibilityCategory {
            // 큰 텍스트용 세로 레이아웃
            NSLayoutConstraint.deactivate(regularConstraints)
            NSLayoutConstraint.activate(largeTextConstraints)
            stackView.axis = .vertical
            stackView.spacing = 20
        } else {
            // 일반 가로 레이아웃
            NSLayoutConstraint.deactivate(largeTextConstraints)
            NSLayoutConstraint.activate(regularConstraints)
            stackView.axis = .horizontal
            stackView.spacing = 10
        }
        
        layoutIfNeeded()
    }
}
```

## 텍스트 자르기 처리
```swift
class ScalableCallButton: UIButton {
    override func layoutSubviews() {
        super.layoutSubviews()
        
        titleLabel?.numberOfLines = 0
        titleLabel?.lineBreakMode = .byWordWrapping
        
        // 텍스트 크기에 따른 버튼 크기 조정
        let category = traitCollection.preferredContentSizeCategory
        
        if category >= .accessibilityMedium {
            // 버튼 크기 확대
            constraints.first { $0.firstAttribute == .height }?.constant = 60
            constraints.first { $0.firstAttribute == .width }?.constant = 200
        } else {
            // 기본 크기
            constraints.first { $0.firstAttribute == .height }?.constant = 44
            constraints.first { $0.firstAttribute == .width }?.constant = 150
        }
    }
}
```

## 최대 텍스트 크기 제한
```swift
// 특정 UI 요소에 최대 크기 제한
extension UILabel {
    func limitMaximumContentSize() {
        maximumContentSizeCategory = .accessibilityMedium
    }
}

// 사용 예
phoneNumberLabel.limitMaximumContentSize() // 전화번호는 너무 크면 안됨
```

## 연관 개념
- [[accessibility_voiceover]]
- [[dark_mode_support]]
- [[localization_rtl]]

## 태그
#accessibility #dynamictype #ui #fonts #adaptive