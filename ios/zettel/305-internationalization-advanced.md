# 고급 국제화: 전 세계를 위한 앱 설계

## Core Insight
진정한 국제화는 단순한 텍스트 번역을 넘어서, 문화적 맥락과 지역적 관습을 깊이 이해하여 각 지역 사용자에게 네이티브한 경험을 제공하는 것이다.

국제화(i18n)는 앱을 전 세계로 확장하는 열쇠다. 하지만 단순히 문자열을 번역하는 것으로는 충분하지 않다. 아랍어의 오른쪽-왼쪽 읽기, 일본어의 세로쓰기, 독일어의 긴 복합어, 중국어의 번체/간체 구분 등 각 언어와 문화의 고유한 특성을 이해해야 한다.

**고급 지역화 아키텍처:**

**1. 다국어 문자열 관리 시스템:**
```swift
import Foundation

// 고급 지역화 매니저
class AdvancedLocalizationManager {
    private static let shared = AdvancedLocalizationManager()
    private var currentLocale: Locale = .current
    private var fallbackLocale: Locale = Locale(identifier: "en")
    
    // 복수형 처리를 포함한 지역화
    func localizedString(key: String, count: Int? = nil, arguments: CVarArg...) -> String {
        if let count = count {
            return localizedPluralString(key: key, count: count, arguments: arguments)
        }
        
        let formatString = NSLocalizedString(key, comment: "")
        
        if arguments.isEmpty {
            return formatString
        }
        
        return String(format: formatString, arguments: arguments)
    }
    
    private func localizedPluralString(key: String, count: Int, arguments: [CVarArg]) -> String {
        // ICU 메시지 포맷을 사용한 복수형 처리
        let pluralRule = getPluralRule(for: count, locale: currentLocale)
        let pluralKey = "\(key).\(pluralRule)"
        
        let formatString = NSLocalizedString(pluralKey, comment: "")
        let allArguments = [count] + arguments
        
        return String(format: formatString, arguments: allArguments)
    }
    
    private func getPluralRule(for count: Int, locale: Locale) -> String {
        let formatter = NumberFormatter()
        formatter.locale = locale
        
        // 언어별 복수형 규칙
        switch locale.languageCode {
        case "ko", "ja", "zh": // 단수형만 존재
            return "other"
        case "ru", "uk": // 복잡한 복수형 규칙
            return russianPluralRule(count: count)
        case "ar": // 아랍어의 특수한 복수형
            return arabicPluralRule(count: count)
        default: // 영어식 단수/복수
            return count == 1 ? "one" : "other"
        }
    }
    
    private func russianPluralRule(count: Int) -> String {
        let lastDigit = count % 10
        let lastTwoDigits = count % 100
        
        if lastTwoDigits >= 11 && lastTwoDigits <= 19 {
            return "many"
        } else if lastDigit == 1 {
            return "one"
        } else if lastDigit >= 2 && lastDigit <= 4 {
            return "few"
        } else {
            return "many"
        }
    }
    
    private func arabicPluralRule(count: Int) -> String {
        switch count {
        case 0: return "zero"
        case 1: return "one"
        case 2: return "two"
        case 3...10: return "few"
        case let n where n % 100 >= 11 && n % 100 <= 99: return "many"
        default: return "other"
        }
    }
}

// 사용 예시
extension String {
    func localized(count: Int? = nil, _ arguments: CVarArg...) -> String {
        return AdvancedLocalizationManager.shared.localizedString(key: self, count: count, arguments: arguments)
    }
}
```

