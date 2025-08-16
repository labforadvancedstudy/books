# 한국 사용자 UX 패턴 완벽 가이드

## 한국형 UI/UX 디자인 시스템

```swift
import SwiftUI
import UIKit

// MARK: - Korean Design System
struct KoreanDesignSystem {
    
    // MARK: - Typography
    struct Typography {
        // 한글 최적화 폰트
        static let fonts = KoreanFonts()
        
        struct KoreanFonts {
            // 제목용 폰트
            let largeTitle = Font.custom("AppleSDGothicNeo-Bold", size: 34)
            let title1 = Font.custom("AppleSDGothicNeo-Bold", size: 28)
            let title2 = Font.custom("AppleSDGothicNeo-Semibold", size: 22)
            let title3 = Font.custom("AppleSDGothicNeo-Medium", size: 20)
            
            // 본문용 폰트
            let headline = Font.custom("AppleSDGothicNeo-Semibold", size: 17)
            let body = Font.custom("AppleSDGothicNeo-Regular", size: 17)
            let callout = Font.custom("AppleSDGothicNeo-Regular", size: 16)
            let subheadline = Font.custom("AppleSDGothicNeo-Regular", size: 15)
            let footnote = Font.custom("AppleSDGothicNeo-Regular", size: 13)
            let caption1 = Font.custom("AppleSDGothicNeo-Regular", size: 12)
            let caption2 = Font.custom("AppleSDGothicNeo-Regular", size: 11)
            
            // 강조용 스타일
            let emphasis = Font.custom("AppleSDGothicNeo-Bold", size: 17)
            let highlight = Font.custom("AppleSDGothicNeo-Semibold", size: 16)
            
            // 가격 표시용
            let priceMain = Font.custom("AppleSDGothicNeo-Bold", size: 24)
            let priceSub = Font.custom("AppleSDGothicNeo-Regular", size: 14)
            let priceOriginal = Font.custom("AppleSDGothicNeo-Light", size: 14)
                .strikethrough()
        }
        
        // 줄 간격 설정
        static func lineSpacing(for font: Font) -> CGFloat {
            // 한글은 영문보다 더 넓은 줄 간격 필요
            return 4.0
        }
        
        // 자간 설정
        static func letterSpacing(for font: Font) -> CGFloat {
            // 한글 가독성을 위한 자간
            return -0.2
        }
    }
    
    // MARK: - Colors
    struct Colors {
        // 메인 색상 (한국 앱 선호 색상)
        static let primary = Color(hex: "5B5FCF")      // 카카오뱅크 스타일 보라
        static let secondary = Color(hex: "FEE500")    // 카카오 노란색
        static let accent = Color(hex: "03C75A")       // 네이버 초록색
        
        // 배경 색상
        static let background = Color(hex: "F8F9FA")
        static let surface = Color.white
        static let elevated = Color(hex: "FFFFFF")
        
        // 텍스트 색상
        static let textPrimary = Color(hex: "191F28")
        static let textSecondary = Color(hex: "8B95A1")
        static let textTertiary = Color(hex: "B0B8C1")
        
        // 상태 색상
        static let success = Color(hex: "07B53B")      // 쿠팡 구매 버튼
        static let warning = Color(hex: "FFB800")
        static let error = Color(hex: "FF5A5F")        // 배민 빨간색
        static let info = Color(hex: "4A90E2")
        
        // 그라데이션
        static let primaryGradient = LinearGradient(
            colors: [Color(hex: "667EEA"), Color(hex: "764BA2")],
            startPoint: .topLeading,
            endPoint: .bottomTrailing
        )
        
        static let premiumGradient = LinearGradient(
            colors: [Color(hex: "F093FB"), Color(hex: "F5576C")],
            startPoint: .leading,
            endPoint: .trailing
        )
    }
    
    // MARK: - Spacing
    struct Spacing {
        static let xxs: CGFloat = 4
        static let xs: CGFloat = 8
        static let sm: CGFloat = 12
        static let md: CGFloat = 16
        static let lg: CGFloat = 20
        static let xl: CGFloat = 24
        static let xxl: CGFloat = 32
        static let xxxl: CGFloat = 40
    }
    
    // MARK: - Corner Radius
    struct CornerRadius {
        static let sm: CGFloat = 8
        static let md: CGFloat = 12
        static let lg: CGFloat = 16
        static let xl: CGFloat = 20
        static let full: CGFloat = 9999
    }
    
    // MARK: - Shadow
    struct Shadow {
        static let sm = ShadowStyle(
            color: Color.black.opacity(0.04),
            radius: 4,
            x: 0,
            y: 2
        )
        
        static let md = ShadowStyle(
            color: Color.black.opacity(0.08),
            radius: 8,
            x: 0,
            y: 4
        )
        
        static let lg = ShadowStyle(
            color: Color.black.opacity(0.12),
            radius: 16,
            x: 0,
            y: 8
        )
        
        struct ShadowStyle {
            let color: Color
            let radius: CGFloat
            let x: CGFloat
            let y: CGFloat
        }
    }
}

// MARK: - Korean UI Components
struct KoreanUIComponents {
    
    // MARK: - Bottom Navigation (한국 앱 필수)
    struct KoreanBottomNavigation: View {
        @Binding var selectedTab: Int
        
        let tabs = [
            TabItem(icon: "house.fill", title: "홈", badge: nil),
            TabItem(icon: "magnifyingglass", title: "검색", badge: nil),
            TabItem(icon: "plus.circle.fill", title: "등록", badge: nil),
            TabItem(icon: "heart", title: "찜", badge: "3"),
            TabItem(icon: "person", title: "MY", badge: "N")
        ]
        
        var body: some View {
            HStack(spacing: 0) {
                ForEach(0..<tabs.count, id: \.self) { index in
                    TabButton(
                        item: tabs[index],
                        isSelected: selectedTab == index
                    ) {
                        selectedTab = index
                    }
                }
            }
            .padding(.horizontal, 8)
            .padding(.top, 8)
            .padding(.bottom, 8 + safeAreaBottom)
            .background(
                Color.white
                    .shadow(
                        color: Color.black.opacity(0.08),
                        radius: 8,
                        x: 0,
                        y: -2
                    )
            )
        }
        
        var safeAreaBottom: CGFloat {
            UIApplication.shared.windows.first?.safeAreaInsets.bottom ?? 0
        }
        
        struct TabItem {
            let icon: String
            let title: String
            let badge: String?
        }
        
        struct TabButton: View {
            let item: TabItem
            let isSelected: Bool
            let action: () -> Void
            
            var body: some View {
                Button(action: action) {
                    VStack(spacing: 4) {
                        ZStack(alignment: .topTrailing) {
                            Image(systemName: item.icon)
                                .font(.system(size: 24))
                                .foregroundColor(
                                    isSelected ? 
                                    KoreanDesignSystem.Colors.primary : 
                                    Color(.systemGray3)
                                )
                            
                            if let badge = item.badge {
                                BadgeView(text: badge)
                                    .offset(x: 12, y: -4)
                            }
                        }
                        
                        Text(item.title)
                            .font(.system(size: 10))
                            .foregroundColor(
                                isSelected ? 
                                KoreanDesignSystem.Colors.primary : 
                                Color(.systemGray3)
                            )
                    }
                    .frame(maxWidth: .infinity)
                }
            }
        }
        
        struct BadgeView: View {
            let text: String
            
            var body: some View {
                Text(text)
                    .font(.system(size: 9, weight: .bold))
                    .foregroundColor(.white)
                    .padding(.horizontal, text.count == 1 ? 5 : 4)
                    .padding(.vertical, 2)
                    .background(Color.red)
                    .cornerRadius(10)
            }
        }
    }
    
    // MARK: - Floating Action Button (한국 앱 스타일)
    struct KoreanFAB: View {
        let icon: String
        let action: () -> Void
        @State private var isPressed = false
        
        var body: some View {
            Button(action: action) {
                ZStack {
                    Circle()
                        .fill(KoreanDesignSystem.Colors.primary)
                        .frame(width: 56, height: 56)
                    
                    Image(systemName: icon)
                        .font(.system(size: 24))
                        .foregroundColor(.white)
                }
            }
            .scaleEffect(isPressed ? 0.95 : 1.0)
            .shadow(
                color: KoreanDesignSystem.Colors.primary.opacity(0.3),
                radius: 8,
                x: 0,
                y: 4
            )
            .onLongPressGesture(
                minimumDuration: 0,
                maximumDistance: .infinity,
                pressing: { pressing in
                    withAnimation(.easeInOut(duration: 0.1)) {
                        isPressed = pressing
                    }
                },
                perform: {}
            )
        }
    }
    
    // MARK: - 한국형 검색바
    struct KoreanSearchBar: View {
        @Binding var text: String
        var placeholder: String = "검색어를 입력하세요"
        var onSearch: () -> Void = {}
        
        @FocusState private var isFocused: Bool
        @State private var showingVoiceSearch = false
        
        var body: some View {
            HStack(spacing: 12) {
                HStack(spacing: 8) {
                    Image(systemName: "magnifyingglass")
                        .foregroundColor(Color(.systemGray2))
                    
                    TextField(placeholder, text: $text)
                        .focused($isFocused)
                        .onSubmit {
                            onSearch()
                        }
                    
                    if !text.isEmpty {
                        Button {
                            text = ""
                        } label: {
                            Image(systemName: "xmark.circle.fill")
                                .foregroundColor(Color(.systemGray3))
                        }
                    }
                    
                    Button {
                        showingVoiceSearch = true
                    } label: {
                        Image(systemName: "mic.fill")
                            .foregroundColor(KoreanDesignSystem.Colors.primary)
                    }
                }
                .padding(.horizontal, 12)
                .padding(.vertical, 10)
                .background(Color(.systemGray6))
                .cornerRadius(10)
                
                if isFocused {
                    Button("취소") {
                        isFocused = false
                        text = ""
                    }
                    .foregroundColor(KoreanDesignSystem.Colors.primary)
                    .transition(.move(edge: .trailing).combined(with: .opacity))
                }
            }
            .animation(.easeInOut(duration: 0.2), value: isFocused)
        }
    }
    
    // MARK: - 한국형 상품 카드
    struct KoreanProductCard: View {
        let imageName: String
        let title: String
        let subtitle: String
        let price: Int
        let originalPrice: Int?
        let discountRate: Int?
        let rating: Double
        let reviewCount: Int
        let isLiked: Bool
        let isFreeShipping: Bool
        let isRocketDelivery: Bool // 쿠팡 스타일
        
        var body: some View {
            VStack(alignment: .leading, spacing: 0) {
                // 이미지 영역
                ZStack(alignment: .topTrailing) {
                    Rectangle()
                        .fill(Color(.systemGray6))
                        .aspectRatio(1, contentMode: .fit)
                        .overlay(
                            Image(imageName)
                                .resizable()
                                .aspectRatio(contentMode: .fill)
                        )
                        .clipped()
                    
                    // 할인율 배지
                    if let discount = discountRate {
                        Text("\(discount)%")
                            .font(.system(size: 14, weight: .bold))
                            .foregroundColor(.white)
                            .padding(.horizontal, 8)
                            .padding(.vertical, 4)
                            .background(Color.red)
                            .cornerRadius(4)
                            .padding(8)
                    }
                    
                    // 찜 버튼
                    Button {
                        // Toggle like
                    } label: {
                        Image(systemName: isLiked ? "heart.fill" : "heart")
                            .foregroundColor(isLiked ? .red : .white)
                            .padding(8)
                            .background(Color.black.opacity(0.3))
                            .clipShape(Circle())
                    }
                    .padding(8)
                }
                
                // 정보 영역
                VStack(alignment: .leading, spacing: 4) {
                    // 브랜드/카테고리
                    Text(subtitle)
                        .font(.system(size: 12))
                        .foregroundColor(Color(.systemGray))
                    
                    // 상품명
                    Text(title)
                        .font(.system(size: 14))
                        .foregroundColor(KoreanDesignSystem.Colors.textPrimary)
                        .lineLimit(2)
                        .fixedSize(horizontal: false, vertical: true)
                    
                    // 가격 영역
                    HStack(alignment: .firstTextBaseline, spacing: 4) {
                        if let original = originalPrice {
                            Text("₩\(original.formatted())")
                                .font(.system(size: 12))
                                .foregroundColor(Color(.systemGray2))
                                .strikethrough()
                        }
                        
                        Text("₩\(price.formatted())")
                            .font(.system(size: 16, weight: .bold))
                            .foregroundColor(KoreanDesignSystem.Colors.textPrimary)
                    }
                    
                    // 평점 및 리뷰
                    HStack(spacing: 4) {
                        HStack(spacing: 2) {
                            Image(systemName: "star.fill")
                                .font(.system(size: 12))
                                .foregroundColor(.yellow)
                            
                            Text(String(format: "%.1f", rating))
                                .font(.system(size: 12))
                                .foregroundColor(KoreanDesignSystem.Colors.textPrimary)
                        }
                        
                        Text("(\(reviewCount))")
                            .font(.system(size: 12))
                            .foregroundColor(Color(.systemGray2))
                    }
                    
                    // 배송 정보
                    HStack(spacing: 4) {
                        if isRocketDelivery {
                            Label("로켓배송", systemImage: "bolt.fill")
                                .font(.system(size: 11, weight: .bold))
                                .foregroundColor(Color(hex: "0073E9"))
                        } else if isFreeShipping {
                            Text("무료배송")
                                .font(.system(size: 11))
                                .foregroundColor(KoreanDesignSystem.Colors.accent)
                        }
                    }
                }
                .padding(12)
            }
            .background(Color.white)
            .cornerRadius(8)
            .shadow(
                color: Color.black.opacity(0.08),
                radius: 4,
                x: 0,
                y: 2
            )
        }
    }
    
    // MARK: - 한국형 배너 슬라이더
    struct KoreanBannerSlider: View {
        let banners: [Banner]
        @State private var currentIndex = 0
        @State private var timer: Timer?
        
        struct Banner {
            let id: String
            let imageUrl: String
            let title: String
            let subtitle: String
            let link: String
        }
        
        var body: some View {
            GeometryReader { geometry in
                ZStack(alignment: .bottom) {
                    // 배너 이미지
                    TabView(selection: $currentIndex) {
                        ForEach(0..<banners.count, id: \.self) { index in
                            BannerView(banner: banners[index])
                                .tag(index)
                        }
                    }
                    .tabViewStyle(PageTabViewStyle(indexDisplayMode: .never))
                    
                    // 페이지 인디케이터
                    HStack(spacing: 4) {
                        ForEach(0..<banners.count, id: \.self) { index in
                            Circle()
                                .fill(currentIndex == index ? Color.white : Color.white.opacity(0.5))
                                .frame(width: currentIndex == index ? 8 : 6,
                                       height: currentIndex == index ? 8 : 6)
                                .animation(.easeInOut(duration: 0.2), value: currentIndex)
                        }
                    }
                    .padding(.horizontal, 12)
                    .padding(.vertical, 8)
                    .background(Color.black.opacity(0.3))
                    .cornerRadius(12)
                    .padding(.bottom, 16)
                }
                .frame(height: geometry.size.width * 0.5) // 16:9 비율
            }
            .onAppear {
                startAutoScroll()
            }
            .onDisappear {
                timer?.invalidate()
            }
        }
        
        func startAutoScroll() {
            timer = Timer.scheduledTimer(withTimeInterval: 3.0, repeats: true) { _ in
                withAnimation {
                    currentIndex = (currentIndex + 1) % banners.count
                }
            }
        }
        
        struct BannerView: View {
            let banner: Banner
            
            var body: some View {
                ZStack(alignment: .bottomLeading) {
                    // 배너 이미지
                    Rectangle()
                        .fill(
                            LinearGradient(
                                colors: [
                                    KoreanDesignSystem.Colors.primary,
                                    KoreanDesignSystem.Colors.primary.opacity(0.7)
                                ],
                                startPoint: .topLeading,
                                endPoint: .bottomTrailing
                            )
                        )
                    
                    // 텍스트 오버레이
                    VStack(alignment: .leading, spacing: 4) {
                        Text(banner.subtitle)
                            .font(.system(size: 14))
                            .foregroundColor(.white.opacity(0.9))
                        
                        Text(banner.title)
                            .font(.system(size: 20, weight: .bold))
                            .foregroundColor(.white)
                    }
                    .padding(20)
                }
            }
        }
    }
}

// MARK: - Korean Onboarding Flow
struct KoreanOnboardingView: View {
    @State private var currentPage = 0
    @State private var showAgeVerification = false
    @State private var agreedToAll = false
    @State private var agreements = AgreementStatus()
    
    struct AgreementStatus {
        var terms = false
        var privacy = false
        var marketing = false
        var location = false
        var push = false
    }
    
    let pages = [
        OnboardingPage(
            title: "레시피박스에 오신 것을\n환영합니다",
            subtitle: "10만개의 레시피로\n오늘의 요리를 시작하세요",
            imageName: "onboarding1"
        ),
        OnboardingPage(
            title: "AI가 추천하는\n맞춤 레시피",
            subtitle: "취향과 재료에 맞는\n완벽한 레시피를 찾아드려요",
            imageName: "onboarding2"
        ),
        OnboardingPage(
            title: "함께 요리하는\n즐거움",
            subtitle: "레시피를 공유하고\n요리 친구를 만나보세요",
            imageName: "onboarding3"
        )
    ]
    
    var body: some View {
        ZStack {
            if currentPage < pages.count {
                // 온보딩 페이지
                VStack(spacing: 0) {
                    // Skip 버튼
                    HStack {
                        Spacer()
                        Button("건너뛰기") {
                            currentPage = pages.count
                        }
                        .foregroundColor(Color(.systemGray2))
                        .padding()
                    }
                    
                    // 페이지 콘텐츠
                    TabView(selection: $currentPage) {
                        ForEach(0..<pages.count, id: \.self) { index in
                            OnboardingPageView(page: pages[index])
                                .tag(index)
                        }
                    }
                    .tabViewStyle(PageTabViewStyle(indexDisplayMode: .never))
                    
                    // 페이지 인디케이터
                    HStack(spacing: 8) {
                        ForEach(0..<pages.count, id: \.self) { index in
                            Circle()
                                .fill(currentPage == index ? 
                                      KoreanDesignSystem.Colors.primary : 
                                      Color(.systemGray4))
                                .frame(width: 8, height: 8)
                                .animation(.easeInOut, value: currentPage)
                        }
                    }
                    .padding(.vertical, 20)
                    
                    // 다음 버튼
                    Button {
                        if currentPage < pages.count - 1 {
                            withAnimation {
                                currentPage += 1
                            }
                        } else {
                            currentPage = pages.count
                        }
                    } label: {
                        Text(currentPage < pages.count - 1 ? "다음" : "시작하기")
                            .font(.system(size: 16, weight: .semibold))
                            .foregroundColor(.white)
                            .frame(maxWidth: .infinity)
                            .frame(height: 52)
                            .background(KoreanDesignSystem.Colors.primary)
                            .cornerRadius(12)
                    }
                    .padding(.horizontal, 20)
                    .padding(.bottom, 40)
                }
                
            } else if !showAgeVerification {
                // 약관 동의
                AgreementView(
                    agreedToAll: $agreedToAll,
                    agreements: $agreements,
                    onComplete: {
                        showAgeVerification = true
                    }
                )
                
            } else {
                // 연령 인증
                AgeVerificationView()
            }
        }
    }
    
    struct OnboardingPage {
        let title: String
        let subtitle: String
        let imageName: String
    }
    
    struct OnboardingPageView: View {
        let page: OnboardingPage
        
        var body: some View {
            VStack(spacing: 40) {
                Spacer()
                
                // 일러스트
                Image(systemName: "photo")
                    .font(.system(size: 120))
                    .foregroundColor(KoreanDesignSystem.Colors.primary.opacity(0.3))
                
                // 텍스트
                VStack(spacing: 16) {
                    Text(page.title)
                        .font(.system(size: 28, weight: .bold))
                        .multilineTextAlignment(.center)
                        .foregroundColor(KoreanDesignSystem.Colors.textPrimary)
                    
                    Text(page.subtitle)
                        .font(.system(size: 16))
                        .multilineTextAlignment(.center)
                        .foregroundColor(KoreanDesignSystem.Colors.textSecondary)
                }
                .padding(.horizontal, 40)
                
                Spacer()
                Spacer()
            }
        }
    }
}

// MARK: - Agreement View (약관 동의)
struct AgreementView: View {
    @Binding var agreedToAll: Bool
    @Binding var agreements: KoreanOnboardingView.AgreementStatus
    let onComplete: () -> Void
    
    var canProceed: Bool {
        agreements.terms && agreements.privacy
    }
    
    var body: some View {
        VStack(spacing: 0) {
            // 헤더
            VStack(spacing: 8) {
                Text("약관에 동의해주세요")
                    .font(.system(size: 24, weight: .bold))
                
                Text("서비스 이용을 위해 약관 동의가 필요합니다")
                    .font(.system(size: 14))
                    .foregroundColor(Color(.systemGray))
            }
            .padding(.top, 60)
            .padding(.bottom, 40)
            
            // 전체 동의
            Button {
                agreedToAll.toggle()
                if agreedToAll {
                    agreements.terms = true
                    agreements.privacy = true
                    agreements.marketing = true
                    agreements.location = true
                    agreements.push = true
                } else {
                    agreements.terms = false
                    agreements.privacy = false
                    agreements.marketing = false
                    agreements.location = false
                    agreements.push = false
                }
            } label: {
                HStack {
                    Image(systemName: agreedToAll ? "checkmark.circle.fill" : "circle")
                        .font(.system(size: 24))
                        .foregroundColor(agreedToAll ? KoreanDesignSystem.Colors.primary : Color(.systemGray3))
                    
                    Text("전체 동의")
                        .font(.system(size: 18, weight: .semibold))
                        .foregroundColor(KoreanDesignSystem.Colors.textPrimary)
                    
                    Spacer()
                }
                .padding(.horizontal, 20)
                .padding(.vertical, 16)
                .background(Color(.systemGray6))
                .cornerRadius(12)
            }
            .padding(.horizontal, 20)
            
            // 구분선
            Rectangle()
                .fill(Color(.systemGray5))
                .frame(height: 1)
                .padding(.horizontal, 20)
                .padding(.vertical, 20)
            
            // 개별 약관
            VStack(spacing: 0) {
                AgreementRow(
                    isChecked: $agreements.terms,
                    title: "[필수] 이용약관",
                    isRequired: true
                )
                
                AgreementRow(
                    isChecked: $agreements.privacy,
                    title: "[필수] 개인정보 수집 및 이용",
                    isRequired: true
                )
                
                AgreementRow(
                    isChecked: $agreements.marketing,
                    title: "[선택] 마케팅 정보 수신",
                    isRequired: false
                )
                
                AgreementRow(
                    isChecked: $agreements.location,
                    title: "[선택] 위치 정보 이용",
                    isRequired: false
                )
                
                AgreementRow(
                    isChecked: $agreements.push,
                    title: "[선택] 푸시 알림 수신",
                    isRequired: false
                )
            }
            .padding(.horizontal, 20)
            
            Spacer()
            
            // 다음 버튼
            Button {
                onComplete()
            } label: {
                Text("다음")
                    .font(.system(size: 16, weight: .semibold))
                    .foregroundColor(.white)
                    .frame(maxWidth: .infinity)
                    .frame(height: 52)
                    .background(canProceed ? KoreanDesignSystem.Colors.primary : Color(.systemGray4))
                    .cornerRadius(12)
            }
            .disabled(!canProceed)
            .padding(.horizontal, 20)
            .padding(.bottom, 40)
        }
    }
    
    struct AgreementRow: View {
        @Binding var isChecked: Bool
        let title: String
        let isRequired: Bool
        
        var body: some View {
            Button {
                isChecked.toggle()
            } label: {
                HStack {
                    Image(systemName: isChecked ? "checkmark.square.fill" : "square")
                        .font(.system(size: 20))
                        .foregroundColor(isChecked ? KoreanDesignSystem.Colors.primary : Color(.systemGray3))
                    
                    Text(title)
                        .font(.system(size: 14))
                        .foregroundColor(KoreanDesignSystem.Colors.textPrimary)
                    
                    Spacer()
                    
                    Image(systemName: "chevron.right")
                        .font(.system(size: 12))
                        .foregroundColor(Color(.systemGray3))
                }
                .padding(.vertical, 12)
            }
        }
    }
}

// MARK: - Age Verification (연령 인증)
struct AgeVerificationView: View {
    @State private var selectedMethod = 0
    @State private var phoneNumber = ""
    @State private var verificationCode = ""
    @State private var isCodeSent = false
    
    let methods = ["휴대폰 인증", "아이핀 인증", "간편 인증"]
    
    var body: some View {
        VStack(spacing: 0) {
            // 헤더
            VStack(spacing: 8) {
                Text("본인 인증")
                    .font(.system(size: 24, weight: .bold))
                
                Text("안전한 서비스 이용을 위해 본인 인증이 필요합니다")
                    .font(.system(size: 14))
                    .foregroundColor(Color(.systemGray))
                    .multilineTextAlignment(.center)
            }
            .padding(.top, 60)
            .padding(.bottom, 30)
            
            // 인증 방법 선택
            Picker("인증 방법", selection: $selectedMethod) {
                ForEach(0..<methods.count, id: \.self) { index in
                    Text(methods[index]).tag(index)
                }
            }
            .pickerStyle(SegmentedPickerStyle())
            .padding(.horizontal, 20)
            .padding(.bottom, 30)
            
            if selectedMethod == 0 {
                // 휴대폰 인증
                VStack(spacing: 16) {
                    // 통신사 선택
                    HStack(spacing: 12) {
                        ForEach(["SKT", "KT", "LG U+", "알뜰폰"], id: \.self) { carrier in
                            Button {
                                // Select carrier
                            } label: {
                                Text(carrier)
                                    .font(.system(size: 14))
                                    .foregroundColor(KoreanDesignSystem.Colors.textPrimary)
                                    .frame(maxWidth: .infinity)
                                    .frame(height: 44)
                                    .background(Color(.systemGray6))
                                    .cornerRadius(8)
                            }
                        }
                    }
                    
                    // 휴대폰 번호
                    HStack {
                        TextField("휴대폰 번호 (- 없이)", text: $phoneNumber)
                            .keyboardType(.numberPad)
                            .padding(.horizontal, 12)
                            .frame(height: 48)
                            .background(Color(.systemGray6))
                            .cornerRadius(8)
                        
                        Button {
                            isCodeSent = true
                        } label: {
                            Text("인증번호 받기")
                                .font(.system(size: 14, weight: .medium))
                                .foregroundColor(.white)
                                .padding(.horizontal, 16)
                                .frame(height: 48)
                                .background(
                                    phoneNumber.count >= 10 ? 
                                    KoreanDesignSystem.Colors.primary : 
                                    Color(.systemGray4)
                                )
                                .cornerRadius(8)
                        }
                        .disabled(phoneNumber.count < 10)
                    }
                    
                    if isCodeSent {
                        // 인증번호 입력
                        VStack(spacing: 8) {
                            TextField("인증번호 6자리", text: $verificationCode)
                                .keyboardType(.numberPad)
                                .padding(.horizontal, 12)
                                .frame(height: 48)
                                .background(Color(.systemGray6))
                                .cornerRadius(8)
                            
                            HStack {
                                Text("남은 시간 02:48")
                                    .font(.system(size: 12))
                                    .foregroundColor(.red)
                                
                                Spacer()
                                
                                Button("재전송") {
                                    // Resend code
                                }
                                .font(.system(size: 12))
                                .foregroundColor(KoreanDesignSystem.Colors.primary)
                            }
                        }
                    }
                }
                .padding(.horizontal, 20)
                
            } else if selectedMethod == 2 {
                // 간편 인증 (카카오, 네이버, 패스 등)
                VStack(spacing: 12) {
                    SimpleAuthButton(
                        title: "카카오 인증",
                        icon: "kakao",
                        backgroundColor: Color(hex: "FEE500"),
                        textColor: .black
                    )
                    
                    SimpleAuthButton(
                        title: "네이버 인증",
                        icon: "naver",
                        backgroundColor: Color(hex: "03C75A"),
                        textColor: .white
                    )
                    
                    SimpleAuthButton(
                        title: "PASS 인증",
                        icon: "pass",
                        backgroundColor: Color(hex: "FF6B6B"),
                        textColor: .white
                    )
                    
                    SimpleAuthButton(
                        title: "토스 인증",
                        icon: "toss",
                        backgroundColor: Color(hex: "0064FF"),
                        textColor: .white
                    )
                }
                .padding(.horizontal, 20)
            }
            
            Spacer()
            
            // 인증 버튼
            Button {
                // Complete verification
            } label: {
                Text("인증 완료")
                    .font(.system(size: 16, weight: .semibold))
                    .foregroundColor(.white)
                    .frame(maxWidth: .infinity)
                    .frame(height: 52)
                    .background(
                        verificationCode.count == 6 ? 
                        KoreanDesignSystem.Colors.primary : 
                        Color(.systemGray4)
                    )
                    .cornerRadius(12)
            }
            .disabled(verificationCode.count != 6)
            .padding(.horizontal, 20)
            .padding(.bottom, 40)
        }
    }
    
    struct SimpleAuthButton: View {
        let title: String
        let icon: String
        let backgroundColor: Color
        let textColor: Color
        
        var body: some View {
            Button {
                // Authenticate
            } label: {
                HStack {
                    Image(systemName: "lock.shield")
                        .font(.system(size: 20))
                    
                    Text(title)
                        .font(.system(size: 16, weight: .medium))
                    
                    Spacer()
                }
                .foregroundColor(textColor)
                .padding(.horizontal, 20)
                .frame(height: 52)
                .background(backgroundColor)
                .cornerRadius(12)
            }
        }
    }
}

// MARK: - Korean Style Loading View
struct KoreanLoadingView: View {
    @State private var isAnimating = false
    let message: String
    
    var body: some View {
        VStack(spacing: 20) {
            // 로딩 애니메이션
            ZStack {
                ForEach(0..<3) { index in
                    Circle()
                        .fill(KoreanDesignSystem.Colors.primary.opacity(0.3))
                        .frame(width: 10, height: 10)
                        .offset(x: CGFloat(index - 1) * 20)
                        .scaleEffect(isAnimating ? 1.0 : 0.5)
                        .animation(
                            Animation.easeInOut(duration: 0.6)
                                .repeatForever()
                                .delay(Double(index) * 0.2),
                            value: isAnimating
                        )
                }
            }
            
            Text(message)
                .font(.system(size: 14))
                .foregroundColor(KoreanDesignSystem.Colors.textSecondary)
        }
        .padding(40)
        .background(Color.white)
        .cornerRadius(16)
        .shadow(
            color: Color.black.opacity(0.1),
            radius: 10,
            x: 0,
            y: 5
        )
        .onAppear {
            isAnimating = true
        }
    }
}

// MARK: - Korean Toast/Snackbar
struct KoreanToast: View {
    let message: String
    let type: ToastType
    
    enum ToastType {
        case success, error, warning, info
        
        var color: Color {
            switch self {
            case .success: return KoreanDesignSystem.Colors.success
            case .error: return KoreanDesignSystem.Colors.error
            case .warning: return KoreanDesignSystem.Colors.warning
            case .info: return KoreanDesignSystem.Colors.info
            }
        }
        
        var icon: String {
            switch self {
            case .success: return "checkmark.circle.fill"
            case .error: return "xmark.circle.fill"
            case .warning: return "exclamationmark.triangle.fill"
            case .info: return "info.circle.fill"
            }
        }
    }
    
    var body: some View {
        HStack(spacing: 12) {
            Image(systemName: type.icon)
                .foregroundColor(.white)
            
            Text(message)
                .font(.system(size: 14))
                .foregroundColor(.white)
            
            Spacer()
        }
        .padding(.horizontal, 16)
        .padding(.vertical, 12)
        .background(type.color)
        .cornerRadius(8)
        .shadow(
            color: type.color.opacity(0.3),
            radius: 8,
            x: 0,
            y: 4
        )
    }
}

// MARK: - Extensions
extension Color {
    init(hex: String) {
        let hex = hex.trimmingCharacters(in: CharacterSet.alphanumerics.inverted)
        var int: UInt64 = 0
        Scanner(string: hex).scanHexInt64(&int)
        let a, r, g, b: UInt64
        switch hex.count {
        case 3: // RGB (12-bit)
            (a, r, g, b) = (255, (int >> 8) * 17, (int >> 4 & 0xF) * 17, (int & 0xF) * 17)
        case 6: // RGB (24-bit)
            (a, r, g, b) = (255, int >> 16, int >> 8 & 0xFF, int & 0xFF)
        case 8: // ARGB (32-bit)
            (a, r, g, b) = (int >> 24, int >> 16 & 0xFF, int >> 8 & 0xFF, int & 0xFF)
        default:
            (a, r, g, b) = (1, 1, 1, 0)
        }

        self.init(
            .sRGB,
            red: Double(r) / 255,
            green: Double(g) / 255,
            blue:  Double(b) / 255,
            opacity: Double(a) / 255
        )
    }
}

// MARK: - Preview
struct KoreanUXPreview: PreviewProvider {
    static var previews: some View {
        KoreanOnboardingView()
    }
}
```