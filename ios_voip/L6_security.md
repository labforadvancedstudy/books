# L6: 보안과 프라이버시

## Core Insight
VoIP 보안은 단순히 암호화가 아니다. 신원 확인, 키 교환, 메타데이터 보호, 그리고 인간의 실수 방지까지 포함하는 전체적인 시스템이다.

---

## 위협 모델: 누가 듣고 있나?

VoIP 통화를 노리는 적들:

```swift
enum ThreatActor {
    case passiveEavesdropper    // 네트워크 스니핑
    case activeAttacker         // MITM (Man-in-the-Middle)
    case maliciousServer        // 악의적인 서버
    case compromisedDevice      // 해킹된 단말
    case legalInterception      // 합법적 감청
    case metadataCollector      // 통화 패턴 분석
}

struct ThreatModel {
    let actors: [ThreatActor]
    let assets: [String] = [
        "음성 내용",
        "발신자/수신자 정보",
        "통화 시간과 빈도",
        "위치 정보",
        "연락처 목록"
    ]
    
    func calculateRisk() -> RiskLevel {
        // 위협 수준 평가
        if actors.contains(.activeAttacker) {
            return .critical
        } else if actors.contains(.passiveEavesdropper) {
            return .high
        }
        return .medium
    }
}
```

## End-to-End 암호화: 진짜 E2EE

많은 앱이 E2EE를 주장하지만, 진짜는 드물다.

### Signal Protocol 구현

```swift
import CryptoKit
import SignalProtocol

class EndToEndEncryption {
    private let localStorage = SignalProtocolStore()
    private var sessionCipher: SessionCipher?
    
    // 초기 키 교환 (X3DH)
    func performKeyExchange(with remoteUser: String) async throws {
        // 1. Identity Key Pair (장기 키)
        let identityKeyPair = Curve25519.KeyAgreement.PrivateKey()
        
        // 2. Signed Pre-Key (중기 키)
        let signedPreKey = Curve25519.KeyAgreement.PrivateKey()
        let signature = try identityKeyPair.signature(
            for: signedPreKey.publicKey.rawRepresentation
        )
        
        // 3. One-Time Pre-Keys (일회용 키들)
        var oneTimePreKeys: [Curve25519.KeyAgreement.PrivateKey] = []
        for _ in 0..<100 {
            oneTimePreKeys.append(Curve25519.KeyAgreement.PrivateKey())
        }
        
        // 4. 서버에 공개키 번들 업로드
        let bundle = PreKeyBundle(
            identityKey: identityKeyPair.publicKey,
            signedPreKey: signedPreKey.publicKey,
            signature: signature,
            oneTimePreKeys: oneTimePreKeys.map { $0.publicKey }
        )
        
        await uploadPreKeyBundle(bundle)
    }
    
    // 세션 초기화
    func initializeSession(with remoteUser: String) async throws {
        // 1. 상대방의 Pre-Key Bundle 가져오기
        let bundle = try await fetchPreKeyBundle(for: remoteUser)
        
        // 2. X3DH 프로토콜로 공유 비밀 생성
        let ephemeralKey = Curve25519.KeyAgreement.PrivateKey()
        
        // DH1: IK_A -> SPK_B
        let dh1 = try identityKey.sharedSecretFromKeyAgreement(
            with: bundle.signedPreKey
        )
        
        // DH2: EK_A -> IK_B
        let dh2 = try ephemeralKey.sharedSecretFromKeyAgreement(
            with: bundle.identityKey
        )
        
        // DH3: EK_A -> SPK_B
        let dh3 = try ephemeralKey.sharedSecretFromKeyAgreement(
            with: bundle.signedPreKey
        )
        
        // DH4: EK_A -> OPK_B (if available)
        var dh4: SharedSecret?
        if let oneTimeKey = bundle.oneTimePreKey {
            dh4 = try ephemeralKey.sharedSecretFromKeyAgreement(
                with: oneTimeKey
            )
        }
        
        // 3. Master Secret 도출
        let masterSecret = deriveMasterSecret(dh1, dh2, dh3, dh4)
        
        // 4. Double Ratchet 초기화
        let sessionCipher = SessionCipher(
            remoteAddress: remoteUser,
            masterSecret: masterSecret
        )
        
        self.sessionCipher = sessionCipher
    }
}
```

### Double Ratchet 알고리즘

