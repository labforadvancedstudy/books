# MOS Score (통화 품질 지표)

## 개념
Mean Opinion Score - 사용자가 체감하는 통화 품질을 1-5점 척도로 정량화. VoIP 앱에서 실시간 품질 모니터링과 개선에 필수.

## MOS 계산 알고리즘
```swift
struct MOSCalculator {
    // E-Model (ITU-T G.107) 기반 계산
    static func calculateMOS(
        packetLoss: Float,    // 0-100%
        jitter: Float,        // ms
        latency: Float,       // ms
        codec: AudioCodec
    ) -> Float {
        // R-Factor 계산
        var R: Float = 93.2  // 기본값
        
        // 코덱 품질 감소
        R -= codec.impairmentFactor
        
        // 패킷 손실 영향
        let Ie = 30 * log10(1 + 15 * packetLoss)
        R -= Ie
        
        // 지연 영향
        if latency > 150 {
            R -= (latency - 150) * 0.1
        }
        
        // Jitter 영향
        R -= jitter * 0.02
        
        // R-Factor를 MOS로 변환
        if R < 0 { return 1.0 }
        if R > 100 { return 4.5 }
        
        return 1 + 0.035 * R + 7e-6 * R * (R - 60) * (100 - R)
    }
}
```

## MOS 척도 해석
| MOS | 품질 | 사용자 체감 |
|-----|------|------------|
| 4.3-5.0 | Excellent | 유선 전화 수준 |
| 4.0-4.3 | Good | 명확한 통화 |
| 3.6-4.0 | Fair | 약간 불편 |
| 3.1-3.6 | Poor | 상당히 불편 |
| 1.0-3.1 | Bad | 통화 불가 |

## 실시간 MOS 모니터링
```swift
class QualityMonitor {
    private var samples: [Float] = []
    private let sampleWindow = 30 // 30초 윈도우
    
    func updateMOS() {
        let currentMOS = MOSCalculator.calculateMOS(
            packetLoss: rtpStats.packetLossRate,
            jitter: rtpStats.jitter,
            latency: rtpStats.roundTripTime,
            codec: currentCodec
        )
        
        samples.append(currentMOS)
        
        // 이동 평균
        let averageMOS = samples.suffix(sampleWindow).reduce(0, +) / Float(sampleWindow)
        
        // UI 업데이트
        if averageMOS < 3.6 {
            showPoorQualityWarning()
        }
    }
}
```

## 코덱별 Impairment Factor
```swift
enum AudioCodec {
    case opus
    case g711
    case g729
    
    var impairmentFactor: Float {
        switch self {
        case .opus: return 0    // 최고 품질
        case .g711: return 0    // 표준 품질
        case .g729: return 10   // 압축 손실
        }
    }
}
```

## 연관 개념
- [[call_quality_perception]]
- [[network_quality_indicator]]
- [[opus_codec]]
- [[jitter_buffer]]

## 태그
#quality #metrics #mos #monitoring #ux