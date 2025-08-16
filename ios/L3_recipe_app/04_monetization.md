# RecipeBox: 수익화 시스템 완벽 구현

## StoreKit 2 구독 & 인앱 구매

```swift
import SwiftUI
import StoreKit
import Combine

// MARK: - Product Identifiers
enum ProductIdentifier: String, CaseIterable {
    // Subscriptions
    case premiumMonthly = "com.recipebox.premium.monthly"
    case premiumYearly = "com.recipebox.premium.yearly"
    case proMonthly = "com.recipebox.pro.monthly"
    case proYearly = "com.recipebox.pro.yearly"
    
    // One-time Purchases
    case removeAds = "com.recipebox.removeads"
    case recipePack_korean = "com.recipebox.pack.korean"
    case recipePack_italian = "com.recipebox.pack.italian"
    case recipePack_dessert = "com.recipebox.pack.dessert"
    case masterClass_basics = "com.recipebox.class.basics"
    case masterClass_advanced = "com.recipebox.class.advanced"
    
    // Consumables
    case credits_100 = "com.recipebox.credits.100"
    case credits_500 = "com.recipebox.credits.500"
    case credits_1000 = "com.recipebox.credits.1000"
    
    var isSubscription: Bool {
        switch self {
        case .premiumMonthly, .premiumYearly, .proMonthly, .proYearly:
            return true
        default:
            return false
        }
    }
    
    var isConsumable: Bool {
        switch self {
        case .credits_100, .credits_500, .credits_1000:
            return true
        default:
            return false
        }
    }
}

// MARK: - Store Manager
@MainActor
@Observable
final class StoreManager: NSObject {
    static let shared = StoreManager()
    
    // Products
    private(set) var products: [Product] = []
    private(set) var purchasedProducts: Set<Product> = []
    private(set) var purchasedSubscriptions: [Product] = []
    
    // Subscription Status
    private(set) var subscriptionStatus: Product.SubscriptionInfo.Status?
    private(set) var isSubscribed: Bool = false
    private(set) var subscriptionLevel: SubscriptionLevel = .free
    
    // Transaction Listener
    private var updateListenerTask: Task<Void, Error>?
    
    // Credits
    @AppStorage("user_credits") private var userCredits: Int = 0
    
    // State
    private(set) var isLoading = false
    private(set) var errorMessage: String?
    
    override init() {
        super.init()
        
        // Start transaction listener
        updateListenerTask = listenForTransactions()
        
        // Load products
        Task {
            await loadProducts()
            await updateCustomerProductStatus()
        }
    }
    
    deinit {
        updateListenerTask?.cancel()
    }
    
    // MARK: - Load Products
    func loadProducts() async {
        isLoading = true
        
        do {
            // Request products from App Store
            let storeProducts = try await Product.products(for: ProductIdentifier.allCases.map { $0.rawValue })
            
            // Sort products
            products = storeProducts.sorted { $0.price < $1.price }
            
            isLoading = false
        } catch {
            print("Failed to load products: \(error)")
            errorMessage = "상품을 불러올 수 없습니다"
            isLoading = false
        }
    }
    
    // MARK: - Purchase Product
    func purchase(_ product: Product) async throws -> Transaction? {
        // Start purchase
        let result = try await product.purchase()
        
        switch result {
        case .success(let verification):
            // Check verification
            let transaction = try checkVerified(verification)
            
            // Update customer status
            await updateCustomerProductStatus()
            
            // Deliver content
            await deliverPurchasedContent(for: transaction)
            
            // Finish transaction
            await transaction.finish()
            
            return transaction
            
        case .userCancelled:
            return nil
            
        case .pending:
            // Handle pending transactions (e.g., parental approval)
            return nil
            
        @unknown default:
            return nil
        }
    }
    
    // MARK: - Restore Purchases
    func restorePurchases() async {
        // Sync with App Store
        try? await AppStore.sync()
        
        // Update status
        await updateCustomerProductStatus()
    }
    
    // MARK: - Update Customer Status
    @MainActor
    func updateCustomerProductStatus() async {
        var purchasedProducts: Set<Product> = []
        var purchasedSubscriptions: [Product] = []
        
        // Check all transactions
        for await result in Transaction.currentEntitlements {
            do {
                let transaction = try checkVerified(result)
                
                // Find matching product
                if let product = products.first(where: { $0.id == transaction.productID }) {
                    purchasedProducts.insert(product)
                    
                    if product.type == .autoRenewable {
                        purchasedSubscriptions.append(product)
                    }
                }
            } catch {
                print("Transaction verification failed: \(error)")
            }
        }
        
        // Update properties
        self.purchasedProducts = purchasedProducts
        self.purchasedSubscriptions = purchasedSubscriptions
        
        // Update subscription status
        await updateSubscriptionStatus()
    }
    
    // MARK: - Update Subscription Status
    @MainActor
    private func updateSubscriptionStatus() async {
        // Check subscription status
        guard let subscription = purchasedSubscriptions.first else {
            subscriptionStatus = nil
            isSubscribed = false
            subscriptionLevel = .free
            return
        }
        
        do {
            // Get subscription status
            let statuses = try await subscription.subscription?.status ?? []
            
            // Find valid subscription
            for status in statuses {
                if status.state == .subscribed || status.state == .inGracePeriod {
                    subscriptionStatus = status
                    isSubscribed = true
                    
                    // Determine subscription level
                    switch subscription.id {
                    case ProductIdentifier.premiumMonthly.rawValue,
                         ProductIdentifier.premiumYearly.rawValue:
                        subscriptionLevel = .premium
                    case ProductIdentifier.proMonthly.rawValue,
                         ProductIdentifier.proYearly.rawValue:
                        subscriptionLevel = .pro
                    default:
                        subscriptionLevel = .free
                    }
                    
                    return
                }
            }
            
            // No valid subscription
            subscriptionStatus = nil
            isSubscribed = false
            subscriptionLevel = .free
            
        } catch {
            print("Failed to check subscription status: \(error)")
            subscriptionStatus = nil
            isSubscribed = false
            subscriptionLevel = .free
        }
    }
    
    // MARK: - Listen for Transactions
    private func listenForTransactions() -> Task<Void, Error> {
        return Task.detached {
            // Listen for transaction updates
            for await result in Transaction.updates {
                do {
                    let transaction = try self.checkVerified(result)
                    
                    // Update customer status
                    await self.updateCustomerProductStatus()
                    
                    // Deliver content
                    await self.deliverPurchasedContent(for: transaction)
                    
                    // Finish transaction
                    await transaction.finish()
                } catch {
                    print("Transaction failed verification: \(error)")
                }
            }
        }
    }
    
    // MARK: - Verify Transaction
    private func checkVerified<T>(_ result: VerificationResult<T>) throws -> T {
        switch result {
        case .unverified:
            throw StoreError.failedVerification
        case .verified(let safe):
            return safe
        }
    }
    
    // MARK: - Deliver Content
    @MainActor
    private func deliverPurchasedContent(for transaction: Transaction) async {
        // Deliver content based on product
        guard let identifier = ProductIdentifier(rawValue: transaction.productID) else { return }
        
        switch identifier {
        case .credits_100:
            userCredits += 100
            await trackPurchase(identifier, amount: 100)
            
        case .credits_500:
            userCredits += 500
            await trackPurchase(identifier, amount: 500)
            
        case .credits_1000:
            userCredits += 1000
            await trackPurchase(identifier, amount: 1000)
            
        case .removeAds:
            UserDefaults.standard.set(true, forKey: "ads_removed")
            await NotificationCenter.default.post(name: .adsRemoved, object: nil)
            
        case .recipePack_korean, .recipePack_italian, .recipePack_dessert:
            await unlockRecipePack(identifier)
            
        case .masterClass_basics, .masterClass_advanced:
            await unlockMasterClass(identifier)
            
        default:
            break
        }
    }
    
    // MARK: - Helper Methods
    private func unlockRecipePack(_ identifier: ProductIdentifier) async {
        // Unlock recipe pack in database
        let packId = identifier.rawValue
        UserDefaults.standard.set(true, forKey: "pack_\(packId)")
        
        // Download recipes if needed
        await RecipePackManager.shared.downloadPack(packId)
    }
    
    private func unlockMasterClass(_ identifier: ProductIdentifier) async {
        // Unlock master class
        let classId = identifier.rawValue
        UserDefaults.standard.set(true, forKey: "class_\(classId)")
        
        // Download class content
        await MasterClassManager.shared.downloadClass(classId)
    }
    
    private func trackPurchase(_ identifier: ProductIdentifier, amount: Int? = nil) async {
        // Analytics tracking
        var properties: [String: Any] = ["product_id": identifier.rawValue]
        if let amount = amount {
            properties["amount"] = amount
        }
        
        Analytics.track(.purchase, properties: properties)
    }
    
    // MARK: - Check Access
    func hasAccess(to feature: PremiumFeature) -> Bool {
        switch feature {
        case .removeAds:
            return UserDefaults.standard.bool(forKey: "ads_removed") || subscriptionLevel != .free
            
        case .unlimitedRecipes:
            return subscriptionLevel != .free
            
        case .advancedFilters:
            return subscriptionLevel != .free
            
        case .mealPlanning:
            return subscriptionLevel == .premium || subscriptionLevel == .pro
            
        case .nutritionTracking:
            return subscriptionLevel == .premium || subscriptionLevel == .pro
            
        case .chefConsultation:
            return subscriptionLevel == .pro
            
        case .liveClasses:
            return subscriptionLevel == .pro
            
        case .exclusiveRecipes:
            return subscriptionLevel == .pro
        }
    }
    
    // MARK: - Manage Subscription
    func openSubscriptionManagement() async {
        if let windowScene = UIApplication.shared.connectedScenes.first as? UIWindowScene {
            do {
                try await AppStore.showManageSubscriptions(in: windowScene)
            } catch {
                print("Failed to show subscription management: \(error)")
            }
        }
    }
}

// MARK: - Subscription Levels
enum SubscriptionLevel: String, CaseIterable {
    case free = "무료"
    case premium = "프리미엄"
    case pro = "프로"
    
    var features: [PremiumFeature] {
        switch self {
        case .free:
            return []
        case .premium:
            return [.removeAds, .unlimitedRecipes, .advancedFilters, .mealPlanning, .nutritionTracking]
        case .pro:
            return PremiumFeature.allCases
        }
    }
    
    var color: Color {
        switch self {
        case .free: return .gray
        case .premium: return .blue
        case .pro: return .purple
        }
    }
    
    var icon: String {
        switch self {
        case .free: return "person"
        case .premium: return "star"
        case .pro: return "crown"
        }
    }
}

// MARK: - Premium Features
enum PremiumFeature: String, CaseIterable {
    case removeAds = "광고 제거"
    case unlimitedRecipes = "무제한 레시피"
    case advancedFilters = "고급 필터"
    case mealPlanning = "식단 계획"
    case nutritionTracking = "영양 추적"
    case chefConsultation = "셰프 상담"
    case liveClasses = "라이브 클래스"
    case exclusiveRecipes = "독점 레시피"
    
    var icon: String {
        switch self {
        case .removeAds: return "xmark.circle"
        case .unlimitedRecipes: return "infinity"
        case .advancedFilters: return "slider.horizontal.3"
        case .mealPlanning: return "calendar"
        case .nutritionTracking: return "chart.pie"
        case .chefConsultation: return "person.2"
        case .liveClasses: return "video"
        case .exclusiveRecipes: return "lock.open"
        }
    }
    
    var description: String {
        switch self {
        case .removeAds: return "모든 광고를 제거하고 깔끔한 환경에서 요리하세요"
        case .unlimitedRecipes: return "20,000개 이상의 레시피에 무제한 접근"
        case .advancedFilters: return "칼로리, 영양소, 조리시간 등 세밀한 필터링"
        case .mealPlanning: return "주간 식단 자동 생성 및 장보기 리스트"
        case .nutritionTracking: return "일일 영양소 섭취량 자동 계산 및 추적"
        case .chefConsultation: return "전문 셰프와 1:1 화상 상담 (월 2회)"
        case .liveClasses: return "매주 진행되는 라이브 쿠킹 클래스"
        case .exclusiveRecipes: return "미슐랭 셰프들의 시그니처 레시피"
        }
    }
}

// MARK: - Store Error
enum StoreError: Error {
    case failedVerification
    case productNotFound
    case purchaseFailed
    case restoreFailed
}

// MARK: - Paywall View
struct PaywallView: View {
    @StateObject private var store = StoreManager.shared
    @State private var selectedProduct: Product?
    @State private var isPurchasing = false
    @State private var showingError = false
    @State private var errorMessage = ""
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        NavigationStack {
            ScrollView {
                VStack(spacing: 24) {
                    // Header
                    VStack(spacing: 16) {
                        Image(systemName: "crown.fill")
                            .font(.system(size: 60))
                            .foregroundStyle(
                                LinearGradient(
                                    colors: [.yellow, .orange],
                                    startPoint: .topLeading,
                                    endPoint: .bottomTrailing
                                )
                            )
                        
                        Text("RecipeBox Pro로 업그레이드")
                            .font(.largeTitle)
                            .bold()
                        
                        Text("모든 프리미엄 기능을 이용하고\n요리의 달인이 되어보세요")
                            .font(.body)
                            .multilineTextAlignment(.center)
                            .foregroundColor(.secondary)
                    }
                    .padding(.top, 20)
                    
                    // Features List
                    VStack(alignment: .leading, spacing: 16) {
                        ForEach(PremiumFeature.allCases, id: \.self) { feature in
                            FeatureRow(feature: feature, isIncluded: true)
                        }
                    }
                    .padding(.horizontal)
                    
                    // Subscription Plans
                    VStack(spacing: 16) {
                        Text("구독 플랜 선택")
                            .font(.headline)
                        
                        ForEach(store.products.filter { $0.type == .autoRenewable }, id: \.id) { product in
                            SubscriptionPlanCard(
                                product: product,
                                isSelected: selectedProduct == product
                            ) {
                                selectedProduct = product
                            }
                        }
                    }
                    .padding(.horizontal)
                    
                    // Purchase Button
                    Button {
                        Task {
                            await purchaseSelected()
                        }
                    } label: {
                        ZStack {
                            if isPurchasing {
                                ProgressView()
                                    .progressViewStyle(CircularProgressViewStyle(tint: .white))
                            } else {
                                Text(purchaseButtonText)
                                    .font(.headline)
                            }
                        }
                        .foregroundColor(.white)
                        .frame(maxWidth: .infinity)
                        .frame(height: 56)
                        .background(
                            selectedProduct != nil ? Color.blue : Color.gray
                        )
                        .cornerRadius(16)
                    }
                    .disabled(selectedProduct == nil || isPurchasing)
                    .padding(.horizontal)
                    
                    // Terms & Restore
                    VStack(spacing: 12) {
                        Button {
                            Task {
                                await store.restorePurchases()
                            }
                        } label: {
                            Text("이전 구매 복원")
                                .font(.subheadline)
                                .foregroundColor(.blue)
                        }
                        
                        HStack(spacing: 16) {
                            Link("이용약관", destination: URL(string: "https://recipebox.app/terms")!)
                            Link("개인정보처리방침", destination: URL(string: "https://recipebox.app/privacy")!)
                        }
                        .font(.caption)
                        .foregroundColor(.secondary)
                        
                        Text("구독은 현재 기간이 끝나기 최소 24시간 전에 취소하지 않으면 자동으로 갱신됩니다.")
                            .font(.caption2)
                            .foregroundColor(.secondary)
                            .multilineTextAlignment(.center)
                            .padding(.horizontal)
                    }
                    .padding(.bottom, 20)
                }
            }
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("닫기") {
                        dismiss()
                    }
                }
            }
            .alert("구매 실패", isPresented: $showingError) {
                Button("확인", role: .cancel) { }
            } message: {
                Text(errorMessage)
            }
        }
    }
    
    var purchaseButtonText: String {
        if let product = selectedProduct {
            return "지금 시작하기 - \(product.displayPrice)"
        }
        return "플랜을 선택하세요"
    }
    
    func purchaseSelected() async {
        guard let product = selectedProduct else { return }
        
        isPurchasing = true
        
        do {
            let transaction = try await store.purchase(product)
            
            if transaction != nil {
                // Purchase successful
                dismiss()
            }
        } catch {
            errorMessage = "구매 처리 중 오류가 발생했습니다"
            showingError = true
        }
        
        isPurchasing = false
    }
}

// MARK: - Subscription Management View
struct SubscriptionManagementView: View {
    @StateObject private var store = StoreManager.shared
    @State private var showingCancelConfirmation = false
    
    var body: some View {
        NavigationStack {
            Form {
                // Current Subscription
                Section("현재 구독") {
                    HStack {
                        Image(systemName: store.subscriptionLevel.icon)
                            .font(.title2)
                            .foregroundColor(store.subscriptionLevel.color)
                        
                        VStack(alignment: .leading) {
                            Text(store.subscriptionLevel.rawValue)
                                .font(.headline)
                            
                            if let status = store.subscriptionStatus {
                                Text(subscriptionStatusText(status))
                                    .font(.caption)
                                    .foregroundColor(.secondary)
                            }
                        }
                        
                        Spacer()
                    }
                    .padding(.vertical, 8)
                    
                    if store.isSubscribed {
                        // Renewal Date
                        if let renewalDate = store.subscriptionStatus?.renewalInfo?.renewalDate {
                            LabeledContent("다음 갱신일") {
                                Text(renewalDate, format: .dateTime.year().month().day())
                            }
                        }
                        
                        // Auto-Renewal Status
                        if let willAutoRenew = store.subscriptionStatus?.renewalInfo?.willAutoRenew {
                            LabeledContent("자동 갱신") {
                                Text(willAutoRenew ? "켜짐" : "꺼짐")
                                    .foregroundColor(willAutoRenew ? .green : .orange)
                            }
                        }
                    }
                }
                
                // Features
                Section("포함된 기능") {
                    ForEach(store.subscriptionLevel.features, id: \.self) { feature in
                        FeatureRow(feature: feature, isIncluded: true)
                    }
                }
                
                // Actions
                if store.isSubscribed {
                    Section {
                        Button {
                            Task {
                                await store.openSubscriptionManagement()
                            }
                        } label: {
                            Label("구독 관리", systemImage: "creditcard")
                        }
                        
                        Button {
                            showingCancelConfirmation = true
                        } label: {
                            Label("구독 취소", systemImage: "xmark.circle")
                                .foregroundColor(.red)
                        }
                    }
                }
                
                // Other Purchases
                Section("추가 구매") {
                    NavigationLink(destination: RecipePacksView()) {
                        Label("레시피 팩", systemImage: "book.pages")
                    }
                    
                    NavigationLink(destination: MasterClassesView()) {
                        Label("마스터 클래스", systemImage: "video")
                    }
                    
                    NavigationLink(destination: CreditsView()) {
                        Label("크레딧 구매", systemImage: "bitcoinsign.circle")
                    }
                }
            }
            .navigationTitle("구독 관리")
            .alert("구독 취소", isPresented: $showingCancelConfirmation) {
                Button("취소", role: .cancel) { }
                Button("구독 취소", role: .destructive) {
                    Task {
                        await store.openSubscriptionManagement()
                    }
                }
            } message: {
                Text("구독을 취소하시면 현재 구독 기간이 끝날 때까지 프리미엄 기능을 사용할 수 있습니다.")
            }
        }
    }
    
    func subscriptionStatusText(_ status: Product.SubscriptionInfo.Status) -> String {
        switch status.state {
        case .subscribed:
            return "활성"
        case .expired:
            return "만료됨"
        case .inBillingRetryPeriod:
            return "결제 재시도 중"
        case .inGracePeriod:
            return "유예 기간"
        case .revoked:
            return "취소됨"
        default:
            return "알 수 없음"
        }
    }
}

// MARK: - Recipe Packs View
struct RecipePacksView: View {
    @StateObject private var store = StoreManager.shared
    @State private var purchasedPacks: Set<String> = []
    
    let recipePacks = [
        RecipePack(
            id: ProductIdentifier.recipePack_korean.rawValue,
            title: "한식 마스터 팩",
            description: "전통 한식부터 퓨전 한식까지 300개 레시피",
            imageNmae: "korean_pack",
            recipeCount: 300,
            originalPrice: "₩15,000"
        ),
        RecipePack(
            id: ProductIdentifier.recipePack_italian.rawValue,
            title: "이탈리안 컬렉션",
            description: "정통 파스타와 피자 레시피 250개",
            imageNmae: "italian_pack",
            recipeCount: 250,
            originalPrice: "₩12,000"
        ),
        RecipePack(
            id: ProductIdentifier.recipePack_dessert.rawValue,
            title: "디저트 천국",
            description: "케이크, 쿠키, 마카롱 등 디저트 레시피 200개",
            imageNmae: "dessert_pack",
            recipeCount: 200,
            originalPrice: "₩10,000"
        )
    ]
    
    var body: some View {
        ScrollView {
            VStack(spacing: 20) {
                ForEach(recipePacks) { pack in
                    RecipePackCard(
                        pack: pack,
                        product: store.products.first { $0.id == pack.id },
                        isPurchased: purchasedPacks.contains(pack.id),
                        onPurchase: {
                            await purchasePack(pack)
                        }
                    )
                }
            }
            .padding()
        }
        .navigationTitle("레시피 팩")
        .onAppear {
            loadPurchasedPacks()
        }
    }
    
    func loadPurchasedPacks() {
        for pack in recipePacks {
            if UserDefaults.standard.bool(forKey: "pack_\(pack.id)") {
                purchasedPacks.insert(pack.id)
            }
        }
    }
    
    func purchasePack(_ pack: RecipePack) async {
        guard let product = store.products.first(where: { $0.id == pack.id }) else { return }
        
        do {
            let transaction = try await store.purchase(product)
            if transaction != nil {
                purchasedPacks.insert(pack.id)
            }
        } catch {
            print("Failed to purchase pack: \(error)")
        }
    }
}

// MARK: - Master Classes View
struct MasterClassesView: View {
    @StateObject private var store = StoreManager.shared
    @State private var purchasedClasses: Set<String> = []
    
    let masterClasses = [
        MasterClass(
            id: ProductIdentifier.masterClass_basics.rawValue,
            title: "요리 기초 마스터",
            instructor: "김민수 셰프",
            duration: "총 12시간",
            lessons: 24,
            description: "칼질부터 소스 만들기까지 요리의 모든 기초"
        ),
        MasterClass(
            id: ProductIdentifier.masterClass_advanced.rawValue,
            title: "미슐랭 요리 비법",
            instructor: "박지훈 셰프",
            duration: "총 20시간",
            lessons: 40,
            description: "미슐랭 레스토랑의 요리 기법과 플레이팅"
        )
    ]
    
    var body: some View {
        ScrollView {
            VStack(spacing: 20) {
                ForEach(masterClasses) { masterClass in
                    MasterClassCard(
                        masterClass: masterClass,
                        product: store.products.first { $0.id == masterClass.id },
                        isPurchased: purchasedClasses.contains(masterClass.id),
                        onPurchase: {
                            await purchaseClass(masterClass)
                        }
                    )
                }
            }
            .padding()
        }
        .navigationTitle("마스터 클래스")
        .onAppear {
            loadPurchasedClasses()
        }
    }
    
    func loadPurchasedClasses() {
        for masterClass in masterClasses {
            if UserDefaults.standard.bool(forKey: "class_\(masterClass.id)") {
                purchasedClasses.insert(masterClass.id)
            }
        }
    }
    
    func purchaseClass(_ masterClass: MasterClass) async {
        guard let product = store.products.first(where: { $0.id == masterClass.id }) else { return }
        
        do {
            let transaction = try await store.purchase(product)
            if transaction != nil {
                purchasedClasses.insert(masterClass.id)
            }
        } catch {
            print("Failed to purchase class: \(error)")
        }
    }
}

// MARK: - Credits View
struct CreditsView: View {
    @StateObject private var store = StoreManager.shared
    @AppStorage("user_credits") private var userCredits: Int = 0
    @State private var isPurchasing = false
    @State private var selectedProduct: Product?
    
    let creditPackages = [
        CreditPackage(
            id: ProductIdentifier.credits_100.rawValue,
            credits: 100,
            bonus: 0,
            popular: false
        ),
        CreditPackage(
            id: ProductIdentifier.credits_500.rawValue,
            credits: 500,
            bonus: 50,
            popular: true
        ),
        CreditPackage(
            id: ProductIdentifier.credits_1000.rawValue,
            credits: 1000,
            bonus: 200,
            popular: false
        )
    ]
    
    var body: some View {
        ScrollView {
            VStack(spacing: 24) {
                // Current Credits
                VStack(spacing: 8) {
                    Text("보유 크레딧")
                        .font(.subheadline)
                        .foregroundColor(.secondary)
                    
                    Text("\(userCredits)")
                        .font(.system(size: 48, weight: .bold))
                        .foregroundStyle(
                            LinearGradient(
                                colors: [.yellow, .orange],
                                startPoint: .leading,
                                endPoint: .trailing
                            )
                        )
                    
                    Text("크레딧으로 프리미엄 레시피와 클래스를 구매하세요")
                        .font(.caption)
                        .foregroundColor(.secondary)
                        .multilineTextAlignment(.center)
                }
                .padding(.vertical, 20)
                
                // Credit Packages
                VStack(spacing: 16) {
                    ForEach(creditPackages) { package in
                        CreditPackageCard(
                            package: package,
                            product: store.products.first { $0.id == package.id },
                            isSelected: selectedProduct?.id == package.id,
                            onSelect: { product in
                                selectedProduct = product
                            }
                        )
                    }
                }
                .padding(.horizontal)
                
                // Purchase Button
                Button {
                    Task {
                        await purchaseCredits()
                    }
                } label: {
                    ZStack {
                        if isPurchasing {
                            ProgressView()
                                .progressViewStyle(CircularProgressViewStyle(tint: .white))
                        } else {
                            Text(selectedProduct != nil ? "크레딧 구매하기" : "패키지를 선택하세요")
                                .font(.headline)
                        }
                    }
                    .foregroundColor(.white)
                    .frame(maxWidth: .infinity)
                    .frame(height: 56)
                    .background(selectedProduct != nil ? Color.orange : Color.gray)
                    .cornerRadius(16)
                }
                .disabled(selectedProduct == nil || isPurchasing)
                .padding(.horizontal)
                
                // Usage Info
                VStack(alignment: .leading, spacing: 12) {
                    Text("크레딧 사용처")
                        .font(.headline)
                    
                    CreditUsageRow(icon: "book", title: "프리미엄 레시피", credits: 10)
                    CreditUsageRow(icon: "video", title: "라이브 클래스 참여", credits: 50)
                    CreditUsageRow(icon: "person.2", title: "1:1 셰프 상담", credits: 100)
                    CreditUsageRow(icon: "gift", title: "선물하기", credits: 5)
                }
                .padding()
                .background(Color(.systemGray6))
                .cornerRadius(16)
                .padding(.horizontal)
            }
        }
        .navigationTitle("크레딧")
    }
    
    func purchaseCredits() async {
        guard let product = selectedProduct else { return }
        
        isPurchasing = true
        
        do {
            let transaction = try await store.purchase(product)
            if transaction != nil {
                // Credits are automatically added in StoreManager
                selectedProduct = nil
            }
        } catch {
            print("Failed to purchase credits: \(error)")
        }
        
        isPurchasing = false
    }
}

// MARK: - Ad Manager
class AdManager: NSObject {
    static let shared = AdManager()
    
    private override init() {
        super.init()
    }
    
    func showBannerAd(in view: UIView) {
        guard !StoreManager.shared.hasAccess(to: .removeAds) else { return }
        
        // Show banner ad
        // Implementation with AdMob or other ad network
    }
    
    func showInterstitialAd(from viewController: UIViewController) {
        guard !StoreManager.shared.hasAccess(to: .removeAds) else { return }
        
        // Show interstitial ad
        // Implementation with AdMob or other ad network
    }
    
    func showRewardedAd(from viewController: UIViewController, completion: @escaping (Bool) -> Void) {
        // Show rewarded ad
        // Implementation with AdMob or other ad network
        completion(true) // Call when ad is watched
    }
}

// MARK: - Supporting Views
struct FeatureRow: View {
    let feature: PremiumFeature
    let isIncluded: Bool
    
    var body: some View {
        HStack(spacing: 12) {
            Image(systemName: isIncluded ? "checkmark.circle.fill" : "xmark.circle")
                .foregroundColor(isIncluded ? .green : .red)
            
            VStack(alignment: .leading, spacing: 2) {
                Text(feature.rawValue)
                    .font(.subheadline)
                    .fontWeight(.medium)
                
                Text(feature.description)
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            
            Spacer()
        }
    }
}

struct SubscriptionPlanCard: View {
    let product: Product
    let isSelected: Bool
    let onSelect: () -> Void
    
    var isYearly: Bool {
        product.id.contains("yearly")
    }
    
    var savings: String? {
        if isYearly {
            return "17% 할인"
        }
        return nil
    }
    
    var body: some View {
        Button(action: onSelect) {
            VStack(spacing: 8) {
                HStack {
                    VStack(alignment: .leading, spacing: 4) {
                        Text(product.displayName)
                            .font(.headline)
                        
                        Text(product.description)
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }
                    
                    Spacer()
                    
                    VStack(alignment: .trailing, spacing: 2) {
                        Text(product.displayPrice)
                            .font(.headline)
                        
                        if let savings = savings {
                            Text(savings)
                                .font(.caption)
                                .padding(.horizontal, 6)
                                .padding(.vertical, 2)
                                .background(Color.green)
                                .foregroundColor(.white)
                                .cornerRadius(4)
                        }
                    }
                }
                
                if isYearly {
                    Text("매월 \(monthlyPrice(for: product))")
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
            }
            .padding()
            .background(isSelected ? Color.blue.opacity(0.1) : Color(.systemGray6))
            .overlay(
                RoundedRectangle(cornerRadius: 12)
                    .stroke(isSelected ? Color.blue : Color.clear, lineWidth: 2)
            )
            .cornerRadius(12)
        }
        .buttonStyle(PlainButtonStyle())
    }
    
    func monthlyPrice(for product: Product) -> String {
        let monthly = product.price / 12
        return monthly.formatted(.currency(code: "KRW"))
    }
}

struct RecipePackCard: View {
    let pack: RecipePack
    let product: Product?
    let isPurchased: Bool
    let onPurchase: () async -> Void
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            // Image
            RoundedRectangle(cornerRadius: 12)
                .fill(LinearGradient(
                    colors: [.blue, .purple],
                    startPoint: .topLeading,
                    endPoint: .bottomTrailing
                ))
                .frame(height: 150)
                .overlay(
                    VStack {
                        Text(pack.title)
                            .font(.title2)
                            .bold()
                            .foregroundColor(.white)
                        
                        Text("\(pack.recipeCount)개 레시피")
                            .font(.subheadline)
                            .foregroundColor(.white.opacity(0.9))
                    }
                )
            
            Text(pack.description)
                .font(.body)
            
            HStack {
                if isPurchased {
                    Label("구매 완료", systemImage: "checkmark.circle.fill")
                        .foregroundColor(.green)
                } else {
                    VStack(alignment: .leading, spacing: 2) {
                        if let product = product {
                            Text(product.displayPrice)
                                .font(.headline)
                        }
                        
                        Text(pack.originalPrice)
                            .font(.caption)
                            .strikethrough()
                            .foregroundColor(.secondary)
                    }
                    
                    Spacer()
                    
                    Button {
                        Task {
                            await onPurchase()
                        }
                    } label: {
                        Text("구매하기")
                            .font(.subheadline)
                            .bold()
                            .padding(.horizontal, 20)
                            .padding(.vertical, 8)
                            .background(Color.blue)
                            .foregroundColor(.white)
                            .cornerRadius(20)
                    }
                }
            }
        }
        .padding()
        .background(Color(.systemGray6))
        .cornerRadius(16)
    }
}

struct MasterClassCard: View {
    let masterClass: MasterClass
    let product: Product?
    let isPurchased: Bool
    let onPurchase: () async -> Void
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            HStack {
                // Instructor Image
                Circle()
                    .fill(Color(.systemGray5))
                    .frame(width: 60, height: 60)
                    .overlay(
                        Image(systemName: "person.fill")
                            .foregroundColor(.secondary)
                    )
                
                VStack(alignment: .leading, spacing: 4) {
                    Text(masterClass.title)
                        .font(.headline)
                    
                    Text(masterClass.instructor)
                        .font(.subheadline)
                        .foregroundColor(.secondary)
                }
                
                Spacer()
            }
            
            Text(masterClass.description)
                .font(.body)
            
            HStack(spacing: 20) {
                Label("\(masterClass.lessons)개 강의", systemImage: "play.circle")
                Label(masterClass.duration, systemImage: "clock")
            }
            .font(.caption)
            .foregroundColor(.secondary)
            
            HStack {
                if isPurchased {
                    Label("수강 가능", systemImage: "checkmark.circle.fill")
                        .foregroundColor(.green)
                } else if let product = product {
                    Text(product.displayPrice)
                        .font(.headline)
                    
                    Spacer()
                    
                    Button {
                        Task {
                            await onPurchase()
                        }
                    } label: {
                        Text("수강 신청")
                            .font(.subheadline)
                            .bold()
                            .padding(.horizontal, 20)
                            .padding(.vertical, 8)
                            .background(Color.blue)
                            .foregroundColor(.white)
                            .cornerRadius(20)
                    }
                }
            }
        }
        .padding()
        .background(Color(.systemGray6))
        .cornerRadius(16)
    }
}

struct CreditPackageCard: View {
    let package: CreditPackage
    let product: Product?
    let isSelected: Bool
    let onSelect: (Product) -> Void
    
    var totalCredits: Int {
        package.credits + package.bonus
    }
    
    var body: some View {
        Button {
            if let product = product {
                onSelect(product)
            }
        } label: {
            VStack(spacing: 12) {
                if package.popular {
                    Text("인기")
                        .font(.caption)
                        .bold()
                        .padding(.horizontal, 12)
                        .padding(.vertical, 4)
                        .background(Color.orange)
                        .foregroundColor(.white)
                        .cornerRadius(12)
                }
                
                HStack {
                    VStack(alignment: .leading, spacing: 4) {
                        HStack(alignment: .firstTextBaseline, spacing: 4) {
                            Text("\(totalCredits)")
                                .font(.title)
                                .bold()
                            
                            Text("크레딧")
                                .font(.subheadline)
                            
                            if package.bonus > 0 {
                                Text("+\(package.bonus) 보너스")
                                    .font(.caption)
                                    .padding(.horizontal, 6)
                                    .padding(.vertical, 2)
                                    .background(Color.green)
                                    .foregroundColor(.white)
                                    .cornerRadius(4)
                            }
                        }
                        
                        if let product = product {
                            Text(product.displayPrice)
                                .font(.headline)
                                .foregroundColor(.blue)
                        }
                    }
                    
                    Spacer()
                    
                    Image(systemName: "bitcoinsign.circle.fill")
                        .font(.system(size: 40))
                        .foregroundColor(.orange)
                }
            }
            .padding()
            .background(isSelected ? Color.orange.opacity(0.1) : Color(.systemGray6))
            .overlay(
                RoundedRectangle(cornerRadius: 12)
                    .stroke(isSelected ? Color.orange : Color.clear, lineWidth: 2)
            )
            .cornerRadius(12)
        }
        .buttonStyle(PlainButtonStyle())
    }
}

struct CreditUsageRow: View {
    let icon: String
    let title: String
    let credits: Int
    
    var body: some View {
        HStack {
            Image(systemName: icon)
                .foregroundColor(.orange)
            
            Text(title)
                .font(.subheadline)
            
            Spacer()
            
            Text("\(credits) 크레딧")
                .font(.caption)
                .foregroundColor(.secondary)
        }
    }
}

// MARK: - Supporting Models
struct RecipePack: Identifiable {
    let id: String
    let title: String
    let description: String
    let imageNmae: String
    let recipeCount: Int
    let originalPrice: String
}

struct MasterClass: Identifiable {
    let id: String
    let title: String
    let instructor: String
    let duration: String
    let lessons: Int
    let description: String
}

struct CreditPackage: Identifiable {
    let id: String
    let credits: Int
    let bonus: Int
    let popular: Bool
}

// MARK: - Helper Managers
class RecipePackManager {
    static let shared = RecipePackManager()
    
    func downloadPack(_ packId: String) async {
        // Download recipe pack content
    }
}

class MasterClassManager {
    static let shared = MasterClassManager()
    
    func downloadClass(_ classId: String) async {
        // Download class videos
    }
}

// MARK: - Notifications
extension Notification.Name {
    static let adsRemoved = Notification.Name("adsRemoved")
    static let subscriptionUpdated = Notification.Name("subscriptionUpdated")
}
```