```swift
class DoubleRatchet {
    private var rootKey: SymmetricKey
    private var sendingChainKey: SymmetricKey
    private var receivingChainKey: SymmetricKey?
    private var sendMessageNumber = 0
    private var receiveMessageNumber = 0
    
    // 메시지 암호화
    func encrypt(_ message: Data) throws -> EncryptedMessage {
        // 1. 메시지 키 도출
        let messageKey = deriveMessageKey(from: sendingChainKey, 
                                         index: sendMessageNumber)
        
        // 2. AES-GCM 암호화
        let nonce = AES.GCM.Nonce()
        let sealedBox = try AES.GCM.seal(message, 
                                         using: messageKey, 
                                         nonce: nonce)
        
        // 3. 체인 키 래칫
        sendingChainKey = ratchetChainKey(sendingChainKey)
        sendMessageNumber += 1
        
        // 4. 헤더 생성 (현재 래칫 공개키, 메시지 번호 등)
        let header = MessageHeader(
            ratchetKey: currentRatchetKey.publicKey,
            previousChainLength: previousChainLength,
            messageNumber: sendMessageNumber
        )
        
        return EncryptedMessage(header: header, 
                               ciphertext: sealedBox.ciphertext)
    }
    
    // DH 래칫 (상대방 키 받았을 때)
    func performDHRatchet(remoteKey: Curve25519.KeyAgreement.PublicKey) {
        // 1. 새 DH 키 쌍 생성
        let newKeyPair = Curve25519.KeyAgreement.PrivateKey()
        
        // 2. 공유 비밀 계산
        let sharedSecret = try! newKeyPair.sharedSecretFromKeyAgreement(
            with: remoteKey
        )
        
        // 3. 루트 키와 체인 키 업데이트
        let (newRootKey, newChainKey) = kdfRootKey(
            rootKey: rootKey,
            dhOutput: sharedSecret
        )
        
        rootKey = newRootKey
        receivingChainKey = newChainKey
        
        // 4. 송신 체인도 업데이트
        let (nextRootKey, nextSendingKey) = kdfRootKey(
            rootKey: newRootKey,
            dhOutput: try! newKeyPair.sharedSecretFromKeyAgreement(
                with: remoteKey
            )
        )
        
        rootKey = nextRootKey
        sendingChainKey = nextSendingKey
    }
}
```

## SRTP: 미디어 스트림 보호

WebRTC는 SRTP로 미디어를 보호한다:

```swift
class SRTPProtection {
    private var srtpContext: OpaquePointer?
    
    init(masterKey: Data, masterSalt: Data) {
        // libsrtp 초기화
        srtp_init()
        
        // SRTP 정책 설정
        var policy = srtp_policy_t()
        
        // 암호화 알고리즘
        srtp_crypto_policy_set_aes_gcm_128_16_auth(&policy.rtp)
        srtp_crypto_policy_set_aes_gcm_128_16_auth(&policy.rtcp)
        
        // 키 설정
        policy.key = UnsafeMutablePointer<UInt8>.allocate(capacity: 30)
        masterKey.withUnsafeBytes { keyBytes in
            masterSalt.withUnsafeBytes { saltBytes in
                memcpy(policy.key, keyBytes.baseAddress, 16)
                memcpy(policy.key.advanced(by: 16), saltBytes.baseAddress, 14)
            }
        }
        
        // Context 생성
        srtp_create(&srtpContext, &policy)
    }
    
    func protectRTP(_ packet: inout Data) -> Bool {
        var length = packet.count
        
        let result = packet.withUnsafeMutableBytes { bytes in
            srtp_protect(srtpContext, bytes.baseAddress, &length)
        }
        
        if result == srtp_err_status_ok {
            packet = packet.prefix(length)
            return true
        }
        return false
    }
    
    func unprotectRTP(_ packet: inout Data) -> Bool {
        var length = packet.count
        
        let result = packet.withUnsafeMutableBytes { bytes in
            srtp_unprotect(srtpContext, bytes.baseAddress, &length)
        }
        
        if result == srtp_err_status_ok {
            packet = packet.prefix(length)
            return true
        }
        return false
    }
}
```

## ZRTP: 키 교환 without 서버

ZRTP는 서버 없이 키를 교환한다:

