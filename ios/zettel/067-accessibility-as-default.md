# Accessibility: 포용의 설계

## Core Insight
접근성은 옵션이 아니라 기본값이다. 15%의 사용자가 어떤 형태로든 장애를 가지고 있다. 접근성은 도덕이자 비즈니스다.

SwiftUI는 접근성을 기본으로 제공한다. Text는 자동으로 VoiceOver를 지원하고, Button은 자동으로 초점을 받는다. 하지만 기본값에 만족하면 안 된다.

```swift
Image("coffee")
    .accessibilityLabel("라떼 한 잔")
    .accessibilityHint("탭하여 주문하기")
    .accessibilityAddTraits(.isButton)
    
Slider(value: $volume)
    .accessibilityValue("\(Int(volume * 100))퍼센트")
    .accessibilityAdjustableAction { direction in
        switch direction {
        case .increment: volume += 0.1
        case .decrement: volume -= 0.1
        }
    }
```

Dynamic Type은 사용자의 선택을 존중한다. 고정 폰트 크기를 쓰지 마라. .body, .headline, .caption을 써라. 레이아웃이 깨지는 것보다 읽을 수 없는 것이 더 나쁘다.

색맹 사용자를 고려하라. 색상만으로 정보를 전달하지 마라. 아이콘, 텍스트, 패턴을 함께 사용하라. Increase Contrast 모드에서도 테스트하라.

## Connections
→ [[068-voiceover-navigation]]
→ [[069-dynamic-type-layout]]
← [[002-interface-invisibility]]

---
Level: L6
Date: 2025-08-15
Tags: #ios #accessibility #inclusive #voiceover