**2. RTL(Right-to-Left) 언어 지원:**
```swift
import UIKit

class RTLLayoutManager {
    static func setupRTLSupport() {
        // 자동 RTL 지원 활성화
        UIView.appearance().semanticContentAttribute = .unspecified
        UIApplication.shared.userInterfaceLayoutDirection = .unspecified
    }
    
    static func isRTLLanguage() -> Bool {
        return Locale.current.characterDirection == .rightToLeft
    }
    
    static func adaptLayoutForRTL(_ view: UIView) {
        if isRTLLanguage() {
            // 이미지 뒤집기
            adaptImagesForRTL(in: view)
            
            // 텍스트 정렬 조정
            adaptTextAlignmentForRTL(in: view)
            
            // 네비게이션 요소 조정
            adaptNavigationForRTL(in: view)
        }
    }
    
    private static func adaptImagesForRTL(in view: UIView) {
        view.subviews.forEach { subview in
            if let imageView = subview as? UIImageView {
                // 방향성이 있는 이미지는 뒤집기
                if shouldFlipImage(imageView.image) {
                    imageView.image = imageView.image?.imageFlippedForRightToLeftLayoutDirection()
                }
            }
            adaptImagesForRTL(in: subview)
        }
    }
    
    private static func shouldFlipImage(_ image: UIImage?) -> Bool {
        // 화살표, 방향 지시자 등은 뒤집어야 함
        // 텍스트, 얼굴, 로고 등은 뒤집으면 안 됨
        return image?.accessibilityIdentifier?.contains("directional") ?? false
    }
    
    private static func adaptTextAlignmentForRTL(in view: UIView) {
        view.subviews.forEach { subview in
            if let label = subview as? UILabel {
                // 자연스러운 텍스트 정렬
                label.textAlignment = .natural
            } else if let textField = subview as? UITextField {
                textField.textAlignment = .natural
            }
            adaptTextAlignmentForRTL(in: subview)
        }
    }
}
```

**3. 문화별 날짜/시간/숫자 형식:**
```swift
class CulturalFormatManager {
    private let locale: Locale
    
    init(locale: Locale = .current) {
        self.locale = locale
    }
    
    // 문화권별 날짜 표시
    func formatDate(_ date: Date, style: DateStyle = .full) -> String {
        let formatter = DateFormatter()
        formatter.locale = locale
        
        switch style {
        case .full:
            formatter.dateStyle = .full
            formatter.timeStyle = .short
        case .relative:
            return formatRelativeDate(date)
        case .cultural:
            return formatCulturalDate(date)
        }
        
        return formatter.string(from: date)
    }
    
    private func formatRelativeDate(_ date: Date) -> String {
        let formatter = RelativeDateTimeFormatter()
        formatter.locale = locale
        formatter.unitsStyle = .full
        
        return formatter.localizedString(for: date, relativeTo: Date())
    }
    
    private func formatCulturalDate(_ date: Date) -> String {
        switch locale.regionCode {
        case "US":
            // MM/dd/yyyy 형식
            let formatter = DateFormatter()
            formatter.dateFormat = "MM/dd/yyyy"
            return formatter.string(from: date)
            
        case "KR", "JP":
            // yyyy년 MM월 dd일 형식
            let formatter = DateFormatter()
            formatter.locale = locale
            formatter.dateFormat = "yyyy년 MM월 dd일"
            return formatter.string(from: date)
            
        case "DE", "FR":
            // dd.MM.yyyy 형식
            let formatter = DateFormatter()
            formatter.dateFormat = "dd.MM.yyyy"
            return formatter.string(from: date)
            
        default:
            return formatDate(date, style: .full)
        }
    }
    
    enum DateStyle {
        case full, relative, cultural
    }
    
    // 통화 형식
    func formatCurrency(_ amount: Decimal) -> String {
        let formatter = NumberFormatter()
        formatter.numberStyle = .currency
        formatter.locale = locale
        
        // 지역별 통화 처리
        if let currencyCode = getCurrencyCode() {
            formatter.currencyCode = currencyCode
        }
        
        return formatter.string(from: amount as NSDecimalNumber) ?? "\(amount)"
    }
    
    private func getCurrencyCode() -> String? {
        switch locale.regionCode {
        case "US": return "USD"
        case "KR": return "KRW"
        case "JP": return "JPY"
        case "GB": return "GBP"
        case "DE", "FR", "IT", "ES": return "EUR"
        default: return locale.currencyCode
        }
    }
    
    // 주소 형식
    func formatAddress(_ address: Address) -> String {
        switch locale.regionCode {
        case "KR":
            return "\(address.country) \(address.state) \(address.city) \(address.street) \(address.number)"
        case "JP":
            return "〒\(address.postalCode) \(address.state)\(address.city)\(address.street)\(address.number)"
        case "US":
            return "\(address.number) \(address.street)\n\(address.city), \(address.state) \(address.postalCode)"
        default:
            return "\(address.street) \(address.number)\n\(address.postalCode) \(address.city)"
        }
    }
}

struct Address {
    let country: String
    let state: String
    let city: String
    let street: String
    let number: String
    let postalCode: String
}
```

