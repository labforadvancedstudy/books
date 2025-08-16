# VoiceOver 접근성

## 개념
시각 장애인을 위한 iOS 스크린 리더 지원. VoIP 앱에서 통화 제어와 상태 정보를 음성으로 제공.

## VoIP UI 접근성 구현
```swift
class AccessibleCallButton: UIButton {
    var callState: CallState = .idle {
        didSet {
            updateAccessibility()
        }
    }
    
    private func updateAccessibility() {
        // 레이블 설정
        switch callState {
        case .idle:
            accessibilityLabel = "Start Call"
            accessibilityHint = "Double tap to start a voice call"
        case .ringing:
            accessibilityLabel = "Incoming call"
            accessibilityHint = "Double tap to answer, triple tap to decline"
        case .active:
            accessibilityLabel = "End Call"
            accessibilityHint = "Double tap to end the current call"
        }
        
        // Trait 설정
        accessibilityTraits = [.button, .startsMediaSession]
        
        // Custom action 추가
        accessibilityCustomActions = [
            UIAccessibilityCustomAction(
                name: "Mute",
                target: self,
                selector: #selector(toggleMute)
            ),
            UIAccessibilityCustomAction(
                name: "Speaker",
                target: self,
                selector: #selector(toggleSpeaker)
            )
        ]
    }
}
```

## 통화 상태 알림
```swift
class CallStateAnnouncer {
    static func announce(_ state: CallState, caller: String? = nil) {
        var announcement: String
        
        switch state {
        case .incoming:
            announcement = "Incoming call from \(caller ?? "Unknown")"
        case .connecting:
            announcement = "Connecting"
        case .connected:
            announcement = "Call connected"
        case .ended:
            announcement = "Call ended"
        case .failed:
            announcement = "Call failed"
        }
        
        // VoiceOver 즉시 알림
        UIAccessibility.post(
            notification: .announcement,
            argument: announcement
        )
    }
    
    // 화면 변경 알림
    static func notifyScreenChange() {
        UIAccessibility.post(
            notification: .screenChanged,
            argument: nil
        )
    }
}
```

## 커스텀 로터 컨트롤
```swift
class CallControlsAccessibilityContainer: UIAccessibilityElement {
    private let muteButton: UIButton
    private let speakerButton: UIButton
    private let keypadButton: UIButton
    
    override var accessibilityElements: [Any]? {
        get { [muteButton, speakerButton, keypadButton] }
        set {}
    }
    
    // 로터 제스처 지원
    override func accessibilityIncrement() {
        // 다음 컨트롤로 이동
        moveToNextControl()
    }
    
    override func accessibilityDecrement() {
        // 이전 컨트롤로 이동
        moveToPreviousControl()
    }
}
```

## 통화 시간 표시
```swift
class CallDurationLabel: UILabel {
    private var callDuration: TimeInterval = 0
    
    override var accessibilityLabel: String? {
        get {
            let formatter = DateComponentsFormatter()
            formatter.allowedUnits = [.hour, .minute, .second]
            formatter.unitsStyle = .spellOut
            
            return "Call duration: \(formatter.string(from: callDuration) ?? "0 seconds")"
        }
        set {}
    }
    
    // 주기적 업데이트
    func startAccessibilityUpdates() {
        Timer.scheduledTimer(withTimeInterval: 30, repeats: true) { _ in
            UIAccessibility.post(
                notification: .announcement,
                argument: self.accessibilityLabel
            )
        }
    }
}
```

## 연관 개념
- [[dynamic_type]]
- [[dark_mode_support]]
- [[haptic_feedback]]

## 태그
#accessibility #voiceover #a11y #ui #inclusive