# 통화 요금 계산

## 개념
VoIP 앱에서 국제전화, 로밍, 프리미엄 기능 등에 대한 요금 계산 및 과금 시스템.

## 요금 모델
```swift
struct RateCard {
    let countryCode: String
    let countryName: String
    let ratePerMinute: Decimal  // USD 기준
    let connectionFee: Decimal
    let minimumDuration: Int    // 초 단위
    let billingIncrement: Int    // 초 단위
    
    // 예시 요금표
    static let rates = [
        RateCard(
            countryCode: "+1",
            countryName: "USA/Canada",
            ratePerMinute: 0.01,
            connectionFee: 0,
            minimumDuration: 6,
            billingIncrement: 6
        ),
        RateCard(
            countryCode: "+82",
            countryName: "South Korea",
            ratePerMinute: 0.02,
            connectionFee: 0.01,
            minimumDuration: 30,
            billingIncrement: 6
        ),
        RateCard(
            countryCode: "+44",
            countryName: "UK",
            ratePerMinute: 0.015,
            connectionFee: 0,
            minimumDuration: 60,
            billingIncrement: 60
        )
    ]
}
```

## 요금 계산 엔진
```swift
class BillingCalculator {
    func calculateCallCost(
        duration: TimeInterval,
        destinationNumber: String,
        userPlan: SubscriptionPlan
    ) -> Decimal {
        // 국가 코드 추출
        let countryCode = extractCountryCode(from: destinationNumber)
        guard let rate = RateCard.rates.first(where: { $0.countryCode == countryCode }) else {
            return 0
        }
        
        // 무료 통화 시간 확인
        if userPlan.hasUnlimitedCalls(to: countryCode) {
            return 0
        }
        
        let freeMinutes = userPlan.freeMinutesRemaining(for: countryCode)
        let chargeableDuration = max(0, duration - freeMinutes * 60)
        
        if chargeableDuration == 0 {
            return 0
        }
        
        // 최소 통화 시간 적용
        let billableDuration = max(Double(rate.minimumDuration), chargeableDuration)
        
        // 과금 단위로 올림
        let billableIncrements = ceil(billableDuration / Double(rate.billingIncrement))
        let billableSeconds = billableIncrements * Double(rate.billingIncrement)
        
        // 요금 계산
        let minuteRate = rate.ratePerMinute
        let callCost = (Decimal(billableSeconds) / 60) * minuteRate + rate.connectionFee
        
        // 할인 적용
        let discount = userPlan.discountRate
        return callCost * (1 - discount)
    }
}
```

## 사용량 추적
```swift
class UsageTracker {
    private let database: Database
    
    struct CallRecord {
        let id: UUID
        let startTime: Date
        let endTime: Date
        let duration: TimeInterval
        let destinationNumber: String
        let cost: Decimal
        let billingStatus: BillingStatus
    }
    
    enum BillingStatus {
        case pending
        case billed
        case failed
        case refunded
    }
    
    func trackCall(_ call: CallRecord) async {
        // 데이터베이스에 저장
        await database.save(call)
        
        // 실시간 잔액 업데이트
        await updateBalance(cost: call.cost)
        
        // 임계값 경고
        await checkBalanceThreshold()
    }
    
    func getMonthlyUsage() async -> UsageSummary {
        let startOfMonth = Calendar.current.dateInterval(of: .month, for: Date())?.start ?? Date()
        let calls = await database.fetchCalls(since: startOfMonth)
        
        return UsageSummary(
            totalCalls: calls.count,
            totalDuration: calls.reduce(0) { $0 + $1.duration },
            totalCost: calls.reduce(0) { $0 + $1.cost },
            averageCallDuration: calls.isEmpty ? 0 : calls.reduce(0) { $0 + $1.duration } / Double(calls.count)
        )
    }
}
```

## 선불/후불 관리
```swift
class BalanceManager {
    private var currentBalance: Decimal = 0
    private let minimumBalance: Decimal = 1.0
    
    enum PaymentMethod {
        case prepaid
        case postpaid
        case hybrid  // 선불 + 초과시 후불
    }
    
    func canMakeCall(to destination: String, method: PaymentMethod) -> Bool {
        switch method {
        case .prepaid:
            // 최소 잔액 확인
            return currentBalance >= minimumBalance
            
        case .postpaid:
            // 신용 한도 확인
            return checkCreditLimit()
            
        case .hybrid:
            // 선불 잔액 또는 신용 한도
            return currentBalance > 0 || checkCreditLimit()
        }
    }
    
    func rechargeBalance(amount: Decimal) async throws {
        // 결제 처리
        let payment = try await processPayment(amount: amount)
        
        // 잔액 업데이트
        currentBalance += amount
        
        // 트랜잭션 기록
        await recordTransaction(payment)
    }
}
```

## 실시간 요금 표시
```swift
class RealTimeBillingView: UIView {
    @IBOutlet weak var durationLabel: UILabel!
    @IBOutlet weak var costLabel: UILabel!
    @IBOutlet weak var rateLabel: UILabel!
    
    private var timer: Timer?
    private var callStartTime: Date?
    private var ratePerSecond: Decimal = 0
    
    func startBilling(for destination: String) {
        callStartTime = Date()
        ratePerSecond = calculateRate(for: destination) / 60
        
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { _ in
            self.updateBilling()
        }
    }
    
    private func updateBilling() {
        guard let startTime = callStartTime else { return }
        
        let duration = Date().timeIntervalSince(startTime)
        let cost = Decimal(duration) * ratePerSecond
        
        durationLabel.text = formatDuration(duration)
        costLabel.text = formatCurrency(cost)
        
        // 잔액 부족 경고
        if cost > currentBalance - 1.0 {
            showLowBalanceWarning()
        }
    }
}
```

## 연관 개념
- [[usage_tracking]]
- [[analytics_integration]]
- [[subscription_model]]
- [[payment_integration]]

## 태그
#billing #payment #monetization #usage #tracking