```swift
class ZRTPKeyExchange {
    private let sas = ShortAuthenticationString()
    
    // SAS (Short Authentication String) 생성
    func performKeyExchange() async -> String {
        // 1. Hello 메시지 교환
        let hello = ZRTPHello(
            version: "1.10",
            clientId: "MyVoIPApp",
            hashAlgorithms: [.sha256, .sha384],
            cipherAlgorithms: [.aes128, .aes256],
            authTags: [.hs32, .hs80],
            keyAgreement: [.dh3k, .ec25],
            sas: [.b32, .b256]
        )
        
        await sendHello(hello)
        let remoteHello = await receiveHello()
        
        // 2. Commit 메시지 (initiator만)
        let commit = ZRTPCommit(
            hashAlgorithm: .sha256,
            cipherAlgorithm: .aes128,
            authTag: .hs32,
            keyAgreement: .dh3k,
            sas: .b32
        )
        
        await sendCommit(commit)
        
        // 3. DH 교환
        let dhPart1 = await receiveDHPart1()
        let dhPart2 = generateDHPart2()
        await sendDHPart2(dhPart2)
        
        // 4. 공유 비밀 계산
        let sharedSecret = computeSharedSecret(dhPart1, dhPart2)
        
        // 5. SAS 생성 (사용자가 확인)
        let sasValue = generateSAS(from: sharedSecret)
        
        // 6. Confirm 메시지
        await sendConfirm()
        await receiveConfirm()
        
        return sasValue  // "alpha bravo charlie delta"
    }
    
    // SAS를 단어로 변환
    func generateSAS(from secret: Data) -> String {
        let words = [
            "alpha", "bravo", "charlie", "delta", "echo",
            "foxtrot", "golf", "hotel", "india", "juliet",
            // ... PGP word list
        ]
        
        let hash = SHA256.hash(data: secret)
        let bytes = Array(hash.prefix(4))
        
        return bytes.map { words[Int($0) % words.count] }.joined(separator: " ")
    }
}
```

## 메타데이터 보호

암호화해도 메타데이터는 노출된다:

```swift
class MetadataProtection {
    // Tor 같은 Onion Routing
    func routeThroughOnion(packet: Data, path: [RelayNode]) -> Data {
        var wrapped = packet
        
        // 역순으로 암호화 (양파 껍질처럼)
        for node in path.reversed() {
            let encrypted = node.publicKey.encrypt(wrapped)
            wrapped = encrypted
        }
        
        return wrapped
    }
    
    // 더미 트래픽 생성
    func generateDummyTraffic() {
        Timer.scheduledTimer(withTimeInterval: 0.02, repeats: true) { _ in
            if !self.isActiveCall {
                // 무음 패킷 전송
                let dummyPacket = self.createDummyPacket()
                self.sendPacket(dummyPacket)
            }
        }
    }
    
    // 패킷 크기 정규화
    func normalizePacM: Data) -> Data {
        let standardSize = 160  // bytes
        
        if packet.count < standardSize {
            // 패딩 추가
            var padded = packet
            padded.append(Data(repeating: 0, 
                              count: standardSize - packet.count))
            return padded
        } else if packet.count > standardSize {
            // 분할
            return packet.prefix(standardSize)
        }
        
        return packet
    }
}
```

## iOS 보안 기능 활용

### Keychain에 안전하게 저장

```swift
class SecureStorage {
    // 비밀키 저장
    func savePrivateKey(_ key: Data, identifier: String) throws {
        let query: [String: Any] = [
            kSecClass as String: kSecClassKey,
            kSecAttrApplicationTag as String: identifier,
            kSecAttrKeyType as String: kSecAttrKeyTypeECSECPrimeRandom,
            kSecValueData as String: key,
            kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly,
            kSecAttrSynchronizable as String: false
        ]
        
        let status = SecItemAdd(query as CFDictionary, nil)
        
        if status != errSecSuccess {
            throw KeychainError.saveFailed(status)
        }
    }
    
    // Secure Enclave 활용
    func generateKeyInSecureEnclave() throws -> SecKey {
        let attributes: [String: Any] = [
            kSecAttrKeyType as String: kSecAttrKeyTypeECSECPrimeRandom,
            kSecAttrKeySizeInBits as String: 256,
            kSecAttrTokenID as String: kSecAttrTokenIDSecureEnclave,
            kSecPrivateKeyAttrs as String: [
                kSecAttrIsPermanent as String: true,
                kSecAttrApplicationTag as String: "com.app.voip.key",
                kSecAttrAccessControl as String: SecAccessControlCreateWithFlags(
                    kCFAllocatorDefault,
                    kSecAttrAccessibleWhenUnlockedThisDeviceOnly,
                    [.privateKeyUsage, .biometryCurrentSet],
                    nil
                )!
            ]
        ]
        
        var error: Unmanaged<CFError>?
        guard let privateKey = SecKeyCreateRandomKey(
            attributes as CFDictionary,
            &error
        ) else {
            throw error!.takeRetainedValue() as Error
        }
        
        return privateKey
    }
}
```

