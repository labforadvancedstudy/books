# Performance Optimization: 보이지 않는 사용자 경험

## Core Insight
성능 최적화는 단순한 기술적 개선이 아니라 사용자 존중의 표현이다. 빠른 앱은 사용자의 시간을 소중히 여긴다는 메시지다.

성능의 다섯 가지 차원:
1. **Launch time**: 앱 시작 속도 (2초 이내)
2. **Response time**: 터치 반응 속도 (100ms 이내)
3. **Frame rate**: 애니메이션 부드러움 (60fps)
4. **Memory usage**: 메모리 효율성
5. **Battery life**: 배터리 소모

```swift
// 성능 측정의 시작점
import os.signpost

let log = OSLog(subsystem: "com.app.performance", category: "networking")

os_signpost(.begin, log: log, name: "API Call")
// 작업 수행
os_signpost(.end, log: log, name: "API Call")
```

가장 중요한 원칙들:

**1. Premature optimization is evil**
- 측정 없는 최적화는 시간 낭비
- 병목지점을 먼저 찾아라
- 사용자가 느끼는 부분부터 개선

**2. 60fps의 비밀**
- 16.67ms 안에 모든 작업 완료
- 메인 스레드에서 무거운 작업 금지
- 레이아웃 계산 최소화

**3. 메모리 관리**
```swift
// 이미지 다운샘플링으로 메모리 절약
func downsample(imageAt url: URL, to pointSize: CGSize, scale: CGFloat) -> UIImage? {
    let options = [
        kCGImageSourceCreateThumbnailFromImageAlways: true,
        kCGImageSourceCreateThumbnailWithTransform: true,
        kCGImageSourceThumbnailMaxPixelSize: max(pointSize.width, pointSize.height) * scale
    ] as [CFString: Any]
    
    guard let imageSource = CGImageSourceCreateWithURL(url as CFURL, nil),
          let image = CGImageSourceCreateThumbnailAtIndex(imageSource, 0, options as CFDictionary) else {
        return nil
    }
    
    return UIImage(cgImage: image)
}
```

**4. 배터리 최적화**
- 위치 서비스 최소화
- 백그라운드 작업 제한
- 네트워크 요청 배칭

Instruments가 최고의 친구다: Time Profiler, Allocations, Energy Log.

## Connections
→ [[281-instruments-profiling-tools]]
→ [[282-memory-optimization-techniques]]
→ [[283-battery-life-optimization]]
← [[064-performance-as-feature]]

---
Level: L5
Date: 2025-08-16
Tags: #ios #performance #optimization #instruments #user-experience #respect