**4. 폰트와 타이포그래피 지역화:**
```swift
class InternationalTypography {
    static func setupFonts() {
        // 언어별 최적 폰트 설정
        let currentLanguage = Locale.current.languageCode
        
        switch currentLanguage {
        case "ko":
            setupKoreanFonts()
        case "ja":
            setupJapaneseFonts()
        case "zh":
            setupChineseFonts()
        case "ar":
            setupArabicFonts()
        case "th":
            setupThaiFonts()
        case "hi":
            setupHindiFonts()
        default:
            setupLatinFonts()
        }
    }
    
    private static func setupKoreanFonts() {
        // 한글 전용 폰트 설정
        let descriptor = UIFontDescriptor.preferredFontDescriptor(withTextStyle: .body)
        let koreanDescriptor = descriptor.addingAttributes([
            UIFontDescriptor.AttributeName.family: "Apple SD Gothic Neo"
        ])
        
        UIFont.systemFont(ofSize: 16) // 한글은 라틴 문자보다 약간 큰 크기 필요
    }
    
    private static func setupArabicFonts() {
        // 아랍어 폰트는 특별한 처리 필요
        let arabicDescriptor = UIFontDescriptor.preferredFontDescriptor(withTextStyle: .body)
            .addingAttributes([
                UIFontDescriptor.AttributeName.family: "Geeza Pro"
            ])
        
        // 아랍어는 RTL이므로 문단 스타일도 조정
        let paragraphStyle = NSMutableParagraphStyle()
        paragraphStyle.alignment = .natural
        paragraphStyle.baseWritingDirection = .rightToLeft
    }
    
    static func calculateOptimalLineHeight(for text: String, font: UIFont) -> CGFloat {
        let language = detectLanguage(text)
        
        switch language {
        case .korean, .japanese, .chinese:
            // CJK 문자는 더 넓은 줄 간격 필요
            return font.lineHeight * 1.4
        case .arabic, .hebrew:
            // 셈족 언어는 위아래 확장자가 많음
            return font.lineHeight * 1.3
        case .thai, .khmer:
            // 동남아시아 언어는 복잡한 문자 조합
            return font.lineHeight * 1.5
        default:
            return font.lineHeight * 1.2
        }
    }
    
    enum Language {
        case korean, japanese, chinese, arabic, hebrew, thai, khmer, latin
    }
    
    private static func detectLanguage(_ text: String) -> Language {
        let tagger = NSLinguisticTagger(tagSchemes: [.language], options: 0)
        tagger.string = text
        
        guard let language = tagger.tag(at: 0, unit: .word, scheme: .language, options: []).0?.rawValue else {
            return .latin
        }
        
        switch language {
        case "ko": return .korean
        case "ja": return .japanese
        case "zh": return .chinese
        case "ar": return .arabic
        case "he": return .hebrew
        case "th": return .thai
        case "km": return .khmer
        default: return .latin
        }
    }
}
```

