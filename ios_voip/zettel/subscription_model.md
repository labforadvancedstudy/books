# 구독 모델 및 수익화

## 개념
VoIP 앱의 지속가능한 수익 창출을 위한 구독, 프리미엄, 광고 등 다양한 수익화 전략.

## 구독 플랜 구조
```swift
import StoreKit

enum SubscriptionTier: String, CaseIterable {
    case free = "com.voipapp.free"
    case basic = "com.voipapp.basic.monthly"
    case premium = "com.voipapp.premium.monthly"
    case business = "com.voipapp.business.monthly"
    
    var features: Set<Feature> {
        switch self {
        case .free:
            return [.basicCalling, .limitedMinutes]
        case .basic:
            return [.basicCalling, .unlimitedDomestic, .callRecording]
        case .premium:
            return [.basicCalling, .unlimitedInternational, .callRecording,
                   .videoCall, .groupCall, .customRingtones]
        case .business:
            return [.allFeatures, .prioritySupport, .analytics, .apiAccess]
        }
    }
    
    var monthlyPrice: Decimal {
        switch self {
        case .free: return 0
        case .basic: return 4.99
        case .premium: return 9.99
        case .business: return 29.99
        }
    }
    
    var freeMinutes: Int {
        switch self {
        case .free: return 100
        case .basic: return 1000
        case .premium, .business: return .max
        }
    }
}
```

## In-App Purchase 구현
```swift
class IAPManager: NSObject, SKPaymentTransactionObserver {
    static let shared = IAPManager()
    private var products: [SKProduct] = []
    private var purchaseCompletion: ((Bool, Error?) -> Void)?
    
    func loadProducts() {
        let productIds = Set(SubscriptionTier.allCases.map { $0.rawValue })
        let request = SKProductsRequest(productIdentifiers: productIds)
        request.delegate = self
        request.start()
    }
    
    func purchase(_ tier: SubscriptionTier, completion: @escaping (Bool, Error?) -> Void) {
        guard let product = products.first(where: { $0.productIdentifier == tier.rawValue }) else {
            completion(false, IAPError.productNotFound)
            return
        }
        
        purchaseCompletion = completion
        
        let payment = SKPayment(product: product)
        SKPaymentQueue.default().add(payment)
        
        // 분석 이벤트
        Analytics.logEvent("subscription_initiated", parameters: [
            "tier": tier.rawValue,
            "price": product.price.doubleValue,
            "currency": product.priceLocale.currencyCode ?? "USD"
        ])
    }
    
    func restorePurchases() {
        SKPaymentQueue.default().restoreCompletedTransactions()
    }
}
```

## 영수증 검증
```swift
class ReceiptValidator {
    static func validateReceipt(completion: @escaping (SubscriptionStatus?) -> Void) {
        guard let receiptURL = Bundle.main.appStoreReceiptURL,
              let receiptData = try? Data(contentsOf: receiptURL) else {
            completion(nil)
            return
        }
        
        let receiptString = receiptData.base64EncodedString()
        
        // 서버 검증 (Apple 권장)
        validateOnServer(receipt: receiptString) { result in
            switch result {
            case .success(let status):
                self.cacheSubscriptionStatus(status)
                completion(status)
            case .failure:
                // 로컬 캠시 사용
                completion(self.getCachedStatus())
            }
        }
    }
    
    private static func validateOnServer(
        receipt: String,
        completion: @escaping (Result<SubscriptionStatus, Error>) -> Void
    ) {
        let url = URL(string: "https://api.voipapp.com/validate-receipt")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.httpBody = try? JSONEncoder().encode(["receipt": receipt])
        
        URLSession.shared.dataTask(with: request) { data, _, error in
            if let data = data,
               let status = try? JSONDecoder().decode(SubscriptionStatus.self, from: data) {
                completion(.success(status))
            } else {
                completion(.failure(error ?? ValidationError.unknown))
            }
        }.resume()
    }
}
```

