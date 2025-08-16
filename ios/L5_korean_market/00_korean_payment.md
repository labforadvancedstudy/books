# 한국 결제 시스템 완벽 통합

## 토스페이먼츠 SDK 통합

```swift
import UIKit
import TossPayments

// MARK: - TossPayments Manager
class TossPaymentsManager: NSObject {
    static let shared = TossPaymentsManager()
    
    private let clientKey = "test_ck_D5GePWvyJnrK0W0k6q8gLzN97Eoq" // 실제 앱에서는 안전하게 관리
    private let customerKey = UUID().uuidString
    
    private var paymentWidget: PaymentWidget?
    private var agreementWidget: AgreementWidget?
    
    override init() {
        super.init()
    }
    
    // MARK: - 결제 위젯 초기화
    func initializePaymentWidget(in viewController: UIViewController) async throws {
        let paymentWidget = try await PaymentWidget(
            clientKey: clientKey,
            customerKey: customerKey
        )
        
        self.paymentWidget = paymentWidget
        
        // 결제 수단 위젯 렌더링
        try await paymentWidget.renderPaymentMethods(
            method: paymentWidget.paymentMethodWidget,
            amount: PaymentAmount(value: 50000),
            options: PaymentMethodOptions(
                variantKey: "DEFAULT"
            )
        )
        
        // 이용약관 위젯 렌더링
        let agreementWidget = try await paymentWidget.renderAgreement(
            method: paymentWidget.agreementWidget,
            options: AgreementOptions(
                variantKey: "DEFAULT"
            )
        )
        
        self.agreementWidget = agreementWidget
    }
    
    // MARK: - 결제 요청
    func requestPayment(
        orderId: String,
        orderName: String,
        amount: Int,
        customerName: String,
        customerEmail: String,
        customerMobilePhone: String,
        completion: @escaping (Result<PaymentResult, Error>) -> Void
    ) {
        guard let paymentWidget = paymentWidget else {
            completion(.failure(PaymentError.widgetNotInitialized))
            return
        }
        
        Task {
            do {
                // 결제 요청
                let result = try await paymentWidget.requestPayment(
                    orderId: orderId,
                    orderName: orderName,
                    successUrl: "recipebox://payment-success",
                    failUrl: "recipebox://payment-fail"
                )
                
                // 서버에 결제 승인 요청
                let paymentResult = try await confirmPayment(
                    paymentKey: result.paymentKey,
                    orderId: orderId,
                    amount: amount
                )
                
                completion(.success(paymentResult))
                
            } catch {
                completion(.failure(error))
            }
        }
    }
    
    // MARK: - 결제 승인
    private func confirmPayment(
        paymentKey: String,
        orderId: String,
        amount: Int
    ) async throws -> PaymentResult {
        let url = URL(string: "https://api.tosspayments.com/v1/payments/confirm")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("Basic \(base64EncodedSecretKey())", forHTTPHeaderField: "Authorization")
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        let body: [String: Any] = [
            "paymentKey": paymentKey,
            "orderId": orderId,
            "amount": amount
        ]
        
        request.httpBody = try JSONSerialization.data(withJSONObject: body)
        
        let (data, response) = try await URLSession.shared.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse,
              httpResponse.statusCode == 200 else {
            throw PaymentError.confirmationFailed
        }
        
        let paymentResult = try JSONDecoder().decode(PaymentResult.self, from: data)
        return paymentResult
    }
    
    // MARK: - 정기 결제 빌링키 발급
    func issueBillingKey(
        customerKey: String,
        cardNumber: String,
        cardExpirationYear: String,
        cardExpirationMonth: String,
        cardPassword: String,
        birthOrBusinessRegistrationNumber: String,
        completion: @escaping (Result<BillingKey, Error>) -> Void
    ) {
        Task {
            do {
                let url = URL(string: "https://api.tosspayments.com/v1/billing/authorizations/card")!
                var request = URLRequest(url: url)
                request.httpMethod = "POST"
                request.setValue("Basic \(base64EncodedSecretKey())", forHTTPHeaderField: "Authorization")
                request.setValue("application/json", forHTTPHeaderField: "Content-Type")
                
                let body: [String: Any] = [
                    "customerKey": customerKey,
                    "cardNumber": cardNumber,
                    "cardExpirationYear": cardExpirationYear,
                    "cardExpirationMonth": cardExpirationMonth,
                    "cardPassword": cardPassword,
                    "customerIdentityNumber": birthOrBusinessRegistrationNumber
                ]
                
                request.httpBody = try JSONSerialization.data(withJSONObject: body)
                
                let (data, response) = try await URLSession.shared.data(for: request)
                
                guard let httpResponse = response as? HTTPURLResponse,
                      httpResponse.statusCode == 200 else {
                    throw PaymentError.billingKeyIssueFailed
                }
                
                let billingKey = try JSONDecoder().decode(BillingKey.self, from: data)
                completion(.success(billingKey))
                
            } catch {
                completion(.failure(error))
            }
        }
    }
    
    // MARK: - 빌링키로 자동 결제
    func requestBillingPayment(
        billingKey: String,
        customerKey: String,
        amount: Int,
        orderId: String,
        orderName: String,
        completion: @escaping (Result<PaymentResult, Error>) -> Void
    ) {
        Task {
            do {
                let url = URL(string: "https://api.tosspayments.com/v1/billing/\(billingKey)")!
                var request = URLRequest(url: url)
                request.httpMethod = "POST"
                request.setValue("Basic \(base64EncodedSecretKey())", forHTTPHeaderField: "Authorization")
                request.setValue("application/json", forHTTPHeaderField: "Content-Type")
                
                let body: [String: Any] = [
                    "customerKey": customerKey,
                    "amount": amount,
                    "orderId": orderId,
                    "orderName": orderName
                ]
                
                request.httpBody = try JSONSerialization.data(withJSONObject: body)
                
                let (data, response) = try await URLSession.shared.data(for: request)
                
                guard let httpResponse = response as? HTTPURLResponse,
                      httpResponse.statusCode == 200 else {
                    throw PaymentError.billingPaymentFailed
                }
                
                let paymentResult = try JSONDecoder().decode(PaymentResult.self, from: data)
                completion(.success(paymentResult))
                
            } catch {
                completion(.failure(error))
            }
        }
    }
    
    // MARK: - 결제 취소
    func cancelPayment(
        paymentKey: String,
        cancelReason: String,
        cancelAmount: Int? = nil,
        completion: @escaping (Result<CancelResult, Error>) -> Void
    ) {
        Task {
            do {
                let url = URL(string: "https://api.tosspayments.com/v1/payments/\(paymentKey)/cancel")!
                var request = URLRequest(url: url)
                request.httpMethod = "POST"
                request.setValue("Basic \(base64EncodedSecretKey())", forHTTPHeaderField: "Authorization")
                request.setValue("application/json", forHTTPHeaderField: "Content-Type")
                
                var body: [String: Any] = [
                    "cancelReason": cancelReason
                ]
                
                if let cancelAmount = cancelAmount {
                    body["cancelAmount"] = cancelAmount
                }
                
                request.httpBody = try JSONSerialization.data(withJSONObject: body)
                
                let (data, response) = try await URLSession.shared.data(for: request)
                
                guard let httpResponse = response as? HTTPURLResponse,
                      httpResponse.statusCode == 200 else {
                    throw PaymentError.cancellationFailed
                }
                
                let cancelResult = try JSONDecoder().decode(CancelResult.self, from: data)
                completion(.success(cancelResult))
                
            } catch {
                completion(.failure(error))
            }
        }
    }
    
    private func base64EncodedSecretKey() -> String {
        let secretKey = "test_sk_zXLkKEypNArWmo50nX3lmeaxYG5R:" // 실제 앱에서는 안전하게 관리
        return Data(secretKey.utf8).base64EncodedString()
    }
}

// MARK: - 카카오페이 통합
class KakaoPayManager: NSObject {
    static let shared = KakaoPayManager()
    
    private let adminKey = "YOUR_ADMIN_KEY" // 실제 앱에서는 안전하게 관리
    private let cid = "TC0ONETIME" // 테스트용 CID
    
    private var tid: String? // 결제 고유 번호
    private var redirectURL: String?
    
    // MARK: - 결제 준비
    func preparePayment(
        itemName: String,
        quantity: Int,
        totalAmount: Int,
        taxFreeAmount: Int = 0,
        completion: @escaping (Result<KakaoPayReadyResponse, Error>) -> Void
    ) {
        let url = URL(string: "https://kapi.kakao.com/v1/payment/ready")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("KakaoAK \(adminKey)", forHTTPHeaderField: "Authorization")
        request.setValue("application/x-www-form-urlencoded;charset=utf-8", forHTTPHeaderField: "Content-Type")
        
        let partnerOrderId = UUID().uuidString
        let partnerUserId = UserDefaults.standard.string(forKey: "userId") ?? "guest"
        
        let parameters: [String: Any] = [
            "cid": cid,
            "partner_order_id": partnerOrderId,
            "partner_user_id": partnerUserId,
            "item_name": itemName,
            "quantity": quantity,
            "total_amount": totalAmount,
            "tax_free_amount": taxFreeAmount,
            "approval_url": "recipebox://payment/kakao/success",
            "cancel_url": "recipebox://payment/kakao/cancel",
            "fail_url": "recipebox://payment/kakao/fail"
        ]
        
        request.httpBody = parameters
            .map { "\($0.key)=\($0.value)" }
            .joined(separator: "&")
            .data(using: .utf8)
        
        Task {
            do {
                let (data, response) = try await URLSession.shared.data(for: request)
                
                guard let httpResponse = response as? HTTPURLResponse,
                      httpResponse.statusCode == 200 else {
                    throw PaymentError.preparationFailed
                }
                
                let readyResponse = try JSONDecoder().decode(KakaoPayReadyResponse.self, from: data)
                self.tid = readyResponse.tid
                self.redirectURL = readyResponse.nextRedirectMobileUrl
                
                completion(.success(readyResponse))
                
            } catch {
                completion(.failure(error))
            }
        }
    }
    
    // MARK: - 결제 승인
    func approvePayment(
        pgToken: String,
        completion: @escaping (Result<KakaoPayApprovalResponse, Error>) -> Void
    ) {
        guard let tid = tid else {
            completion(.failure(PaymentError.tidNotFound))
            return
        }
        
        let url = URL(string: "https://kapi.kakao.com/v1/payment/approve")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("KakaoAK \(adminKey)", forHTTPHeaderField: "Authorization")
        request.setValue("application/x-www-form-urlencoded;charset=utf-8", forHTTPHeaderField: "Content-Type")
        
        let partnerOrderId = UserDefaults.standard.string(forKey: "currentOrderId") ?? ""
        let partnerUserId = UserDefaults.standard.string(forKey: "userId") ?? "guest"
        
        let parameters: [String: Any] = [
            "cid": cid,
            "tid": tid,
            "partner_order_id": partnerOrderId,
            "partner_user_id": partnerUserId,
            "pg_token": pgToken
        ]
        
        request.httpBody = parameters
            .map { "\($0.key)=\($0.value)" }
            .joined(separator: "&")
            .data(using: .utf8)
        
        Task {
            do {
                let (data, response) = try await URLSession.shared.data(for: request)
                
                guard let httpResponse = response as? HTTPURLResponse,
                      httpResponse.statusCode == 200 else {
                    throw PaymentError.approvalFailed
                }
                
                let approvalResponse = try JSONDecoder().decode(KakaoPayApprovalResponse.self, from: data)
                completion(.success(approvalResponse))
                
            } catch {
                completion(.failure(error))
            }
        }
    }
    
    // MARK: - 정기 결제 등록
    func registerSubscription(
        itemName: String,
        completion: @escaping (Result<KakaoPaySubscriptionResponse, Error>) -> Void
    ) {
        let url = URL(string: "https://kapi.kakao.com/v1/payment/subscription")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("KakaoAK \(adminKey)", forHTTPHeaderField: "Authorization")
        request.setValue("application/x-www-form-urlencoded;charset=utf-8", forHTTPHeaderField: "Content-Type")
        
        let partnerUserId = UserDefaults.standard.string(forKey: "userId") ?? "guest"
        
        let parameters: [String: Any] = [
            "cid": "TCSUBSCRIP", // 정기결제 CID
            "partner_order_id": UUID().uuidString,
            "partner_user_id": partnerUserId,
            "item_name": itemName,
            "quantity": 1,
            "total_amount": 0, // 최초 등록시 0원
            "tax_free_amount": 0,
            "approval_url": "recipebox://subscription/kakao/success",
            "cancel_url": "recipebox://subscription/kakao/cancel",
            "fail_url": "recipebox://subscription/kakao/fail"
        ]
        
        request.httpBody = parameters
            .map { "\($0.key)=\($0.value)" }
            .joined(separator: "&")
            .data(using: .utf8)
        
        Task {
            do {
                let (data, response) = try await URLSession.shared.data(for: request)
                
                guard let httpResponse = response as? HTTPURLResponse,
                      httpResponse.statusCode == 200 else {
                    throw PaymentError.subscriptionRegistrationFailed
                }
                
                let subscriptionResponse = try JSONDecoder().decode(KakaoPaySubscriptionResponse.self, from: data)
                
                // SID 저장
                UserDefaults.standard.set(subscriptionResponse.sid, forKey: "kakaoPaySID")
                
                completion(.success(subscriptionResponse))
                
            } catch {
                completion(.failure(error))
            }
        }
    }
    
    // MARK: - 결제 취소
    func cancelPayment(
        tid: String,
        cancelAmount: Int,
        cancelTaxFreeAmount: Int = 0,
        completion: @escaping (Result<KakaoPayCancelResponse, Error>) -> Void
    ) {
        let url = URL(string: "https://kapi.kakao.com/v1/payment/cancel")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("KakaoAK \(adminKey)", forHTTPHeaderField: "Authorization")
        request.setValue("application/x-www-form-urlencoded;charset=utf-8", forHTTPHeaderField: "Content-Type")
        
        let parameters: [String: Any] = [
            "cid": cid,
            "tid": tid,
            "cancel_amount": cancelAmount,
            "cancel_tax_free_amount": cancelTaxFreeAmount
        ]
        
        request.httpBody = parameters
            .map { "\($0.key)=\($0.value)" }
            .joined(separator: "&")
            .data(using: .utf8)
        
        Task {
            do {
                let (data, response) = try await URLSession.shared.data(for: request)
                
                guard let httpResponse = response as? HTTPURLResponse,
                      httpResponse.statusCode == 200 else {
                    throw PaymentError.cancellationFailed
                }
                
                let cancelResponse = try JSONDecoder().decode(KakaoPayCancelResponse.self, from: data)
                completion(.success(cancelResponse))
                
            } catch {
                completion(.failure(error))
            }
        }
    }
}

// MARK: - 네이버페이 통합
class NaverPayManager: NSObject {
    static let shared = NaverPayManager()
    
    private let clientId = "YOUR_CLIENT_ID"
    private let clientSecret = "YOUR_CLIENT_SECRET"
    private let chainId = "YOUR_CHAIN_ID"
    
    // MARK: - 결제 창 호출
    func requestPayment(
        merchantPayKey: String,
        productName: String,
        totalPayAmount: Int,
        taxScopeAmount: Int,
        taxExScopeAmount: Int,
        returnUrl: String,
        completion: @escaping (Result<NaverPayResponse, Error>) -> Void
    ) {
        let url = URL(string: "https://dev.apis.naver.com/naverpay-partner/naverpay/payments/v2.2/apply/payment")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue(clientId, forHTTPHeaderField: "X-Naver-Client-Id")
        request.setValue(clientSecret, forHTTPHeaderField: "X-Naver-Client-Secret")
        request.setValue("application/x-www-form-urlencoded;charset=utf-8", forHTTPHeaderField: "Content-Type")
        
        let modelVersion = "2.2"
        let merchantUserKey = UserDefaults.standard.string(forKey: "userId") ?? "guest"
        
        let productItems: [[String: Any]] = [[
            "categoryType": "FOOD",
            "categoryId": "FOOD",
            "uid": UUID().uuidString,
            "name": productName,
            "payReferrer": "RECIPE_BOX",
            "count": 1
        ]]
        
        let parameters: [String: Any] = [
            "modelVersion": modelVersion,
            "merchantPayKey": merchantPayKey,
            "merchantUserKey": merchantUserKey,
            "productName": productName,
            "totalPayAmount": totalPayAmount,
            "taxScopeAmount": taxScopeAmount,
            "taxExScopeAmount": taxExScopeAmount,
            "returnUrl": returnUrl,
            "productItems": productItems
        ]
        
        do {
            request.httpBody = try JSONSerialization.data(withJSONObject: parameters)
        } catch {
            completion(.failure(error))
            return
        }
        
        Task {
            do {
                let (data, response) = try await URLSession.shared.data(for: request)
                
                guard let httpResponse = response as? HTTPURLResponse,
                      httpResponse.statusCode == 200 else {
                    throw PaymentError.requestFailed
                }
                
                let naverPayResponse = try JSONDecoder().decode(NaverPayResponse.self, from: data)
                completion(.success(naverPayResponse))
                
            } catch {
                completion(.failure(error))
            }
        }
    }
    
    // MARK: - 결제 승인
    func approvePayment(
        paymentId: String,
        completion: @escaping (Result<NaverPayApprovalResponse, Error>) -> Void
    ) {
        let url = URL(string: "https://dev.apis.naver.com/naverpay-partner/naverpay/payments/v2.2/\(paymentId)")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue(clientId, forHTTPHeaderField: "X-Naver-Client-Id")
        request.setValue(clientSecret, forHTTPHeaderField: "X-Naver-Client-Secret")
        
        Task {
            do {
                let (data, response) = try await URLSession.shared.data(for: request)
                
                guard let httpResponse = response as? HTTPURLResponse,
                      httpResponse.statusCode == 200 else {
                    throw PaymentError.approvalFailed
                }
                
                let approvalResponse = try JSONDecoder().decode(NaverPayApprovalResponse.self, from: data)
                completion(.success(approvalResponse))
                
            } catch {
                completion(.failure(error))
            }
        }
    }
    
    // MARK: - 결제 취소
    func cancelPayment(
        paymentId: String,
        cancelAmount: Int,
        cancelReason: String,
        cancelRequester: String = "2", // 1: 구매자, 2: 가맹점 관리자
        completion: @escaping (Result<NaverPayCancelResponse, Error>) -> Void
    ) {
        let url = URL(string: "https://dev.apis.naver.com/naverpay-partner/naverpay/payments/v1/\(paymentId)/cancel")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue(clientId, forHTTPHeaderField: "X-Naver-Client-Id")
        request.setValue(clientSecret, forHTTPHeaderField: "X-Naver-Client-Secret")
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        let parameters: [String: Any] = [
            "cancelAmount": cancelAmount,
            "cancelReason": cancelReason,
            "cancelRequester": cancelRequester
        ]
        
        do {
            request.httpBody = try JSONSerialization.data(withJSONObject: parameters)
        } catch {
            completion(.failure(error))
            return
        }
        
        Task {
            do {
                let (data, response) = try await URLSession.shared.data(for: request)
                
                guard let httpResponse = response as? HTTPURLResponse,
                      httpResponse.statusCode == 200 else {
                    throw PaymentError.cancellationFailed
                }
                
                let cancelResponse = try JSONDecoder().decode(NaverPayCancelResponse.self, from: data)
                completion(.success(cancelResponse))
                
            } catch {
                completion(.failure(error))
            }
        }
    }
}

// MARK: - NICE/이니시스 PG 통합
class NicePayManager: NSObject {
    static let shared = NicePayManager()
    
    private let merchantID = "nicepay00m"
    private let merchantKey = "EYzu8jGGMfqaDEp76gSckuvnaHHu+bC4opsSN6lHv3b2lurNYkVXrZ7Z1AoqQnXI3eLuaUFyoRNC6FkrzVjceg=="
    
    // MARK: - 결제 요청
    func requestPayment(
        orderId: String,
        amount: Int,
        goodsName: String,
        buyerName: String,
        buyerTel: String,
        buyerEmail: String,
        returnURL: String,
        completion: @escaping (Result<NicePayResponse, Error>) -> Void
    ) {
        let ediDate = DateFormatter.nicepayFormat.string(from: Date())
        let hashData = "\(ediDate)\(merchantID)\(amount)\(merchantKey)"
        let signData = hashData.sha256()
        
        let parameters: [String: Any] = [
            "PayMethod": "CARD",
            "GoodsName": goodsName,
            "Amt": amount,
            "MID": merchantID,
            "Moid": orderId,
            "BuyerName": buyerName,
            "BuyerEmail": buyerEmail,
            "BuyerTel": buyerTel,
            "ReturnURL": returnURL,
            "CharSet": "utf-8",
            "SignData": signData,
            "EdiDate": ediDate
        ]
        
        let url = URL(string: "https://webapi.nicepay.co.kr/webapi/pay_process.jsp")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/x-www-form-urlencoded", forHTTPHeaderField: "Content-Type")
        
        request.httpBody = parameters
            .map { "\($0.key)=\($0.value)" }
            .joined(separator: "&")
            .data(using: .utf8)
        
        Task {
            do {
                let (data, response) = try await URLSession.shared.data(for: request)
                
                guard let httpResponse = response as? HTTPURLResponse,
                      httpResponse.statusCode == 200 else {
                    throw PaymentError.requestFailed
                }
                
                let nicePayResponse = try JSONDecoder().decode(NicePayResponse.self, from: data)
                completion(.success(nicePayResponse))
                
            } catch {
                completion(.failure(error))
            }
        }
    }
    
    // MARK: - 결제 취소
    func cancelPayment(
        tid: String,
        cancelAmount: Int,
        cancelMessage: String,
        partialCancelCode: String = "0", // 0: 전체취소, 1: 부분취소
        completion: @escaping (Result<NicePayCancelResponse, Error>) -> Void
    ) {
        let ediDate = DateFormatter.nicepayFormat.string(from: Date())
        let hashData = "\(merchantID)\(cancelAmount)\(ediDate)\(merchantKey)"
        let signData = hashData.sha256()
        
        let parameters: [String: Any] = [
            "TID": tid,
            "MID": merchantID,
            "CancelAmt": cancelAmount,
            "CancelMsg": cancelMessage,
            "PartialCancelCode": partialCancelCode,
            "EdiDate": ediDate,
            "SignData": signData,
            "CharSet": "utf-8"
        ]
        
        let url = URL(string: "https://webapi.nicepay.co.kr/webapi/cancel_process.jsp")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/x-www-form-urlencoded", forHTTPHeaderField: "Content-Type")
        
        request.httpBody = parameters
            .map { "\($0.key)=\($0.value)" }
            .joined(separator: "&")
            .data(using: .utf8)
        
        Task {
            do {
                let (data, response) = try await URLSession.shared.data(for: request)
                
                guard let httpResponse = response as? HTTPURLResponse,
                      httpResponse.statusCode == 200 else {
                    throw PaymentError.cancellationFailed
                }
                
                let cancelResponse = try JSONDecoder().decode(NicePayCancelResponse.self, from: data)
                completion(.success(cancelResponse))
                
            } catch {
                completion(.failure(error))
            }
        }
    }
}

// MARK: - 세금계산서 발행
class TaxInvoiceManager {
    static let shared = TaxInvoiceManager()
    
    private let apiKey = "YOUR_POPBILL_API_KEY"
    private let apiSecret = "YOUR_POPBILL_API_SECRET"
    
    // MARK: - 세금계산서 발행
    func issueTaxInvoice(
        businessNumber: String,
        invoiceInfo: TaxInvoiceInfo,
        completion: @escaping (Result<TaxInvoiceResponse, Error>) -> Void
    ) {
        let url = URL(string: "https://popbill.com/api/taxinvoice")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("Bearer \(getAccessToken())", forHTTPHeaderField: "Authorization")
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        let invoice = TaxInvoice(
            writeDate: DateFormatter.yyyyMMdd.string(from: Date()),
            chargeDirection: "정과금",
            issueType: "정발행",
            purposeType: "영수",
            taxType: "과세",
            invoicerCorpNum: businessNumber,
            invoicerMgtKey: UUID().uuidString,
            invoicerCorpName: invoiceInfo.corpName,
            invoicerCEOName: invoiceInfo.ceoName,
            invoicerAddr: invoiceInfo.address,
            invoicerBizType: invoiceInfo.bizType,
            invoicerBizClass: invoiceInfo.bizClass,
            invoicerContactName: invoiceInfo.contactName,
            invoicerEmail: invoiceInfo.email,
            invoicerTEL: invoiceInfo.tel,
            invoiceeType: "사업자",
            invoiceeCorpNum: invoiceInfo.invoiceeCorpNum,
            invoiceeCorpName: invoiceInfo.invoiceeCorpName,
            invoiceeCEOName: invoiceInfo.invoiceeCEOName,
            invoiceeAddr: invoiceInfo.invoiceeAddr,
            invoiceeBizType: invoiceInfo.invoiceeBizType,
            invoiceeBizClass: invoiceInfo.invoiceeBizClass,
            invoiceeContactName1: invoiceInfo.invoiceeContactName,
            invoiceeEmail1: invoiceInfo.invoiceeEmail,
            invoiceeTEL1: invoiceInfo.invoiceeTEL,
            supplyCostTotal: invoiceInfo.supplyCostTotal,
            taxTotal: invoiceInfo.taxTotal,
            totalAmount: invoiceInfo.totalAmount,
            remark1: invoiceInfo.remark ?? "",
            detailList: invoiceInfo.detailList
        )
        
        do {
            request.httpBody = try JSONEncoder().encode(invoice)
        } catch {
            completion(.failure(error))
            return
        }
        
        Task {
            do {
                let (data, response) = try await URLSession.shared.data(for: request)
                
                guard let httpResponse = response as? HTTPURLResponse,
                      httpResponse.statusCode == 200 else {
                    throw PaymentError.taxInvoiceFailed
                }
                
                let invoiceResponse = try JSONDecoder().decode(TaxInvoiceResponse.self, from: data)
                completion(.success(invoiceResponse))
                
            } catch {
                completion(.failure(error))
            }
        }
    }
    
    // MARK: - 현금영수증 발행
    func issueCashReceipt(
        identityNum: String, // 휴대폰번호, 카드번호, 사업자번호
        amount: Int,
        tax: Int,
        serviceFee: Int = 0,
        completion: @escaping (Result<CashReceiptResponse, Error>) -> Void
    ) {
        let url = URL(string: "https://api.barobill.co.kr/CashReceipt/RegistCashReceipt")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        let receipt = CashReceipt(
            mgtKey: UUID().uuidString,
            tradeType: "승인거래",
            tradeUsage: "소득공제용",
            identityNum: identityNum,
            amount: amount,
            tax: tax,
            serviceFee: serviceFee,
            totalAmount: amount + tax + serviceFee,
            franchiseCorpNum: "1234567890",
            franchiseCorpName: "RecipeBox",
            franchiseCEOName: "대표자",
            franchiseAddr: "서울특별시 강남구",
            franchiseTEL: "02-1234-5678"
        )
        
        do {
            request.httpBody = try JSONEncoder().encode(receipt)
        } catch {
            completion(.failure(error))
            return
        }
        
        Task {
            do {
                let (data, response) = try await URLSession.shared.data(for: request)
                
                guard let httpResponse = response as? HTTPURLResponse,
                      httpResponse.statusCode == 200 else {
                    throw PaymentError.cashReceiptFailed
                }
                
                let receiptResponse = try JSONDecoder().decode(CashReceiptResponse.self, from: data)
                completion(.success(receiptResponse))
                
            } catch {
                completion(.failure(error))
            }
        }
    }
    
    private func getAccessToken() -> String {
        // 실제 구현에서는 OAuth 토큰 관리
        return "YOUR_ACCESS_TOKEN"
    }
}

// MARK: - 통합 결제 뷰
import SwiftUI

struct KoreanPaymentView: View {
    @State private var selectedPaymentMethod: PaymentMethod = .tossPay
    @State private var amount: Int = 10000
    @State private var isProcessing = false
    @State private var showingResult = false
    @State private var paymentResult: PaymentResultType?
    
    enum PaymentMethod: String, CaseIterable {
        case tossPay = "토스페이"
        case kakaoPay = "카카오페이"
        case naverPay = "네이버페이"
        case creditCard = "신용카드"
        case bankTransfer = "계좌이체"
        case virtualAccount = "가상계좌"
        
        var icon: String {
            switch self {
            case .tossPay: return "toss_logo"
            case .kakaoPay: return "kakao_logo"
            case .naverPay: return "naver_logo"
            case .creditCard: return "creditcard"
            case .bankTransfer: return "building.columns"
            case .virtualAccount: return "banknote"
            }
        }
        
        var color: Color {
            switch self {
            case .tossPay: return .blue
            case .kakaoPay: return .yellow
            case .naverPay: return .green
            case .creditCard: return .purple
            case .bankTransfer: return .orange
            case .virtualAccount: return .gray
            }
        }
    }
    
    var body: some View {
        NavigationStack {
            ScrollView {
                VStack(spacing: 24) {
                    // 결제 금액
                    VStack(alignment: .leading, spacing: 12) {
                        Text("결제 금액")
                            .font(.headline)
                        
                        Text("₩\(amount.formatted())")
                            .font(.largeTitle)
                            .bold()
                    }
                    .frame(maxWidth: .infinity, alignment: .leading)
                    .padding()
                    .background(Color(.systemGray6))
                    .cornerRadius(12)
                    
                    // 결제 수단 선택
                    VStack(alignment: .leading, spacing: 16) {
                        Text("결제 수단 선택")
                            .font(.headline)
                        
                        ForEach(PaymentMethod.allCases, id: \.self) { method in
                            PaymentMethodButton(
                                method: method,
                                isSelected: selectedPaymentMethod == method
                            ) {
                                selectedPaymentMethod = method
                            }
                        }
                    }
                    
                    // 결제 정보
                    VStack(alignment: .leading, spacing: 12) {
                        Text("결제 정보")
                            .font(.headline)
                        
                        LabeledContent("상품명", value: "RecipeBox 프리미엄")
                        LabeledContent("판매자", value: "RecipeBox Inc.")
                        
                        Divider()
                        
                        Toggle("세금계산서 발행", isOn: .constant(false))
                        Toggle("현금영수증 발행", isOn: .constant(false))
                    }
                    .padding()
                    .background(Color(.systemGray6))
                    .cornerRadius(12)
                    
                    // 약관 동의
                    VStack(alignment: .leading, spacing: 12) {
                        HStack {
                            Image(systemName: "checkmark.square.fill")
                                .foregroundColor(.blue)
                            Text("전체 동의")
                                .font(.headline)
                        }
                        
                        VStack(alignment: .leading, spacing: 8) {
                            AgreeementRow(text: "구매조건 확인 및 결제진행 동의")
                            AgreeementRow(text: "개인정보 제3자 제공 동의")
                            AgreeementRow(text: "전자결제대행 이용 동의")
                        }
                        .padding(.leading, 28)
                    }
                    
                    // 결제 버튼
                    Button {
                        processPayment()
                    } label: {
                        ZStack {
                            if isProcessing {
                                ProgressView()
                                    .progressViewStyle(CircularProgressViewStyle(tint: .white))
                            } else {
                                Text("₩\(amount.formatted()) 결제하기")
                                    .font(.headline)
                            }
                        }
                        .foregroundColor(.white)
                        .frame(maxWidth: .infinity)
                        .frame(height: 56)
                        .background(selectedPaymentMethod.color)
                        .cornerRadius(12)
                    }
                    .disabled(isProcessing)
                }
                .padding()
            }
            .navigationTitle("결제")
            .navigationBarTitleDisplayMode(.inline)
            .sheet(isPresented: $showingResult) {
                PaymentResultView(result: paymentResult)
            }
        }
    }
    
    func processPayment() {
        isProcessing = true
        
        Task {
            do {
                switch selectedPaymentMethod {
                case .tossPay:
                    try await processTossPayment()
                case .kakaoPay:
                    try await processKakaoPayment()
                case .naverPay:
                    try await processNaverPayment()
                case .creditCard:
                    try await processCreditCardPayment()
                case .bankTransfer:
                    try await processBankTransfer()
                case .virtualAccount:
                    try await processVirtualAccount()
                }
                
                paymentResult = .success
                showingResult = true
                
            } catch {
                paymentResult = .failure(error.localizedDescription)
                showingResult = true
            }
            
            isProcessing = false
        }
    }
    
    func processTossPayment() async throws {
        // 토스페이 결제 처리
    }
    
    func processKakaoPayment() async throws {
        // 카카오페이 결제 처리
    }
    
    func processNaverPayment() async throws {
        // 네이버페이 결제 처리
    }
    
    func processCreditCardPayment() async throws {
        // 신용카드 결제 처리
    }
    
    func processBankTransfer() async throws {
        // 계좌이체 처리
    }
    
    func processVirtualAccount() async throws {
        // 가상계좌 처리
    }
}

// MARK: - Supporting Views
struct PaymentMethodButton: View {
    let method: KoreanPaymentView.PaymentMethod
    let isSelected: Bool
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            HStack {
                Image(systemName: method.icon)
                    .font(.title2)
                    .foregroundColor(method.color)
                    .frame(width: 30)
                
                Text(method.rawValue)
                    .font(.subheadline)
                    .fontWeight(.medium)
                
                Spacer()
                
                Image(systemName: isSelected ? "checkmark.circle.fill" : "circle")
                    .foregroundColor(isSelected ? .blue : .gray)
            }
            .padding()
            .background(isSelected ? Color.blue.opacity(0.1) : Color(.systemGray6))
            .cornerRadius(12)
        }
        .buttonStyle(PlainButtonStyle())
    }
}

struct AgreeementRow: View {
    let text: String
    
    var body: some View {
        HStack {
            Image(systemName: "checkmark.square")
                .font(.caption)
                .foregroundColor(.gray)
            
            Text(text)
                .font(.caption)
                .foregroundColor(.secondary)
        }
    }
}

struct PaymentResultView: View {
    let result: PaymentResultType?
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        VStack(spacing: 24) {
            if result == .success {
                Image(systemName: "checkmark.circle.fill")
                    .font(.system(size: 80))
                    .foregroundColor(.green)
                
                Text("결제 완료")
                    .font(.largeTitle)
                    .bold()
                
                Text("결제가 성공적으로 완료되었습니다")
                    .font(.body)
                    .foregroundColor(.secondary)
            } else {
                Image(systemName: "xmark.circle.fill")
                    .font(.system(size: 80))
                    .foregroundColor(.red)
                
                Text("결제 실패")
                    .font(.largeTitle)
                    .bold()
                
                if case .failure(let message) = result {
                    Text(message)
                        .font(.body)
                        .foregroundColor(.secondary)
                        .multilineTextAlignment(.center)
                }
            }
            
            Button {
                dismiss()
            } label: {
                Text("확인")
                    .font(.headline)
                    .foregroundColor(.white)
                    .frame(maxWidth: .infinity)
                    .frame(height: 56)
                    .background(Color.blue)
                    .cornerRadius(12)
            }
            .padding(.top)
        }
        .padding()
    }
}

// MARK: - Models
enum PaymentResultType {
    case success
    case failure(String)
}

enum PaymentError: Error {
    case widgetNotInitialized
    case confirmationFailed
    case billingKeyIssueFailed
    case billingPaymentFailed
    case cancellationFailed
    case preparationFailed
    case approvalFailed
    case tidNotFound
    case subscriptionRegistrationFailed
    case requestFailed
    case taxInvoiceFailed
    case cashReceiptFailed
}

// MARK: - Response Models
struct PaymentResult: Codable {
    let paymentKey: String
    let orderId: String
    let orderName: String
    let amount: Int
    let status: String
    let approvedAt: String?
}

struct BillingKey: Codable {
    let billingKey: String
    let customerKey: String
}

struct CancelResult: Codable {
    let cancelAmount: Int
    let cancelReason: String
    let canceledAt: String
}

struct KakaoPayReadyResponse: Codable {
    let tid: String
    let nextRedirectAppUrl: String?
    let nextRedirectMobileUrl: String?
    let nextRedirectPcUrl: String?
    let androidAppScheme: String?
    let iosAppScheme: String?
    let createdAt: String
}

struct KakaoPayApprovalResponse: Codable {
    let aid: String
    let tid: String
    let cid: String
    let partnerOrderId: String
    let partnerUserId: String
    let paymentMethodType: String
    let amount: Amount
    let approvedAt: String
    
    struct Amount: Codable {
        let total: Int
        let taxFree: Int
        let vat: Int
        let point: Int
        let discount: Int
    }
}

struct KakaoPaySubscriptionResponse: Codable {
    let sid: String
    let status: String
}

struct KakaoPayCancelResponse: Codable {
    let aid: String
    let tid: String
    let status: String
    let canceledAt: String
}

struct NaverPayResponse: Codable {
    let code: String
    let message: String
    let body: PaymentBody?
    
    struct PaymentBody: Codable {
        let paymentId: String
        let detail: String
    }
}

struct NaverPayApprovalResponse: Codable {
    let code: String
    let message: String
    let body: ApprovalBody?
    
    struct ApprovalBody: Codable {
        let paymentId: String
        let detail: PaymentDetail
        
        struct PaymentDetail: Codable {
            let paymentId: String
            let payHistId: String
            let merchantPayKey: String
            let totalPayAmount: Int
        }
    }
}

struct NaverPayCancelResponse: Codable {
    let code: String
    let message: String
}

struct NicePayResponse: Codable {
    let resultCode: String
    let resultMsg: String
    let tid: String?
    let authDate: String?
    let authCode: String?
}

struct NicePayCancelResponse: Codable {
    let resultCode: String
    let resultMsg: String
    let cancelDate: String?
    let cancelTime: String?
}

struct TaxInvoice: Codable {
    let writeDate: String
    let chargeDirection: String
    let issueType: String
    let purposeType: String
    let taxType: String
    let invoicerCorpNum: String
    let invoicerMgtKey: String
    let invoicerCorpName: String
    let invoicerCEOName: String
    let invoicerAddr: String
    let invoicerBizType: String
    let invoicerBizClass: String
    let invoicerContactName: String
    let invoicerEmail: String
    let invoicerTEL: String
    let invoiceeType: String
    let invoiceeCorpNum: String
    let invoiceeCorpName: String
    let invoiceeCEOName: String
    let invoiceeAddr: String
    let invoiceeBizType: String
    let invoiceeBizClass: String
    let invoiceeContactName1: String
    let invoiceeEmail1: String
    let invoiceeTEL1: String
    let supplyCostTotal: Int
    let taxTotal: Int
    let totalAmount: Int
    let remark1: String
    let detailList: [TaxInvoiceDetail]
}

struct TaxInvoiceDetail: Codable {
    let serialNum: Int
    let purchaseDT: String
    let itemName: String
    let spec: String?
    let qty: Int
    let unitCost: Int
    let supplyCost: Int
    let tax: Int
}

struct TaxInvoiceInfo {
    let corpName: String
    let ceoName: String
    let address: String
    let bizType: String
    let bizClass: String
    let contactName: String
    let email: String
    let tel: String
    let invoiceeCorpNum: String
    let invoiceeCorpName: String
    let invoiceeCEOName: String
    let invoiceeAddr: String
    let invoiceeBizType: String
    let invoiceeBizClass: String
    let invoiceeContactName: String
    let invoiceeEmail: String
    let invoiceeTEL: String
    let supplyCostTotal: Int
    let taxTotal: Int
    let totalAmount: Int
    let remark: String?
    let detailList: [TaxInvoiceDetail]
}

struct TaxInvoiceResponse: Codable {
    let code: Int
    let message: String
    let ntsConfirmNum: String?
}

struct CashReceipt: Codable {
    let mgtKey: String
    let tradeType: String
    let tradeUsage: String
    let identityNum: String
    let amount: Int
    let tax: Int
    let serviceFee: Int
    let totalAmount: Int
    let franchiseCorpNum: String
    let franchiseCorpName: String
    let franchiseCEOName: String
    let franchiseAddr: String
    let franchiseTEL: String
}

struct CashReceiptResponse: Codable {
    let code: Int
    let message: String
    let confirmNum: String?
}

// MARK: - Extensions
extension String {
    func sha256() -> String {
        // SHA256 해시 구현
        return self // 실제 구현 필요
    }
}

extension DateFormatter {
    static let nicepayFormat: DateFormatter = {
        let formatter = DateFormatter()
        formatter.dateFormat = "yyyyMMddHHmmss"
        return formatter
    }()
    
    static let yyyyMMdd: DateFormatter = {
        let formatter = DateFormatter()
        formatter.dateFormat = "yyyyMMdd"
        return formatter
    }()
}
```