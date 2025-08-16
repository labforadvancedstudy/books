# 국가별 규제 대응

## 개녕
각국의 VoIP 규제, 개인정보 보호법, 통신 법규를 준수하여 합법적으로 서비스 운영.

## 국가별 VoIP 규제
```swift
enum CountryRegulation {
    case usa
    case eu
    case china
    case india
    case uae
    case russia
    
    var requirements: RegulationRequirements {
        switch self {
        case .usa:
            return RegulationRequirements(
                e911Required: true,           // 긴급통화 의무
                dataRetention: 90,            // 데이터 보관 일수
                encryptionRequired: false,    // 암호화 선택
                localServerRequired: false,   // 로컬 서버 필수 여부
                recordingConsent: .oneParty   // 녹음 동의
            )
        case .eu:
            return RegulationRequirements(
                e911Required: false,
                dataRetention: 180,           // GDPR 준수
                encryptionRequired: true,     // 개인정보 암호화
                localServerRequired: true,    // 데이터 지역화
                recordingConsent: .allParties // 모든 참가자 동의
            )
        case .china:
            return RegulationRequirements(
                e911Required: false,
                dataRetention: 365,           // 1년 보관
                encryptionRequired: true,
                localServerRequired: true,    // 중국 내 서버 필수
                recordingConsent: .notRequired,
                realNameVerification: true    // 실명 인증
            )
        case .uae:
            return RegulationRequirements(
                e911Required: false,
                dataRetention: 730,           // 2년 보관
                encryptionRequired: false,
                localServerRequired: false,
                recordingConsent: .prohibited, // VoIP 제한
                voipRestricted: true
            )
        default:
            return RegulationRequirements.default
        }
    }
}
```

## E911 (미국 긴급통화) 지원
```swift
class E911Compliance {
    struct EmergencyAddress {
        let streetAddress: String
        let city: String
        let state: String
        let zipCode: String
        let country: String = "US"
        
        var isValid: Bool {
            // 주소 검증
            return !streetAddress.isEmpty &&
                   !city.isEmpty &&
                   !state.isEmpty &&
                   zipCode.count == 5
        }
    }
    
    static func register911Address(_ address: EmergencyAddress) async throws {
        guard address.isValid else {
            throw E911Error.invalidAddress
        }
        
        // 911 데이터베이스 등록
        let request = E911RegistrationRequest(
            address: address,
            phoneNumber: getUserPhoneNumber(),
            deviceId: getDeviceIdentifier()
        )
        
        try await submitTo911Database(request)
        
        // 로컬 저장
        UserDefaults.standard.set(
            try? JSONEncoder().encode(address),
            forKey: "emergency_address"
        )
    }
    
    static func handleEmergencyCall() {
        // 일반 CallKit 대신 네이티브 다이얼러로 전환
        if let phoneURL = URL(string: "tel://911") {
            UIApplication.shared.open(phoneURL)
        }
        
        // 위치 정보 전송
        sendLocationToEmergencyServices()
    }
}
```

## GDPR 준수 (EU)
```swift
class GDPRCompliance {
    enum ConsentType {
        case dataCollection
        case callRecording
        case marketing
        case analytics
    }
    
    static func requestConsent(
        for types: [ConsentType],
        completion: @escaping ([ConsentType: Bool]) -> Void
    ) {
        let consentVC = GDPRConsentViewController()
        consentVC.consentTypes = types
        consentVC.onComplete = { consents in
            // 동의 기록
            self.recordConsent(consents)
            completion(consents)
        }
        
        presentConsentDialog(consentVC)
    }
    
    static func exportUserData(userId: String) async -> UserDataPackage {
        // GDPR Article 20 - 데이터 이동권
        let userData = await gatherAllUserData(userId)
        
        return UserDataPackage(
            personalInfo: userData.profile,
            callHistory: userData.calls,
            recordings: userData.recordings,
            messages: userData.messages,
            exportDate: Date(),
            format: .json
        )
    }
    
    static func deleteUserData(userId: String) async {
        // GDPR Article 17 - 삭제권 (Right to be forgotten)
        await deleteFromDatabase(userId)
        await deleteFromBackups(userId)
        await deleteFromAnalytics(userId)
        
        // 삭제 증명
        await sendDeletionConfirmation(userId)
    }
}
```

## 통화 녹음 규제
```swift
class RecordingCompliance {
    enum ConsentRequirement {
        case oneParty      // 한 명만 동의 (미국 일부 주)
        case allParties    // 모든 참가자 동의 (EU, 캨리포니아)
        case notRequired   // 동의 불필요
        case prohibited    // 녹음 금지
    }
    
    static func canRecord(in country: String, with participants: [String]) -> Bool {
        let requirement = getConsentRequirement(for: country)
        
        switch requirement {
        case .prohibited:
            return false
            
        case .notRequired:
            return true
            
        case .oneParty:
            // 최소 1명 동의
            return hasAtLeastOneConsent(participants)
            
        case .allParties:
            // 모든 참가자 동의 필요
            return haveAllConsents(participants)
        }
    }
    
    static func announceRecording() {
        // 녹음 안내 방송
        AudioPlayer.play("recording_announcement.mp3")
        
        // UI 표시
        showRecordingIndicator()
    }
}
```

## 데이터 지역화 요구사항
```swift
class DataLocalization {
    static func selectServer(for country: String) -> ServerEndpoint {
        switch country {
        case "CN":
            // 중국: 국내 서버 필수
            return ServerEndpoint(region: .china, url: "voip.cn-north-1.amazonaws.com.cn")
            
        case "RU":
            // 러시아: 로컬 데이터 저장
            return ServerEndpoint(region: .russia, url: "voip.ru-central-1.yandexcloud.net")
            
        case let eu where EUCountries.contains(eu):
            // EU: GDPR 준수 서버
            return ServerEndpoint(region: .eu, url: "voip.eu-west-1.amazonaws.com")
            
        default:
            // 기본: 미국 서버
            return ServerEndpoint(region: .us, url: "voip.us-east-1.amazonaws.com")
        }
    }
    
    static func configureDataRetention(for country: String) {
        let retentionDays = getRetentionRequirement(for: country)
        
        // 자동 삭제 설정
        DataRetentionPolicy.configure(
            callLogs: retentionDays,
            recordings: min(retentionDays, 30), // 녹음은 최대 30일
            messages: retentionDays
        )
    }
}
```

## 암호화 수출 규제
```swift
class EncryptionExportCompliance {
    // 미국 수출 규제 (EAR)
    static func declareEncryption() -> ExportComplianceInfo {
        return ExportComplianceInfo(
            usesEncryption: true,
            encryptionType: .standard,    // AES-256, RSA
            exemptionType: .5D992c,       // Mass market encryption
            ccatsNumber: nil               // 자체 분류
        )
    }
    
    static func restrictedCountries() -> [String] {
        // 수출 금지 국가
        return ["CU", "IR", "KP", "SD", "SY"]
    }
    
    static func canExportTo(country: String) -> Bool {
        return !restrictedCountries().contains(country)
    }
}
```

## 연관 개녕
- [[privacy_permissions]]
- [[export_compliance]]
- [[e2e_encryption]]
- [[localization_i18n]]

## 태그
#regulations #compliance #legal #privacy #gdpr #e911