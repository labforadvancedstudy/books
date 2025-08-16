# 했틱 피드백 활용

## 개념
VoIP 통화 중 중요한 이벤트를 촉각으로 전달. 무음 모드나 시끌러운 환경에서도 상태 변화를 인지.

## Haptic 타입별 구현
```swift
class HapticManager {
    private let impactGenerator = UIImpactFeedbackGenerator()
    private let notificationGenerator = UINotificationFeedbackGenerator()
    private let selectionGenerator = UISelectionFeedbackGenerator()
    
    // 통화 시작/종료
    func callStateChanged(_ state: CallState) {
        switch state {
        case .connecting:
            // 부드러운 탭
            impactGenerator.impactOccurred(intensity: 0.5)
            
        case .connected:
            // 성공 피드백
            notificationGenerator.notificationOccurred(.success)
            
        case .ended:
            // 강한 탭
            impactGenerator.impactOccurred(intensity: 1.0)
            
        case .failed:
            // 실패 피드백
            notificationGenerator.notificationOccurred(.error)
            
        default:
            break
        }
    }
    
    // 버튼 터치
    func buttonTapped() {
        selectionGenerator.selectionChanged()
    }
}
```

## 커스텀 했틱 패턴
```swift
import CoreHaptics

class CustomHapticEngine {
    private var engine: CHHapticEngine?
    
    func setupEngine() {
        guard CHHapticEngine.capabilitiesForHardware().supportsHaptics else { return }
        
        do {
            engine = try CHHapticEngine()
            try engine?.start()
        } catch {
            print("Haptic engine failed: \(error)")
        }
    }
    
    // 전화 벨소리 패턴
    func playRingtoneHaptic() {
        let pattern = [
            CHHapticEvent(eventType: .hapticTransient, parameters: [
                CHHapticEventParameter(parameterID: .hapticIntensity, value: 1.0),
                CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.5)
            ], relativeTime: 0),
            CHHapticEvent(eventType: .hapticTransient, parameters: [
                CHHapticEventParameter(parameterID: .hapticIntensity, value: 0.8),
                CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.3)
            ], relativeTime: 0.5)
        ]
        
        do {
            let pattern = try CHHapticPattern(events: pattern, parameters: [])
            let player = try engine?.makePlayer(with: pattern)
            try player?.start(atTime: 0)
        } catch {
            print("Failed to play haptic: \(error)")
        }
    }
}
```

## 네트워크 상태 했틱
```swift
extension HapticManager {
    func networkQualityChanged(_ quality: NetworkQuality) {
        switch quality {
        case .poor:
            // 경고 패턴: 짧은 진동 2번
            impactGenerator.prepare()
            impactGenerator.impactOccurred(intensity: 0.7)
            DispatchQueue.main.asyncAfter(deadline: .now() + 0.1) {
                self.impactGenerator.impactOccurred(intensity: 0.7)
            }
            
        case .recovered:
            // 회복 알림
            notificationGenerator.notificationOccurred(.success)
            
        default:
            break
        }
    }
}
```

## 접근성 고려사항
```swift
class AccessibleHapticManager: HapticManager {
    override func callStateChanged(_ state: CallState) {
        // VoiceOver 사용자에게 더 강한 피드백
        let isVoiceOverRunning = UIAccessibility.isVoiceOverRunning
        
        if isVoiceOverRunning {
            // 강화된 했틱
            super.callStateChanged(state)
            
            // 추가 패턴
            if state == .connected || state == .ended {
                DispatchQueue.main.asyncAfter(deadline: .now() + 0.2) {
                    self.impactGenerator.impactOccurred(intensity: 0.5)
                }
            }
        } else {
            super.callStateChanged(state)
        }
    }
}
```

## 배터리 최적화
```swift
// 했틱 사용 설정
struct HapticSettings {
    static var isEnabled: Bool {
        get { UserDefaults.standard.bool(forKey: "hapticEnabled") }
        set { UserDefaults.standard.set(newValue, forKey: "hapticEnabled") }
    }
    
    static var intensity: Float {
        get { UserDefaults.standard.float(forKey: "hapticIntensity") }
        set { UserDefaults.standard.set(newValue, forKey: "hapticIntensity") }
    }
    
    // 저전력 모드에서 비활성화
    static var shouldUseHaptics: Bool {
        isEnabled && !ProcessInfo.processInfo.isLowPowerModeEnabled
    }
}
```

## 연관 개념
- [[accessibility_voiceover]]
- [[battery_optimization]]
- [[network_quality_indicator]]

## 태그
#haptic #feedback #ux #accessibility #taptic