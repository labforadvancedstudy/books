# 한국 개인정보보호법 & 앱 심사 대응 가이드

## 개인정보보호법 (PIPA) 완벽 준수

```swift
import SwiftUI
import CryptoKit
import LocalAuthentication

// MARK: - Privacy Manager
class PrivacyManager: NSObject {
    static let shared = PrivacyManager()
    
    private let encryptionKey = SymmetricKey(size: .bits256)
    private let biometricContext = LAContext()
    
    // MARK: - 개인정보 수집 동의
    struct PrivacyConsent: Codable {
        let userId: String
        let timestamp: Date
        let ipAddress: String
        let deviceId: String
        
        // 필수 동의 항목
        var termsOfService: Bool = false
        var privacyPolicy: Bool = false
        var dataCollection: Bool = false
        
        // 선택 동의 항목
        var marketingEmail: Bool = false
        var marketingSMS: Bool = false
        var marketingPush: Bool = false
        var thirdPartySharing: Bool = false
        var locationTracking: Bool = false
        var advertisingId: Bool = false
        
        // 민감정보 동의
        var healthData: Bool = false
        var biometricData: Bool = false
        
        // 아동 보호 (만 14세 미만)
        var isUnder14: Bool = false
        var parentalConsent: Bool = false
        var parentalVerification: ParentalVerification?
        
        // 동의 철회 기록
        var withdrawals: [WithdrawalRecord] = []
    }
    
    struct ParentalVerification: Codable {
        let parentName: String
        let parentPhone: String
        let verificationMethod: String // 휴대폰 인증, 아이핀, 신용카드
        let verificationDate: Date
        let verificationCode: String
    }
    
    struct WithdrawalRecord: Codable {
        let consentType: String
        let withdrawalDate: Date
        let reason: String?
    }
    
    // MARK: - 개인정보 암호화
    func encryptPersonalData<T: Codable>(_ data: T) throws -> Data {
        let encoder = JSONEncoder()
        let plainData = try encoder.encode(data)
        
        let sealedBox = try AES.GCM.seal(plainData, using: encryptionKey)
        return sealedBox.combined ?? Data()
    }
    
    func decryptPersonalData<T: Codable>(_ encryptedData: Data, type: T.Type) throws -> T {
        let sealedBox = try AES.GCM.SealedBox(combined: encryptedData)
        let decryptedData = try AES.GCM.open(sealedBox, using: encryptionKey)
        
        let decoder = JSONDecoder()
        return try decoder.decode(type, from: decryptedData)
    }
    
    // MARK: - 안전한 데이터 저장
    func savePersonalDataSecurely(_ data: PersonalData) throws {
        // 암호화
        let encryptedData = try encryptPersonalData(data)
        
        // Keychain에 저장
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: "com.recipebox.personaldata.\(data.userId)",
            kSecValueData as String: encryptedData,
            kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
        ]
        
        SecItemDelete(query as CFDictionary) // 기존 데이터 삭제
        let status = SecItemAdd(query as CFDictionary, nil)
        
        if status != errSecSuccess {
            throw PrivacyError.keychainSaveFailed
        }
        
        // 로그 기록 (개인정보는 제외)
        logDataAccess(
            action: "SAVE",
            dataType: "PersonalData",
            userId: data.userId,
            purpose: "User Registration"
        )
    }
    
    // MARK: - 데이터 접근 로그
    func logDataAccess(action: String, dataType: String, userId: String, purpose: String) {
        let log = DataAccessLog(
            timestamp: Date(),
            action: action,
            dataType: dataType,
            userId: anonymizeUserId(userId),
            purpose: purpose,
            ipAddress: getIPAddress(),
            deviceId: getDeviceId()
        )
        
        // 로그 저장 (6개월 보관)
        saveAccessLog(log)
    }
    
    private func anonymizeUserId(_ userId: String) -> String {
        // 사용자 ID 익명화
        let data = Data(userId.utf8)
        let hashed = SHA256.hash(data: data)
        return hashed.compactMap { String(format: "%02x", $0) }.joined().prefix(16).uppercased()
    }
    
    // MARK: - 개인정보 파기
    func deletePersonalData(userId: String, reason: String) throws {
        // 즉시 파기 대상
        let immediateDeleteItems = [
            "personalInfo",
            "paymentInfo",
            "healthData",
            "locationData"
        ]
        
        // 법정 보관 기간이 있는 데이터는 별도 처리
        let retentionRequiredItems = [
            "transactionRecords": 5, // 5년 (전자상거래법)
            "taxInvoices": 5,        // 5년 (국세기본법)
            "consumerComplaints": 3,  // 3년 (전자상거래법)
            "accessLogs": 3,         // 3년 (통신비밀보호법)
            "marketingHistory": 2    // 2년 (정보통신망법)
        ]
        
        // 즉시 파기
        for item in immediateDeleteItems {
            deleteFromKeychain(key: "\(userId).\(item)")
        }
        
        // 파기 예정 데이터 분리 보관
        for (item, retentionYears) in retentionRequiredItems {
            markForDeletion(
                key: "\(userId).\(item)",
                deleteAfter: Calendar.current.date(byAdding: .year, value: retentionYears, to: Date())!
            )
        }
        
        // 파기 증적 생성
        let certificate = DeletionCertificate(
            userId: anonymizeUserId(userId),
            deletionDate: Date(),
            reason: reason,
            deletedItems: immediateDeleteItems,
            retainedItems: retentionRequiredItems.map { $0.key }
        )
        
        saveDeletionCertificate(certificate)
    }
    
    // MARK: - 제3자 제공 관리
    func shareDataWithThirdParty(
        userId: String,
        recipient: String,
        dataTypes: [String],
        purpose: String,
        retentionPeriod: Int
    ) throws {
        // 사용자 동의 확인
        guard hasConsentForThirdPartySharing(userId: userId) else {
            throw PrivacyError.noConsentForSharing
        }
        
        // 제공 내역 기록
        let sharingRecord = ThirdPartySharingRecord(
            userId: userId,
            recipient: recipient,
            dataTypes: dataTypes,
            purpose: purpose,
            sharingDate: Date(),
            retentionPeriod: retentionPeriod,
            consentDate: getConsentDate(userId: userId, type: "thirdPartySharing")
        )
        
        saveSharingRecord(sharingRecord)
        
        // 사용자 통지
        notifyUserOfDataSharing(userId: userId, record: sharingRecord)
    }
    
    // MARK: - 개인정보 이용내역 통지
    func generatePrivacyUsageReport(userId: String) -> PrivacyUsageReport {
        let report = PrivacyUsageReport(
            userId: userId,
            reportDate: Date(),
            collectedData: getCollectedDataTypes(userId: userId),
            usagePurposes: getUsagePurposes(userId: userId),
            thirdPartySharing: getThirdPartySharingRecords(userId: userId),
            retentionPeriod: getRetentionPeriods(),
            userRights: getUserRights(),
            contactInfo: getPrivacyContactInfo()
        )
        
        return report
    }
}

// MARK: - 한국 앱스토어 심사 대응
class AppStoreComplianceManager {
    static let shared = AppStoreComplianceManager()
    
    // MARK: - 심사 체크리스트
    struct ReviewChecklist {
        // 필수 준비 사항
        let privacyPolicyURL = "https://recipebox.app/privacy"
        let termsOfServiceURL = "https://recipebox.app/terms"
        let supportURL = "https://recipebox.app/support"
        let supportEmail = "support@recipebox.app"
        
        // 데모 계정
        let demoAccount = DemoAccount(
            username: "apple_review@recipebox.app",
            password: "AppleReview2024!",
            verificationCode: "123456"
        )
        
        // 심사 노트
        let reviewNotes = """
        안녕하세요, Apple 심사팀님
        
        RecipeBox 앱 심사를 위한 안내사항입니다:
        
        1. 데모 계정
        - 이메일: apple_review@recipebox.app
        - 비밀번호: AppleReview2024!
        - 인증 코드: 123456
        
        2. 결제 테스트
        - 테스트 카드 번호: 4242-4242-4242-4242
        - 유효기간: 12/25
        - CVC: 123
        
        3. 위치 권한
        - 주변 음식점 검색 기능을 위해 필요합니다
        - 사용자가 명시적으로 허용한 경우에만 사용됩니다
        
        4. 카메라 권한
        - 레시피 사진 촬영 기능을 위해 필요합니다
        - 선택적 기능이며 거부해도 앱 사용에 지장 없습니다
        
        5. 한국 특화 기능
        - 카카오/네이버 로그인은 한국 사용자를 위한 기능입니다
        - 해외에서는 일반 이메일 로그인을 사용할 수 있습니다
        
        감사합니다.
        """
    }
    
    // MARK: - 메타데이터 검증
    func validateMetadata() -> [ValidationResult] {
        var results: [ValidationResult] = []
        
        // 앱 이름 검증
        results.append(validateAppName())
        
        // 설명 검증
        results.append(validateDescription())
        
        // 키워드 검증
        results.append(validateKeywords())
        
        // 스크린샷 검증
        results.append(validateScreenshots())
        
        // 연령 등급 검증
        results.append(validateAgeRating())
        
        return results
    }
    
    private func validateAppName() -> ValidationResult {
        let appName = "RecipeBox - 한국 No.1 레시피"
        var issues: [String] = []
        
        // 길이 체크 (30자 제한)
        if appName.count > 30 {
            issues.append("앱 이름이 30자를 초과합니다")
        }
        
        // 금지 단어 체크
        let prohibitedWords = ["무료", "세일", "최고", "No.1", "베스트"]
        for word in prohibitedWords {
            if appName.contains(word) {
                issues.append("'\(word)'는 앱 이름에 사용할 수 없습니다")
            }
        }
        
        return ValidationResult(
            category: "앱 이름",
            isValid: issues.isEmpty,
            issues: issues
        )
    }
    
    // MARK: - In-App Purchase 검증
    func validateIAPImplementation() -> Bool {
        // StoreKit 2 구현 확인
        let hasStoreKit = checkStoreKitImplementation()
        
        // 복원 기능 확인
        let hasRestore = checkRestorePurchases()
        
        // 구독 관리 확인
        let hasSubscriptionManagement = checkSubscriptionManagement()
        
        // 환불 정책 확인
        let hasRefundPolicy = checkRefundPolicy()
        
        return hasStoreKit && hasRestore && hasSubscriptionManagement && hasRefundPolicy
    }
    
    // MARK: - 권한 사용 설명
    func generatePermissionDescriptions() -> [String: String] {
        return [
            "NSCameraUsageDescription": "레시피 사진을 촬영하고 재료를 스캔하기 위해 카메라 접근이 필요합니다.",
            "NSPhotoLibraryUsageDescription": "레시피 사진을 선택하고 저장하기 위해 사진 라이브러리 접근이 필요합니다.",
            "NSPhotoLibraryAddUsageDescription": "촬영한 레시피 사진을 저장하기 위해 사진 라이브러리 접근이 필요합니다.",
            "NSLocationWhenInUseUsageDescription": "주변 음식점과 마트를 찾기 위해 위치 정보가 필요합니다.",
            "NSUserTrackingUsageDescription": "맞춤형 레시피 추천과 광고 개인화를 위해 사용자 활동을 추적합니다.",
            "NSMicrophoneUsageDescription": "음성으로 레시피를 검색하고 요리 영상을 녹화하기 위해 마이크 접근이 필요합니다.",
            "NSContactsUsageDescription": "레시피를 친구와 공유하기 위해 연락처 접근이 필요합니다.",
            "NSCalendarsUsageDescription": "식단 계획을 캘린더에 추가하기 위해 캘린더 접근이 필요합니다.",
            "NSHealthShareUsageDescription": "칼로리와 영양 정보를 건강 앱과 연동하기 위해 건강 데이터 읽기 권한이 필요합니다.",
            "NSHealthUpdateUsageDescription": "섭취한 음식의 영양 정보를 건강 앱에 기록하기 위해 건강 데이터 쓰기 권한이 필요합니다.",
            "NSFaceIDUsageDescription": "안전한 결제와 개인정보 보호를 위해 Face ID를 사용합니다."
        ]
    }
}

// MARK: - 앱 심사 대응 뷰
struct ComplianceSettingsView: View {
    @State private var showPrivacyPolicy = false
    @State private var showDataDeletion = false
    @State private var showConsentManagement = false
    @State private var showDataPortability = false
    
    var body: some View {
        NavigationStack {
            Form {
                // 개인정보 관리
                Section("개인정보 관리") {
                    NavigationLink(destination: PrivacyPolicyView()) {
                        Label("개인정보처리방침", systemImage: "doc.text")
                    }
                    
                    NavigationLink(destination: ConsentManagementView()) {
                        Label("동의 관리", systemImage: "checkmark.shield")
                    }
                    
                    NavigationLink(destination: DataUsageReportView()) {
                        Label("개인정보 이용내역", systemImage: "chart.bar.doc.horizontal")
                    }
                    
                    NavigationLink(destination: ThirdPartySharingView()) {
                        Label("제3자 제공 현황", systemImage: "person.3")
                    }
                }
                
                // 사용자 권리
                Section("사용자 권리") {
                    Button {
                        showDataPortability = true
                    } label: {
                        Label("내 데이터 다운로드", systemImage: "square.and.arrow.down")
                    }
                    
                    Button {
                        showDataDeletion = true
                    } label: {
                        Label("계정 삭제", systemImage: "trash")
                            .foregroundColor(.red)
                    }
                    
                    NavigationLink(destination: DataCorrectionView()) {
                        Label("정보 수정 요청", systemImage: "pencil")
                    }
                    
                    NavigationLink(destination: ProcessingRestrictionView()) {
                        Label("처리 제한 요청", systemImage: "lock")
                    }
                }
                
                // 보안 설정
                Section("보안 설정") {
                    NavigationLink(destination: SecuritySettingsView()) {
                        Label("2단계 인증", systemImage: "lock.shield")
                    }
                    
                    NavigationLink(destination: BiometricSettingsView()) {
                        Label("생체 인증", systemImage: "faceid")
                    }
                    
                    NavigationLink(destination: EncryptionSettingsView()) {
                        Label("데이터 암호화", systemImage: "lock.doc")
                    }
                }
                
                // 법적 고지
                Section("법적 고지") {
                    NavigationLink(destination: TermsOfServiceView()) {
                        Label("이용약관", systemImage: "doc.plaintext")
                    }
                    
                    NavigationLink(destination: CookiePolicyView()) {
                        Label("쿠키 정책", systemImage: "globe")
                    }
                    
                    NavigationLink(destination: ChildProtectionView()) {
                        Label("아동 보호 정책", systemImage: "figure.child")
                    }
                }
                
                // 문의
                Section("문의") {
                    Link(destination: URL(string: "mailto:privacy@recipebox.app")!) {
                        Label("개인정보보호 책임자", systemImage: "envelope")
                    }
                    
                    Link(destination: URL(string: "tel:02-1234-5678")!) {
                        Label("고객센터", systemImage: "phone")
                    }
                }
            }
            .navigationTitle("개인정보 및 보안")
            .sheet(isPresented: $showDataDeletion) {
                DataDeletionView()
            }
            .sheet(isPresented: $showDataPortability) {
                DataPortabilityView()
            }
        }
    }
}

// MARK: - 개인정보처리방침 뷰
struct PrivacyPolicyView: View {
    let privacyPolicy = """
    개인정보처리방침
    
    시행일: 2024년 1월 1일
    
    주식회사 레시피박스(이하 '회사')는 개인정보보호법, 정보통신망 이용촉진 및 정보보호 등에 관한 법률 등 관련 법령을 준수하며, 이용자의 개인정보를 보호하기 위해 최선을 다하고 있습니다.
    
    1. 개인정보의 수집 및 이용목적
    회사는 다음의 목적을 위하여 개인정보를 처리합니다.
    
    가. 회원 관리
    - 회원제 서비스 이용에 따른 본인확인
    - 개인 식별
    - 불량회원의 부정 이용 방지
    - 가입 의사 확인
    - 연령 확인
    - 만 14세 미만 아동의 개인정보 처리 시 법정대리인의 동의 여부 확인
    
    나. 서비스 제공
    - 레시피 추천 및 검색 서비스
    - 맞춤형 콘텐츠 제공
    - 식단 관리 서비스
    - 결제 및 정산
    
    2. 수집하는 개인정보 항목
    
    가. 필수 항목
    - 이메일 주소
    - 비밀번호
    - 닉네임
    - 생년월일
    
    나. 선택 항목
    - 프로필 사진
    - 전화번호
    - 주소
    - 알레르기 정보
    - 선호 음식
    
    다. 자동 수집 항목
    - IP 주소
    - 쿠키
    - 방문 일시
    - 서비스 이용 기록
    - 기기 정보
    
    3. 개인정보의 보유 및 이용기간
    
    회사는 법령에 따른 개인정보 보유·이용기간 또는 정보주체로부터 개인정보를 수집 시에 동의받은 개인정보 보유·이용기간 내에서 개인정보를 처리·보유합니다.
    
    가. 회원 정보: 회원 탈퇴 시까지
    나. 거래 기록: 5년 (전자상거래법)
    다. 접속 기록: 3년 (통신비밀보호법)
    라. 소비자 불만 또는 분쟁 처리 기록: 3년 (전자상거래법)
    
    4. 개인정보의 제3자 제공
    
    회사는 원칙적으로 이용자의 개인정보를 제3자에게 제공하지 않습니다. 다만, 다음의 경우에는 예외로 합니다.
    
    가. 이용자가 사전에 동의한 경우
    나. 법령의 규정에 의거하거나, 수사 목적으로 법령에 정해진 절차와 방법에 따라 수사기관의 요구가 있는 경우
    
    5. 개인정보처리 위탁
    
    회사는 서비스 이행을 위해 다음과 같이 개인정보 처리업무를 위탁하고 있습니다.
    
    - AWS: 데이터 보관
    - 토스페이먼츠: 결제 처리
    - SendGrid: 이메일 발송
    
    6. 정보주체의 권리·의무 및 행사방법
    
    이용자는 개인정보주체로서 다음과 같은 권리를 행사할 수 있습니다.
    
    가. 개인정보 열람 요구
    나. 오류 등이 있을 경우 정정 요구
    다. 삭제 요구
    라. 처리 정지 요구
    
    7. 개인정보 보호책임자
    
    성명: 김철수
    직책: 개인정보보호 책임자
    이메일: privacy@recipebox.app
    전화: 02-1234-5678
    
    8. 개인정보 처리방침 변경
    
    이 개인정보처리방침은 시행일로부터 적용되며, 법령 및 방침에 따른 변경내용의 추가, 삭제 및 정정이 있는 경우에는 변경사항의 시행 7일 전부터 공지사항을 통하여 고지할 것입니다.
    """
    
    var body: some View {
        ScrollView {
            Text(privacyPolicy)
                .padding()
                .font(.system(size: 14))
        }
        .navigationTitle("개인정보처리방침")
        .navigationBarTitleDisplayMode(.inline)
    }
}

// MARK: - 동의 관리 뷰
struct ConsentManagementView: View {
    @State private var consents = ConsentSettings()
    
    struct ConsentSettings {
        var termsOfService = true
        var privacyPolicy = true
        var dataCollection = true
        var marketingEmail = false
        var marketingSMS = false
        var marketingPush = false
        var thirdPartySharing = false
        var locationTracking = false
        var advertisingId = false
    }
    
    var body: some View {
        Form {
            Section("필수 동의 항목") {
                ConsentRow(
                    title: "서비스 이용약관",
                    isOn: .constant(true),
                    isRequired: true
                )
                
                ConsentRow(
                    title: "개인정보 수집 및 이용",
                    isOn: .constant(true),
                    isRequired: true
                )
            }
            
            Section("선택 동의 항목") {
                ConsentRow(
                    title: "이메일 마케팅 수신",
                    isOn: $consents.marketingEmail,
                    isRequired: false
                )
                
                ConsentRow(
                    title: "SMS 마케팅 수신",
                    isOn: $consents.marketingSMS,
                    isRequired: false
                )
                
                ConsentRow(
                    title: "푸시 알림 수신",
                    isOn: $consents.marketingPush,
                    isRequired: false
                )
                
                ConsentRow(
                    title: "제3자 정보 제공",
                    isOn: $consents.thirdPartySharing,
                    isRequired: false
                )
                
                ConsentRow(
                    title: "위치 정보 수집",
                    isOn: $consents.locationTracking,
                    isRequired: false
                )
                
                ConsentRow(
                    title: "광고 ID 수집",
                    isOn: $consents.advertisingId,
                    isRequired: false
                )
            }
            
            Section {
                Button {
                    saveConsents()
                } label: {
                    Text("저장")
                        .frame(maxWidth: .infinity)
                        .foregroundColor(.white)
                        .padding()
                        .background(Color.blue)
                        .cornerRadius(8)
                }
            }
        }
        .navigationTitle("동의 관리")
    }
    
    func saveConsents() {
        // Save consent settings
        print("Consents saved")
    }
}

struct ConsentRow: View {
    let title: String
    @Binding var isOn: Bool
    let isRequired: Bool
    
    var body: some View {
        Toggle(isOn: $isOn) {
            HStack {
                Text(title)
                if isRequired {
                    Text("(필수)")
                        .font(.caption)
                        .foregroundColor(.red)
                }
            }
        }
        .disabled(isRequired)
    }
}

// MARK: - 데이터 삭제 뷰
struct DataDeletionView: View {
    @State private var confirmationText = ""
    @State private var selectedReason = ""
    @State private var additionalComments = ""
    @State private var agreedToConsequences = false
    @Environment(\.dismiss) private var dismiss
    
    let deletionReasons = [
        "더 이상 서비스를 이용하지 않음",
        "다른 서비스로 이전",
        "개인정보 보호 우려",
        "서비스 불만족",
        "기타"
    ]
    
    var body: some View {
        NavigationStack {
            ScrollView {
                VStack(alignment: .leading, spacing: 20) {
                    // 경고 메시지
                    VStack(alignment: .leading, spacing: 12) {
                        Label("주의사항", systemImage: "exclamationmark.triangle.fill")
                            .font(.headline)
                            .foregroundColor(.red)
                        
                        Text("""
                        계정을 삭제하면 다음 데이터가 영구적으로 삭제됩니다:
                        • 모든 레시피와 사진
                        • 구매 내역 및 결제 정보
                        • 팔로워 및 팔로잉 정보
                        • 리뷰 및 평점
                        • 식단 계획 및 건강 데이터
                        
                        일부 데이터는 법적 의무에 따라 일정 기간 보관될 수 있습니다.
                        """)
                        .font(.caption)
                    }
                    .padding()
                    .background(Color.red.opacity(0.1))
                    .cornerRadius(12)
                    
                    // 삭제 사유
                    VStack(alignment: .leading, spacing: 8) {
                        Text("삭제 사유")
                            .font(.headline)
                        
                        ForEach(deletionReasons, id: \.self) { reason in
                            Button {
                                selectedReason = reason
                            } label: {
                                HStack {
                                    Image(systemName: selectedReason == reason ? "checkmark.circle.fill" : "circle")
                                    Text(reason)
                                    Spacer()
                                }
                                .foregroundColor(.primary)
                            }
                            .padding(.vertical, 4)
                        }
                    }
                    
                    // 추가 의견
                    VStack(alignment: .leading, spacing: 8) {
                        Text("추가 의견 (선택)")
                            .font(.headline)
                        
                        TextEditor(text: $additionalComments)
                            .frame(height: 100)
                            .padding(8)
                            .background(Color(.systemGray6))
                            .cornerRadius(8)
                    }
                    
                    // 확인 입력
                    VStack(alignment: .leading, spacing: 8) {
                        Text("확인")
                            .font(.headline)
                        
                        Text("계정을 삭제하려면 '삭제합니다'를 입력하세요")
                            .font(.caption)
                            .foregroundColor(.secondary)
                        
                        TextField("삭제합니다", text: $confirmationText)
                            .textFieldStyle(RoundedBorderTextFieldStyle())
                    }
                    
                    // 동의 체크박스
                    Toggle(isOn: $agreedToConsequences) {
                        Text("위 내용을 모두 확인했으며, 데이터 삭제에 따른 결과를 이해했습니다")
                            .font(.caption)
                    }
                    
                    // 버튼
                    HStack {
                        Button("취소") {
                            dismiss()
                        }
                        .frame(maxWidth: .infinity)
                        .padding()
                        .background(Color(.systemGray5))
                        .foregroundColor(.primary)
                        .cornerRadius(8)
                        
                        Button("계정 삭제") {
                            deleteAccount()
                        }
                        .frame(maxWidth: .infinity)
                        .padding()
                        .background(canDelete ? Color.red : Color(.systemGray5))
                        .foregroundColor(.white)
                        .cornerRadius(8)
                        .disabled(!canDelete)
                    }
                }
                .padding()
            }
            .navigationTitle("계정 삭제")
            .navigationBarTitleDisplayMode(.inline)
        }
    }
    
    var canDelete: Bool {
        confirmationText == "삭제합니다" &&
        !selectedReason.isEmpty &&
        agreedToConsequences
    }
    
    func deleteAccount() {
        // Process account deletion
        print("Account deletion requested")
    }
}

// MARK: - Models
struct PersonalData: Codable {
    let userId: String
    let name: String
    let email: String
    let phone: String?
    let birthDate: Date?
    let address: String?
}

struct DataAccessLog: Codable {
    let timestamp: Date
    let action: String
    let dataType: String
    let userId: String
    let purpose: String
    let ipAddress: String
    let deviceId: String
}

struct DeletionCertificate: Codable {
    let userId: String
    let deletionDate: Date
    let reason: String
    let deletedItems: [String]
    let retainedItems: [String]
}

struct ThirdPartySharingRecord: Codable {
    let userId: String
    let recipient: String
    let dataTypes: [String]
    let purpose: String
    let sharingDate: Date
    let retentionPeriod: Int
    let consentDate: Date?
}

struct PrivacyUsageReport: Codable {
    let userId: String
    let reportDate: Date
    let collectedData: [String]
    let usagePurposes: [String]
    let thirdPartySharing: [ThirdPartySharingRecord]
    let retentionPeriod: [String: Int]
    let userRights: [String]
    let contactInfo: ContactInfo
}

struct ContactInfo: Codable {
    let responsiblePerson: String
    let email: String
    let phone: String
    let address: String
}

struct ValidationResult {
    let category: String
    let isValid: Bool
    let issues: [String]
}

struct DemoAccount {
    let username: String
    let password: String
    let verificationCode: String
}

// MARK: - Placeholder Views
struct DataUsageReportView: View {
    var body: some View {
        Text("개인정보 이용내역")
    }
}

struct ThirdPartySharingView: View {
    var body: some View {
        Text("제3자 제공 현황")
    }
}

struct DataCorrectionView: View {
    var body: some View {
        Text("정보 수정 요청")
    }
}

struct ProcessingRestrictionView: View {
    var body: some View {
        Text("처리 제한 요청")
    }
}

struct SecuritySettingsView: View {
    var body: some View {
        Text("2단계 인증")
    }
}

struct BiometricSettingsView: View {
    var body: some View {
        Text("생체 인증")
    }
}

struct EncryptionSettingsView: View {
    var body: some View {
        Text("데이터 암호화")
    }
}

struct TermsOfServiceView: View {
    var body: some View {
        Text("이용약관")
    }
}

struct CookiePolicyView: View {
    var body: some View {
        Text("쿠키 정책")
    }
}

struct ChildProtectionView: View {
    var body: some View {
        Text("아동 보호 정책")
    }
}

struct DataPortabilityView: View {
    var body: some View {
        Text("데이터 다운로드")
    }
}

// MARK: - Errors
enum PrivacyError: Error {
    case keychainSaveFailed
    case noConsentForSharing
    case dataEncryptionFailed
    case invalidUserId
}

// MARK: - Helper Functions
extension PrivacyManager {
    func deleteFromKeychain(key: String) {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key
        ]
        SecItemDelete(query as CFDictionary)
    }
    
    func markForDeletion(key: String, deleteAfter: Date) {
        // Implementation
    }
    
    func saveDeletionCertificate(_ certificate: DeletionCertificate) {
        // Implementation
    }
    
    func saveAccessLog(_ log: DataAccessLog) {
        // Implementation
    }
    
    func saveSharingRecord(_ record: ThirdPartySharingRecord) {
        // Implementation
    }
    
    func hasConsentForThirdPartySharing(userId: String) -> Bool {
        // Implementation
        return true
    }
    
    func getConsentDate(userId: String, type: String) -> Date? {
        // Implementation
        return Date()
    }
    
    func notifyUserOfDataSharing(userId: String, record: ThirdPartySharingRecord) {
        // Implementation
    }
    
    func getIPAddress() -> String {
        // Implementation
        return "127.0.0.1"
    }
    
    func getDeviceId() -> String {
        // Implementation
        return UIDevice.current.identifierForVendor?.uuidString ?? "unknown"
    }
    
    func getCollectedDataTypes(userId: String) -> [String] {
        return ["이름", "이메일", "전화번호", "주소"]
    }
    
    func getUsagePurposes(userId: String) -> [String] {
        return ["서비스 제공", "회원 관리", "마케팅"]
    }
    
    func getThirdPartySharingRecords(userId: String) -> [ThirdPartySharingRecord] {
        return []
    }
    
    func getRetentionPeriods() -> [String: Int] {
        return [
            "회원정보": 0,
            "거래기록": 5,
            "접속기록": 3
        ]
    }
    
    func getUserRights() -> [String] {
        return [
            "개인정보 열람권",
            "정정·삭제권",
            "처리정지권",
            "이동권"
        ]
    }
    
    func getPrivacyContactInfo() -> ContactInfo {
        return ContactInfo(
            responsiblePerson: "김철수",
            email: "privacy@recipebox.app",
            phone: "02-1234-5678",
            address: "서울특별시 강남구"
        )
    }
}

extension AppStoreComplianceManager {
    func checkStoreKitImplementation() -> Bool {
        // Check StoreKit implementation
        return true
    }
    
    func checkRestorePurchases() -> Bool {
        // Check restore functionality
        return true
    }
    
    func checkSubscriptionManagement() -> Bool {
        // Check subscription management
        return true
    }
    
    func checkRefundPolicy() -> Bool {
        // Check refund policy
        return true
    }
    
    func validateDescription() -> ValidationResult {
        return ValidationResult(category: "설명", isValid: true, issues: [])
    }
    
    func validateKeywords() -> ValidationResult {
        return ValidationResult(category: "키워드", isValid: true, issues: [])
    }
    
    func validateScreenshots() -> ValidationResult {
        return ValidationResult(category: "스크린샷", isValid: true, issues: [])
    }
    
    func validateAgeRating() -> ValidationResult {
        return ValidationResult(category: "연령 등급", isValid: true, issues: [])
    }
}
```