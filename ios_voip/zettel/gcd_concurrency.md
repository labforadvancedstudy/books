# GCD와 동시성 처리

## 개념
VoIP 앱에서 오디오 스트림, 네트워크 I/O, UI 업데이트를 동시에 처리하기 위한 멀티스레딩 전략.

## GCD (Grand Central Dispatch) 패턴
```swift
class VoIPCallManager {
    private let audioQueue = DispatchQueue(label: "audio", qos: .userInteractive)
    private let networkQueue = DispatchQueue(label: "network", qos: .default)
    private let uiQueue = DispatchQueue.main
    
    func processIncomingAudio(data: Data) {
        audioQueue.async { [weak self] in
            // 오디오 디코딩 (고우선순위)
            let pcm = self?.decodeAudio(data)
            
            self?.uiQueue.async {
                // UI 업데이트는 메인 큐에서
                self?.updateVolumeIndicator(pcm?.level)
            }
        }
    }
}
```

## async/await 패턴 (iOS 13+)
```swift
actor AudioProcessor {
    private var buffer: [Float] = []
    
    func processFrame(_ frame: AudioFrame) async {
        // Actor로 thread-safe 보장
        buffer.append(contentsOf: frame.samples)
        
        if buffer.count > threshold {
            await encodeAndSend()
        }
    }
}
```

## VoIP 특화 큐 전략
- **Audio Queue**: QoS.userInteractive (최고 우선순위)
- **Network Queue**: QoS.default (일반 우선순위)
- **Database Queue**: QoS.background (낮은 우선순위)

## 동시성 문제 해결
```swift
// Serial queue로 race condition 방지
private let stateQueue = DispatchQueue(label: "state")
private var _callState: CallState = .idle

var callState: CallState {
    get { stateQueue.sync { _callState } }
    set { stateQueue.sync { _callState = newValue } }
}
```

## 연관 개념
- [[call_state_machine]]
- [[audio_unit_processing]]
- [[network_socket]]

## 태그
#concurrency #gcd #threading #async #performance