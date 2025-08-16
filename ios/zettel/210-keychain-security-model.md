# Keychain: 비밀의 금고

## Core Insight
Keychain은 단순한 저장소가 아니라 iOS 보안 아키텍처의 핵심이다. 비밀번호, 토큰, 인증서를 하드웨어 레벨에서 보호한다.

```swift
import Security

// 간단해 보이지만 복잡한 보안 체계가 숨어있다
let query: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrAccount as String: "user_token",
    kSecValueData as String: token.data(using: .utf8)!
]

let status = SecItemAdd(query as CFDictionary, nil)
```

Keychain의 보안 레벨:
1. **Hardware encryption**: Secure Enclave에서 암호화
2. **App-specific**: 다른 앱이 접근 불가
3. **Biometric protection**: Touch ID/Face ID로 추가 보호
4. **Backup exclusion**: iCloud 백업에서 제외 가능

가장 중요한 개념은 **Accessibility levels**:
- `kSecAttrAccessibleWhenUnlocked`: 기기 잠금 해제 시에만
- `kSecAttrAccessibleAfterFirstUnlock`: 첫 잠금 해제 후 계속
- `kSecAttrAccessibleAlways`: 항상 (deprecated, 보안상 위험)
- `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`: 이 기기에서만

Secure Enclave와의 연동:
```swift
let access = SecAccessControlCreateWithFlags(
    nil,
    kSecAttrAccessibleWhenUnlockedThisDeviceOnly,
    .biometryAny,
    nil
)
```

개발자의 실수:
1. UserDefaults에 민감한 정보 저장
2. Keychain 접근 레벨을 너무 느슨하게 설정
3. 에러 처리 부족으로 토큰 유실

현실: Keychain API는 복잡하다. 대부분 래퍼 라이브러리를 사용한다.

## Connections
→ [[211-biometric-authentication]]
→ [[212-secure-enclave-protection]]
→ [[213-certificate-pinning]]
← [[023-privacy-as-architecture]]

---
Level: L4
Date: 2025-08-16
Tags: #ios #security #keychain #encryption #biometrics #secure-enclave