## 프리미엄 기능 게이트
```swift
class PremiumFeatureGate {
    static func canUseFeature(_ feature: Feature) -> Bool {
        let currentTier = SubscriptionManager.currentTier
        return currentTier.features.contains(feature)
    }
    
    static func requestFeature(
        _ feature: Feature,
        from viewController: UIViewController,
        completion: @escaping (Bool) -> Void
    ) {
        if canUseFeature(feature) {
            completion(true)
            return
        }
        
        // 업그레이드 프롬프트
        let requiredTier = getMinimumTierForFeature(feature)
        showUpgradePrompt(
            for: requiredTier,
            feature: feature,
            from: viewController
        ) { upgraded in
            completion(upgraded)
        }
    }
    
    private static func showUpgradePrompt(
        for tier: SubscriptionTier,
        feature: Feature,
        from viewController: UIViewController,
        completion: @escaping (Bool) -> Void
    ) {
        let alert = UIAlertController(
            title: "Premium Feature",
            message: "\(feature.description) requires \(tier) subscription.",
            preferredStyle: .alert
        )
        
        alert.addAction(UIAlertAction(title: "Upgrade", style: .default) { _ in
            PurchaseViewController.present(tier: tier, from: viewController) { success in
                completion(success)
            }
        })
        
        alert.addAction(UIAlertAction(title: "Cancel", style: .cancel) { _ in
            completion(false)
        })
        
        viewController.present(alert, animated: true)
    }
}
```

## 광고 통합 (Freemium)
```swift
import GoogleMobileAds

class AdManager: NSObject, GADFullScreenContentDelegate {
    static let shared = AdManager()
    private var interstitialAd: GADInterstitialAd?
    private var rewardedAd: GADRewardedAd?
    
    func loadAds() {
        // 구독자는 광고 비활성화
        guard !SubscriptionManager.isPremium else { return }
        
        loadInterstitialAd()
        loadRewardedAd()
    }
    
    func showInterstitialAfterCall(from viewController: UIViewController) {
        // 무료 사용자에게만 표시
        guard !SubscriptionManager.isPremium,
              let ad = interstitialAd else { return }
        
        ad.fullScreenContentDelegate = self
        ad.present(fromRootViewController: viewController)
        
        Analytics.logEvent("ad_shown", parameters: ["type": "interstitial"])
    }
    
    func showRewardedAdForMinutes(
        from viewController: UIViewController,
        completion: @escaping (Int) -> Void
    ) {
        guard let ad = rewardedAd else {
            completion(0)
            return
        }
        
        ad.present(fromRootViewController: viewController) {
            let reward = ad.adReward
            let freeMinutes = reward.amount.intValue
            
            // 무료 통화 시간 추가
            UserDefaults.standard.set(
                UserDefaults.standard.integer(forKey: "freeMinutes") + freeMinutes,
                forKey: "freeMinutes"
            )
            
            completion(freeMinutes)
            
            Analytics.logEvent("rewarded_ad_completed", parameters: [
                "minutes_earned": freeMinutes
            ])
        }
    }
}
```

## 사용량 기반 과금
```swift
class UsageBasedBilling {
    struct UsageRate {
        let tierName: String
        let includedMinutes: Int
        let overageRate: Decimal  // per minute
        let dataIncludedGB: Double
        let dataOverageRate: Decimal  // per GB
    }
    
    static func calculateMonthlyBill(usage: MonthlyUsage, plan: UsageRate) -> Decimal {
        var total: Decimal = 0
        
        // 초과 통화 시간
        if usage.totalMinutes > plan.includedMinutes {
            let overageMinutes = usage.totalMinutes - plan.includedMinutes
            total += Decimal(overageMinutes) * plan.overageRate
        }
        
        // 초과 데이터
        if usage.totalDataGB > plan.dataIncludedGB {
            let overageGB = Decimal(usage.totalDataGB - plan.dataIncludedGB)
            total += overageGB * plan.dataOverageRate
        }
        
        return total
    }
}
```

## 연관 개념
- [[call_billing]]
- [[analytics_integration]]
- [[ab_testing]]

## 태그
#monetization #subscription #iap #freemium #revenue