**5. 입력 방법과 키보드 처리:**
```swift
class InternationalInputManager {
    static func setupInputMethods() {
        NotificationCenter.default.addObserver(
            forName: UITextInputMode.currentInputModeDidChangeNotification,
            object: nil,
            queue: .main
        ) { _ in
            handleInputModeChange()
        }
    }
    
    private static func handleInputModeChange() {
        guard let inputMode = UITextInputMode.activeInputModes.first else { return }
        
        let primaryLanguage = inputMode.primaryLanguage ?? "en"
        
        switch primaryLanguage {
        case "ko-KR":
            handleKoreanInput()
        case "ja-JP":
            handleJapaneseInput()
        case "zh-Hans", "zh-Hant":
            handleChineseInput()
        case "ar":
            handleArabicInput()
        default:
            handleDefaultInput()
        }
    }
    
    private static func handleKoreanInput() {
        // 한글 입력 시 특별 처리
        // - 조합 중인 문자 처리
        // - 자동완성 조정
        print("Korean input mode activated")
    }
    
    private static func handleJapaneseInput() {
        // 일본어 입력 (히라가나/가타카나/간지)
        // - IME 상태 관리
        // - 변환 후보 표시
        print("Japanese input mode activated")
    }
    
    static func adaptKeyboardForLanguage(_ textField: UITextField) {
        guard let language = Locale.current.languageCode else { return }
        
        switch language {
        case "ko":
            textField.keyboardType = .default
            textField.autocorrectionType = .yes
            textField.spellCheckingType = .no // 한글에는 스펠체크 불필요
            
        case "ja":
            textField.keyboardType = .default
            textField.autocorrectionType = .no
            textField.spellCheckingType = .no
            
        case "zh":
            textField.keyboardType = .default
            textField.autocorrectionType = .no
            textField.spellCheckingType = .no
            
        case "ar":
            textField.keyboardType = .default
            textField.autocorrectionType = .yes
            textField.textAlignment = .right
            
        default:
            textField.keyboardType = .default
            textField.autocorrectionType = .yes
            textField.spellCheckingType = .yes
        }
    }
}
```

**6. 지역별 컨텐츠 필터링:**
```swift
class RegionalContentManager {
    static func filterContentForRegion(_ content: [ContentItem]) -> [ContentItem] {
        let region = Locale.current.regionCode ?? "US"
        
        return content.filter { item in
            // 법적 제한 확인
            if isLegallyRestricted(item, in: region) {
                return false
            }
            
            // 문화적 적절성 확인
            if !isCulturallyAppropriate(item, for: region) {
                return false
            }
            
            // 연령 제한 확인
            if !meetsAgeRequirements(item, in: region) {
                return false
            }
            
            return true
        }
    }
    
    private static func isLegallyRestricted(_ item: ContentItem, in region: String) -> Bool {
        switch region {
        case "CN":
            // 중국의 컨텐츠 제한
            return item.tags.contains("political") || item.tags.contains("gambling")
        case "DE":
            // 독일의 나치 관련 제한
            return item.tags.contains("nazi-symbols")
        case "AU":
            // 호주의 폭력적 컨텐츠 제한
            return item.violenceLevel > 3
        default:
            return false
        }
    }
    
    private static func isCulturallyAppropriate(_ item: ContentItem, for region: String) -> Bool {
        switch region {
        case "SA", "AE": // 사우디아라비아, UAE
            return !item.tags.contains("alcohol") && !item.tags.contains("nudity")
        case "IN":
            return !item.tags.contains("beef")
        case "IL":
            return !item.tags.contains("pork")
        default:
            return true
        }
    }
    
    private static func meetsAgeRequirements(_ item: ContentItem, in region: String) -> Bool {
        let localAgeLimit = getLocalAgeLimit(for: region)
        return item.minimumAge <= localAgeLimit
    }
    
    private static func getLocalAgeLimit(for region: String) -> Int {
        switch region {
        case "US": return 17
        case "KR": return 19
        case "DE": return 16
        default: return 18
        }
    }
}

struct ContentItem {
    let id: String
    let title: String
    let tags: [String]
    let violenceLevel: Int
    let minimumAge: Int
}
```

