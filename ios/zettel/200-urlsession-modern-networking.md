# URLSession: 네트워킹의 현대화

## Core Insight
URLSession은 단순한 HTTP 클라이언트가 아니라 모든 네트워킹 복잡성을 감춘 고수준 추상화다. async/await와 만나 완전히 새로운 차원이 되었다.

```swift
// 과거: 콜백 지옥
URLSession.shared.dataTask(with: url) { data, response, error in
    DispatchQueue.main.async {
        // UI 업데이트
    }
}.resume()

// 현재: 선형적 사고
let (data, _) = try await URLSession.shared.data(from: url)
```

URLSession의 계층 구조:
1. **URLSession**: 설정 컨테이너
2. **URLSessionConfiguration**: 정책과 동작 정의
3. **URLSessionTask**: 실제 작업 수행

세 가지 configuration:
- **default**: 표준 설정
- **ephemeral**: 캐시 없음 (private browsing)
- **background**: 앱이 꺼져도 계속 실행

가장 혁신적인 것은 **background sessions**다:
```swift
let config = URLSessionConfiguration.background(withIdentifier: "com.app.download")
let session = URLSession(configuration: config, delegate: self, delegateQueue: nil)
```

앱이 종료되어도 시스템이 다운로드를 계속한다. 완료되면 앱을 깨워서 알려준다.

현대적 패턴:
```swift
actor NetworkService {
    func fetchUser(id: String) async throws -> User {
        let url = URL(string: "https://api.example.com/users/\(id)")!
        let (data, _) = try await URLSession.shared.data(from: url)
        return try JSONDecoder().decode(User.self, from: data)
    }
}
```

Error handling도 더 명확해졌다. throws로 명시적 오류 처리.

## Connections
→ [[201-json-codable-protocol]]
→ [[202-network-error-handling]]
→ [[203-cache-and-offline]]
← [[052-async-await-practical]]

---
Level: L2
Date: 2025-08-16
Tags: #ios #networking #urlsession #async #http #api