### 생체 인증

```swift
import LocalAuthentication

class BiometricAuth {
    func authenticateUser() async -> Bool {
        let context = LAContext()
        var error: NSError?
        
        // 생체 인증 가능 여부 확인
        guard context.canEvaluatePolicy(
            .deviceOwnerAuthenticationWithBiometrics,
            error: &error
        ) else {
            return false
        }
        
        // 인증 수행
        do {
            let result = try await context.evaluatePolicy(
                .deviceOwnerAuthenticationWithBiometrics,
                localizedReason: "통화를 시작하려면 인증이 필요합니다"
            )
            return result
        } catch {
            return false
        }
    }
}
```

## 인증서 고정 (Certificate Pinning)

```swift
class CertificatePinning: NSObject, URLSessionDelegate {
    private let pinnedCertificates: [SecCertificate]
    
    init(certificates: [SecCertificate]) {
        self.pinnedCertificates = certificates
        super.init()
    }
    
    func urlSession(_ session: URLSession,
                   didReceive challenge: URLAuthenticationChallenge,
                   completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
        
        guard challenge.protectionSpace.authenticationMethod == 
              NSURLAuthenticationMethodServerTrust,
              let serverTrust = challenge.protectionSpace.serverTrust else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        
        // 서버 인증서 체인 검증
        var secresult = SecTrustResultType.invalid
        SecTrustEvaluate(serverTrust, &secresult)
        
        // 인증서 비교
        if let serverCertificate = SecTrustGetCertificateAtIndex(serverTrust, 0) {
            let serverCertData = SecCertificateCopyData(serverCertificate) as Data
            
            for pinnedCert in pinnedCertificates {
                let pinnedCertData = SecCertificateCopyData(pinnedCert) as Data
                
                if serverCertData == pinnedCertData {
                    // 인증서 일치
                    let credential = URLCredential(trust: serverTrust)
                    completionHandler(.useCredential, credential)
                    return
                }
            }
        }
        
        // 인증서 불일치
        completionHandler(.cancelAuthenticationChallenge, nil)
    }
}
```

## 보안 감사 로깅

```swift
class SecurityAuditLog {
    private let logFile: URL
    private let logQueue = DispatchQueue(label: "security.audit")
    
    enum SecurityEvent {
        case authenticationAttempt(user: String, success: Bool)
        case keyExchange(peer: String, algorithm: String)
        case encryptionStarted(cipher: String)
        case suspiciousActivity(description: String)
        case certificateValidation(host: String, result: Bool)
    }
    
    func log(_ event: SecurityEvent) {
        logQueue.async {
            let entry = self.formatLogEntry(event)
            self.writeToFile(entry)
            
            // 심각한 이벤트는 즉시 서버로 전송
            if self.isCritical(event) {
                self.sendToServer(entry)
            }
        }
    }
    
    private func formatLogEntry(_ event: SecurityEvent) -> String {
        let timestamp = ISO8601DateFormatter().string(from: Date())
        let eventData = serializeEvent(event)
        
        // HMAC으로 무결성 보호
        let hmac = HMAC<SHA256>.authenticationCode(
            for: eventData.data(using: .utf8)!,
            using: logSigningKey
        )
        
        return "\(timestamp) | \(eventData) | \(hmac.hexString)"
    }
}
```

## 취약점 방어

### 재전송 공격 방지

```swift
class ReplayAttackPrevention {
    private var usedNonces = Set<String>()
    private let nonceTimeout: TimeInterval = 300  // 5분
    
    func validateMessage(_ message: SecureMessage) -> Bool {
        // 1. 타임스탬프 확인
        let messageTime = message.timestamp
        let currentTime = Date()
        
        if abs(currentTime.timeIntervalSince(messageTime)) > nonceTimeout {
            return false  // 너무 오래된 메시지
        }
        
        // 2. Nonce 중복 확인
        if usedNonces.contains(message.nonce) {
            return false  // 재전송된 메시지
        }
        
        // 3. Nonce 저장
        usedNonces.insert(message.nonce)
        
        // 4. 오래된 Nonce 정리
        cleanupOldNonces()
        
        return true
    }
}
```