**7. 다국어 검색과 정렬:**
```swift
class InternationalSearchManager {
    static func search(_ query: String, in items: [SearchableItem]) -> [SearchableItem] {
        let locale = Locale.current
        
        // 언어별 특수 검색 로직
        switch locale.languageCode {
        case "ko":
            return koreanSearch(query, in: items)
        case "ja":
            return japaneseSearch(query, in: items)
        case "zh":
            return chineseSearch(query, in: items)
        case "ar":
            return arabicSearch(query, in: items)
        default:
            return latinSearch(query, in: items)
        }
    }
    
    private static func koreanSearch(_ query: String, in items: [SearchableItem]) -> [SearchableItem] {
        // 한글 검색: 초성 검색, 자모 분리 검색 지원
        let normalizedQuery = query.precomposedStringWithCanonicalMapping
        
        return items.filter { item in
            let normalizedTitle = item.title.precomposedStringWithCanonicalMapping
            
            // 일반 검색
            if normalizedTitle.localizedCaseInsensitiveContains(normalizedQuery) {
                return true
            }
            
            // 초성 검색 (예: "ㄱㅅ" -> "구글")
            if matchesKoreanInitials(query: query, text: normalizedTitle) {
                return true
            }
            
            return false
        }
    }
    
    private static func matchesKoreanInitials(query: String, text: String) -> Bool {
        let koreanInitials = "ㄱㄴㄷㄹㅁㅂㅅㅇㅈㅊㅋㅌㅍㅎ"
        
        // 초성만 포함된 검색어인지 확인
        let isInitialOnly = query.allSatisfy { koreanInitials.contains($0) }
        
        if isInitialOnly {
            let textInitials = extractKoreanInitials(from: text)
            return textInitials.hasPrefix(query)
        }
        
        return false
    }
    
    private static func extractKoreanInitials(from text: String) -> String {
        // 한글 유니코드에서 초성 추출
        let initialConsonants = "ㄱㄲㄴㄷㄸㄹㅁㅂㅃㅅㅆㅇㅈㅉㅊㅋㅌㅍㅎ"
        
        return String(text.compactMap { char in
            let scalar = char.unicodeScalars.first?.value ?? 0
            
            if scalar >= 0xAC00 && scalar <= 0xD7A3 { // 한글 완성형 범위
                let index = (scalar - 0xAC00) / 588
                return initialConsonants[initialConsonants.index(initialConsonants.startIndex, offsetBy: Int(index))]
            }
            
            return nil
        })
    }
    
    static func sort(_ items: [SearchableItem], by criteria: SortCriteria) -> [SearchableItem] {
        let locale = Locale.current
        
        switch criteria {
        case .alphabetical:
            return items.sorted { item1, item2 in
                item1.title.localizedCompare(item2.title, locale: locale) == .orderedAscending
            }
        case .cultural:
            return culturalSort(items, locale: locale)
        }
    }
    
    private static func culturalSort(_ items: [SearchableItem], locale: Locale) -> [SearchableItem] {
        switch locale.languageCode {
        case "ko":
            // 한글: 가나다 순
            return items.sorted { $0.title.localizedCompare($1.title, locale: locale) == .orderedAscending }
        case "ja":
            // 일본어: 히라가나 > 가타카나 > 한자 순
            return japaneseSort(items)
        case "zh":
            // 중국어: 병음 순서 또는 획수 순
            return chineseSort(items)
        default:
            return items.sorted { $0.title.localizedCompare($1.title, locale: locale) == .orderedAscending }
        }
    }
    
    enum SortCriteria {
        case alphabetical, cultural
    }
}

struct SearchableItem {
    let title: String
    let content: String
}
```

국제화는 기술적 구현을 넘어서 문화적 감수성의 문제다. 각 지역의 사용자가 마치 그들을 위해 특별히 만들어진 앱처럼 느낄 수 있게 하는 것이 목표다. 이는 단순한 번역이 아닌, 각 문화권의 사고방식과 사용 패턴을 이해하는 과정이다.

## Connections
→ [[306-memory-management-edge-cases]]
→ [[307-offline-experiences-network-failures]]
← [[304-carplay-external-display]]
← [[067-accessibility-as-default]]

---
Level: L3
Date: 2025-08-16
Tags: #internationalization #localization #rtl #cultural-adaptation #typography #input-methods