# 로컬 캠싱 전략

## 개념
VoIP 앱에서 연락처, 프로필 이미지, 설정값, 오디오 파일 등을 효율적으로 캠싱하여 성능 향상과 네트워크 사용량 감소.

## 캠싱 계층 구조
```swift
class CacheManager {
    // L1: 메모리 캠시 (NSCache)
    private let memoryCache = NSCache<NSString, AnyObject>()
    
    // L2: 디스크 캠시 (FileManager)
    private let diskCacheURL = FileManager.default
        .urls(for: .cachesDirectory, in: .userDomainMask)[0]
        .appendingPathComponent("VoIPCache")
    
    // L3: iCloud/CloudKit (동기화)
    private let cloudContainer = CKContainer.default()
    
    init() {
        // 메모리 경고 시 자동 정리
        memoryCache.totalCostLimit = 50 * 1024 * 1024 // 50MB
        memoryCache.countLimit = 100
    }
}
```

## VoIP 특화 캠싱 정책
```swift
enum CachePolicy {
    case contactImages     // 30일 유지
    case callHistory      // 무제한 (사용자 삭제까지)
    case voicemail        // 7일 후 자동 삭제
    case audioCodecs      // 앱 업데이트까지
    case stunServers      // 24시간 TTL
}
```

## 오프라인 지원
```swift
class OfflineCache {
    func cacheForOffline() {
        // 최근 통화 연락처 미리 로드
        let recentContacts = fetchRecentContacts(limit: 50)
        cacheContacts(recentContacts)
        
        // STUN/TURN 서버 정보 캠싱
        cacheServerConfiguration()
        
        // 코덱 설정 캠싱
        cacheAudioCodecs()
    }
}
```

## 캠시 무효화 전략
```swift
extension CacheManager {
    func invalidate(for key: String) {
        // 메모리에서 즉시 제거
        memoryCache.removeObject(forKey: key as NSString)
        
        // 디스크에서 비동기 제거
        DispatchQueue.global().async {
            try? FileManager.default.removeItem(at: self.diskURL(for: key))
        }
    }
    
    func clearExpiredCache() {
        // Background task로 만료된 캠시 정리
        let expirationDate = Date().addingTimeInterval(-7 * 24 * 60 * 60)
        // ...
    }
}
```

## 연관 개념
- [[call_history_storage]]
- [[battery_optimization]]
- [[bandwidth_adaptation]]

## 태그
#cache #performance #offline #storage #optimization