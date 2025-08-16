# Swift-Objective-C 브릿징

## 개념
Swift와 Objective-C 코드를 단일 iOS 앱에서 상호 운용하기 위한 메커니즘. VoIP 앱에서 레거시 라이브러리나 C++ 기반 음성 처리 엔진과 통합할 때 필수적.

## 핵심 요소
- **Bridging Header**: Objective-C를 Swift에 노출
- **@objc 어노테이션**: Swift를 Objective-C에 노출
- **NSObject 상속**: Objective-C 런타임 접근
- **Module Map**: C/C++ 라이브러리 임포트

## VoIP 구현 시 브릿징 필요 영역
```swift
// WebRTC 네이티브 라이브러리 접근
@objc public class RTCWrapper: NSObject {
    private let peerConnection: RTCPeerConnection
    
    @objc func startCall() {
        // Objective-C WebRTC SDK 호출
    }
}
```

## 성능 고려사항
- 브릿징 오버헤드는 일반적으로 무시 가능
- 빈번한 콜백에서는 성능 영향 있음
- 오디오 프레임 처리 시 직접 C++ 사용 권장

## 연관 개념
- [[memory_management_arc]]
- [[webrtc_native_integration]]
- [[audio_unit_processing]]

## 태그
#implementation #bridging #swift #objective-c #interop