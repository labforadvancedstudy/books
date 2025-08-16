# Swift Package Manager: 의존성의 민주화

## Core Insight
SPM은 단순한 패키지 매니저가 아니라 Swift 생태계의 공개적이고 분산된 의존성 관리 시스템이다. 중앙화된 권력 없이 협력한다.

```swift
// Package.swift - 의존성을 코드로 선언
let package = Package(
    name: "MyApp",
    dependencies: [
        .package(url: "https://github.com/Alamofire/Alamofire.git", from: "5.0.0")
    ],
    targets: [
        .target(name: "MyApp", dependencies: ["Alamofire"])
    ]
)
```

SPM의 철학적 차이점:
1. **Decentralized**: 중앙 저장소 없음 (CocoaPods와 대조)
2. **Git-based**: Git URL이 곧 패키지 주소
3. **Xcode integrated**: 별도 도구 설치 불필요
4. **Cross-platform**: iOS, macOS, Linux 모두 지원

Semantic Versioning의 엄격한 적용:
- **Major**: 호환성 깨는 변경 (1.0.0 → 2.0.0)
- **Minor**: 기능 추가 (1.0.0 → 1.1.0)
- **Patch**: 버그 수정 (1.0.0 → 1.0.1)

가장 혁신적인 것은 **dependency resolution**이다:
```swift
dependencies: [
    .package(url: "...", from: "1.0.0"),      // >= 1.0.0, < 2.0.0
    .package(url: "...", .upToNextMajor(from: "1.0.0")),  // 명시적
    .package(url: "...", exact: "1.5.2")     // 정확히 이 버전만
]
```

문제점들:
- **Binary frameworks**: 아직 불완전한 지원
- **Resource handling**: 복잡한 리소스 관리
- **Build performance**: 큰 프로젝트에서 느림

하지만 트렌드는 명확하다: CocoaPods → SPM 마이그레이션이 가속화되고 있다.

실제 사용 패턴:
```swift
import PackageDescription

let package = Package(
    name: "NetworkKit",
    platforms: [.iOS(.v15)],
    products: [
        .library(name: "NetworkKit", targets: ["NetworkKit"])
    ],
    dependencies: [],
    targets: [
        .target(name: "NetworkKit"),
        .testTarget(name: "NetworkKitTests", dependencies: ["NetworkKit"])
    ]
)
```

## Connections
→ [[241-package-dependency-resolution]]
→ [[242-binary-framework-distribution]]
→ [[243-local-package-development]]
← [[160-xcode-as-integrated-environment]]

---
Level: L3
Date: 2025-08-16
Tags: #ios #spm #dependencies #packages #versioning #ecosystem