### SQL Injection 방지

```swift
class SecureDatabase {
    func searchContacts(query: String) -> [Contact] {
        // ❌ 위험한 방법
        // let sql = "SELECT * FROM contacts WHERE name = '\(query)'"
        
        // ✅ 안전한 방법 (Prepared Statement)
        let sql = "SELECT * FROM contacts WHERE name = ?"
        
        do {
            let statement = try db.prepare(sql)
            try statement.bind(query, at: 1)
            
            var contacts: [Contact] = []
            while try statement.step() {
                contacts.append(Contact(from: statement))
            }
            return contacts
        } catch {
            return []
        }
    }
}
```

## 프라이버시 설정

```swift
class PrivacySettings: ObservableObject {
    @Published var allowContactSync = false
    @Published var shareLastSeen = false
    @Published var showOnlineStatus = false
    @Published var allowCallRecording = false
    @Published var requireBiometricAuth = true
    
    // 메타데이터 최소화
    func minimizeMetadata() {
        // 1. 연락처 해시만 동기화
        if allowContactSync {
            syncHashedContacts()
        }
        
        // 2. 정확한 시간 대신 범위로
        if shareLastSeen {
            shareApproximateTime()  // "5분 전" instead of "17:42:31"
        }
        
        // 3. 프로필 사진 압축
        compressProfileImages()
    }
    
    private func syncHashedContacts() {
        let contacts = fetchLocalContacts()
        let hashedContacts = contacts.map { contact in
            SHA256.hash(data: contact.phoneNumber.data(using: .utf8)!)
        }
        
        // 해시만 서버로 전송
        server.syncContacts(hashedContacts)
    }
}
```

## 보안 체크리스트

```swift
struct SecurityChecklist {
    let items = [
        "✓ E2EE 구현 및 검증",
        "✓ 인증서 고정",
        "✓ 안전한 키 저장 (Keychain/Secure Enclave)",
        "✓ 생체 인증 통합",
        "✓ 보안 감사 로깅",
        "✓ 재전송 공격 방지",
        "✓ SQL Injection 방지",
        "✓ 메타데이터 최소화",
        "✓ 정기적인 보안 업데이트",
        "✓ 취약점 스캐닝",
        "✓ 침투 테스트",
        "✓ 사용자 교육"
    ]
    
    func performSecurityAudit() -> SecurityAuditReport {
        var report = SecurityAuditReport()
        
        // 자동화된 검사들
        report.e2eeStatus = checkE2EEImplementation()
        report.certificateStatus = checkCertificatePinning()
        report.keyStorageStatus = checkKeyStorage()
        // ...
        
        return report
    }
}
```

## 긴급 상황 대응

```swift
class SecurityIncidentResponse {
    enum IncidentType {
        case dataBrearh
        case unauthorizedAccess
        case cryptographicFailure
        case ddosAttack
    }
    
    func handleIncident(_ type: IncidentType) {
        switch type {
        case .dataBreach:
            // 1. 영향받은 세션 즉시 종료
            terminateAllSessions()
            
            // 2. 키 재생성
            regenerateAllKeys()
            
            // 3. 사용자 알림
            notifyAffectedUsers()
            
            // 4. 로그 수집
            collectForensicData()
            
        case .cryptographicFailure:
            // 백업 알고리즘으로 전환
            fallbackToSecondaryEncryption()
            
        default:
            break
        }
    }
}
```

## 마치며

보안은 한 번 구현하고 끝나는 게 아니다.
지속적인 감시, 업데이트, 그리고 개선이 필요하다.

"충분히 안전한" 시스템은 없다.
하지만 "공격 비용 > 이익"인 시스템은 만들 수 있다.

다음 레벨에서는 이 모든 보안 기능이
성능에 미치는 영향을 최소화하는 방법을 다룬다.

---

## Connections

→ [[L7_optimization]] - 성능 최적화
→ [[L8_deployment]] - 안전한 배포

← [[L5_audio_processing]] - 오디오 처리로 돌아가기
← [[index]] - 목차

---

Level: L6
Date: 2025-08-15
Tags: #security #encryption #e2ee #privacy #ios