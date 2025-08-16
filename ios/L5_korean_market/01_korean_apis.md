# 한국 주요 API 통합 가이드

## 카카오 API 완벽 통합

```swift
import UIKit
import KakaoSDKCommon
import KakaoSDKAuth
import KakaoSDKUser
import KakaoSDKTalk
import KakaoSDKShare
import KakaoSDKNavi

// MARK: - Kakao SDK Manager
class KakaoManager: NSObject {
    static let shared = KakaoManager()
    
    private override init() {
        super.init()
    }
    
    // MARK: - 초기화
    func initializeSDK() {
        KakaoSDK.initSDK(appKey: "YOUR_NATIVE_APP_KEY")
    }
    
    // MARK: - 카카오 로그인
    func loginWithKakaoTalk(completion: @escaping (Result<OAuthToken, Error>) -> Void) {
        // 카카오톡 실행 가능 여부 확인
        if UserApi.isKakaoTalkLoginAvailable() {
            UserApi.shared.loginWithKakaoTalk { oauthToken, error in
                if let error = error {
                    completion(.failure(error))
                } else if let token = oauthToken {
                    completion(.success(token))
                    self.fetchUserInfo()
                }
            }
        } else {
            // 카카오톡이 없으면 카카오 계정으로 로그인
            loginWithKakaoAccount(completion: completion)
        }
    }
    
    func loginWithKakaoAccount(completion: @escaping (Result<OAuthToken, Error>) -> Void) {
        UserApi.shared.loginWithKakaoAccount { oauthToken, error in
            if let error = error {
                completion(.failure(error))
            } else if let token = oauthToken {
                completion(.success(token))
                self.fetchUserInfo()
            }
        }
    }
    
    // MARK: - 사용자 정보 가져오기
    func fetchUserInfo() {
        UserApi.shared.me { user, error in
            if let error = error {
                print("사용자 정보 가져오기 실패: \(error)")
            } else if let user = user {
                // 사용자 정보 처리
                let userId = user.id
                let nickname = user.kakaoAccount?.profile?.nickname
                let email = user.kakaoAccount?.email
                let profileImageUrl = user.kakaoAccount?.profile?.profileImageUrl
                
                // 사용자 정보 저장
                self.saveUserInfo(
                    id: userId ?? 0,
                    nickname: nickname ?? "",
                    email: email ?? "",
                    profileImageUrl: profileImageUrl?.absoluteString ?? ""
                )
            }
        }
    }
    
    private func saveUserInfo(id: Int64, nickname: String, email: String, profileImageUrl: String) {
        UserDefaults.standard.set(id, forKey: "kakaoUserId")
        UserDefaults.standard.set(nickname, forKey: "kakaoNickname")
        UserDefaults.standard.set(email, forKey: "kakaoEmail")
        UserDefaults.standard.set(profileImageUrl, forKey: "kakaoProfileImage")
    }
    
    // MARK: - 카카오톡 메시지 보내기
    func sendKakaoMessage(recipe: Recipe) {
        // 메시지 템플릿 생성
        let link = Link(
            webUrl: URL(string: "https://recipebox.app/recipe/\(recipe.id)"),
            mobileWebUrl: URL(string: "https://recipebox.app/recipe/\(recipe.id)")
        )
        
        let content = Content(
            title: recipe.title,
            imageUrl: URL(string: recipe.imageUrl ?? ""),
            description: recipe.description_text,
            link: link
        )
        
        let social = Social(
            likeCount: recipe.likes,
            commentCount: recipe.reviews.count,
            sharedCount: recipe.shares
        )
        
        let button = Button(
            title: "레시피 보기",
            link: link
        )
        
        let feedTemplate = FeedTemplate(
            content: content,
            social: social,
            buttons: [button]
        )
        
        // 메시지 보내기
        if let templateJsonObject = try? SdkJSONEncoder.custom.encodeToJsonObject(feedTemplate) {
            TalkApi.shared.sendDefaultMessage(templateObject: templateJsonObject) { error in
                if let error = error {
                    print("메시지 전송 실패: \(error)")
                } else {
                    print("메시지 전송 성공")
                }
            }
        }
    }
    
    // MARK: - 카카오 지도 API
    func searchNearbyRestaurants(
        latitude: Double,
        longitude: Double,
        radius: Int = 1000,
        completion: @escaping (Result<[Restaurant], Error>) -> Void
    ) {
        let urlString = "https://dapi.kakao.com/v2/local/search/category.json"
        let parameters: [String: Any] = [
            "category_group_code": "FD6", // 음식점
            "x": longitude,
            "y": latitude,
            "radius": radius,
            "sort": "distance"
        ]
        
        var components = URLComponents(string: urlString)!
        components.queryItems = parameters.map { URLQueryItem(name: $0.key, value: "\($0.value)") }
        
        var request = URLRequest(url: components.url!)
        request.setValue("KakaoAK YOUR_REST_API_KEY", forHTTPHeaderField: "Authorization")
        
        URLSession.shared.dataTask(with: request) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let data = data else {
                completion(.failure(APIError.noData))
                return
            }
            
            do {
                let response = try JSONDecoder().decode(KakaoPlaceResponse.self, from: data)
                let restaurants = response.documents.map { document in
                    Restaurant(
                        id: document.id,
                        name: document.placeName,
                        category: document.categoryName,
                        address: document.addressName,
                        roadAddress: document.roadAddressName,
                        phone: document.phone,
                        latitude: Double(document.y) ?? 0,
                        longitude: Double(document.x) ?? 0,
                        distance: Int(document.distance) ?? 0,
                        placeUrl: document.placeUrl
                    )
                }
                completion(.success(restaurants))
            } catch {
                completion(.failure(error))
            }
        }.resume()
    }
    
    // MARK: - 카카오 네비게이션
    func navigateToPlace(destination: Location) {
        let destination = NaviLocation(
            name: destination.name,
            x: "\(destination.longitude)",
            y: "\(destination.latitude)"
        )
        
        guard let navigateUrl = NaviApi.shared.navigateUrl(destination: destination) else { return }
        
        if UIApplication.shared.canOpenURL(navigateUrl) {
            UIApplication.shared.open(navigateUrl)
        } else {
            // 카카오내비 앱 스토어로 이동
            if let appStoreUrl = URL(string: "https://apps.apple.com/kr/app/id417698849") {
                UIApplication.shared.open(appStoreUrl)
            }
        }
    }
    
    // MARK: - 카카오 채널 추가
    func addKakaoChannel(channelPublicId: String) {
        TalkApi.shared.addChannel(channelPublicId: channelPublicId) { error in
            if let error = error {
                print("채널 추가 실패: \(error)")
            } else {
                print("채널 추가 성공")
            }
        }
    }
}

// MARK: - 네이버 API 통합
class NaverAPIManager: NSObject {
    static let shared = NaverAPIManager()
    
    private let clientId = "YOUR_CLIENT_ID"
    private let clientSecret = "YOUR_CLIENT_SECRET"
    
    // MARK: - 네이버 로그인
    func loginWithNaver(completion: @escaping (Result<String, Error>) -> Void) {
        let naverConnection = NaverThirdPartyLoginConnection.getSharedInstance()
        naverConnection?.delegate = self
        naverConnection?.requestThirdPartyLogin()
    }
    
    // MARK: - 네이버 쇼핑 API - 재료 가격 검색
    func searchIngredientPrices(
        query: String,
        completion: @escaping (Result<[ShoppingItem], Error>) -> Void
    ) {
        let urlString = "https://openapi.naver.com/v1/search/shop.json"
        var components = URLComponents(string: urlString)!
        components.queryItems = [
            URLQueryItem(name: "query", value: query),
            URLQueryItem(name: "display", value: "20"),
            URLQueryItem(name: "sort", value: "sim") // 정확도순
        ]
        
        var request = URLRequest(url: components.url!)
        request.setValue(clientId, forHTTPHeaderField: "X-Naver-Client-Id")
        request.setValue(clientSecret, forHTTPHeaderField: "X-Naver-Client-Secret")
        
        URLSession.shared.dataTask(with: request) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let data = data else {
                completion(.failure(APIError.noData))
                return
            }
            
            do {
                let response = try JSONDecoder().decode(NaverShoppingResponse.self, from: data)
                let items = response.items.map { item in
                    ShoppingItem(
                        title: item.title.replacingOccurrences(of: "<[^>]+>", with: "", options: .regularExpression),
                        link: item.link,
                        image: item.image,
                        lprice: Int(item.lprice) ?? 0,
                        hprice: Int(item.hprice) ?? 0,
                        mallName: item.mallName,
                        productId: item.productId,
                        productType: item.productType,
                        brand: item.brand,
                        maker: item.maker,
                        category: self.parseCategory(item.category1, item.category2, item.category3, item.category4)
                    )
                }
                completion(.success(items))
            } catch {
                completion(.failure(error))
            }
        }.resume()
    }
    
    private func parseCategory(_ cat1: String, _ cat2: String, _ cat3: String, _ cat4: String) -> String {
        return [cat1, cat2, cat3, cat4]
            .filter { !$0.isEmpty }
            .joined(separator: " > ")
    }
    
    // MARK: - Papago 번역 API
    func translateText(
        text: String,
        source: String = "ko",
        target: String = "en",
        completion: @escaping (Result<String, Error>) -> Void
    ) {
        let urlString = "https://openapi.naver.com/v1/papago/n2mt"
        let url = URL(string: urlString)!
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue(clientId, forHTTPHeaderField: "X-Naver-Client-Id")
        request.setValue(clientSecret, forHTTPHeaderField: "X-Naver-Client-Secret")
        request.setValue("application/x-www-form-urlencoded; charset=UTF-8", forHTTPHeaderField: "Content-Type")
        
        let body = "source=\(source)&target=\(target)&text=\(text.addingPercentEncoding(withAllowedCharacters: .urlQueryAllowed) ?? "")"
        request.httpBody = body.data(using: .utf8)
        
        URLSession.shared.dataTask(with: request) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let data = data else {
                completion(.failure(APIError.noData))
                return
            }
            
            do {
                let response = try JSONDecoder().decode(PapagoResponse.self, from: data)
                completion(.success(response.message.result.translatedText))
            } catch {
                completion(.failure(error))
            }
        }.resume()
    }
    
    // MARK: - 네이버 블로그 검색
    func searchRecipeBlog(
        query: String,
        completion: @escaping (Result<[BlogPost], Error>) -> Void
    ) {
        let urlString = "https://openapi.naver.com/v1/search/blog.json"
        var components = URLComponents(string: urlString)!
        components.queryItems = [
            URLQueryItem(name: "query", value: "\(query) 레시피"),
            URLQueryItem(name: "display", value: "20"),
            URLQueryItem(name: "sort", value: "sim")
        ]
        
        var request = URLRequest(url: components.url!)
        request.setValue(clientId, forHTTPHeaderField: "X-Naver-Client-Id")
        request.setValue(clientSecret, forHTTPHeaderField: "X-Naver-Client-Secret")
        
        URLSession.shared.dataTask(with: request) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let data = data else {
                completion(.failure(APIError.noData))
                return
            }
            
            do {
                let response = try JSONDecoder().decode(NaverBlogResponse.self, from: data)
                let posts = response.items.map { item in
                    BlogPost(
                        title: item.title.replacingOccurrences(of: "<[^>]+>", with: "", options: .regularExpression),
                        link: item.link,
                        description: item.description.replacingOccurrences(of: "<[^>]+>", with: "", options: .regularExpression),
                        bloggername: item.bloggername,
                        bloggerlink: item.bloggerlink,
                        postdate: item.postdate
                    )
                }
                completion(.success(posts))
            } catch {
                completion(.failure(error))
            }
        }.resume()
    }
}

// MARK: - 공공데이터 API
class PublicDataManager {
    static let shared = PublicDataManager()
    
    private let serviceKey = "YOUR_SERVICE_KEY"
    
    // MARK: - 식품 안전 정보
    func getFoodSafetyInfo(
        productName: String,
        completion: @escaping (Result<[FoodSafetyInfo], Error>) -> Void
    ) {
        let urlString = "http://apis.data.go.kr/1471000/FoodSafetyService/getFoodSafetyInfo"
        var components = URLComponents(string: urlString)!
        components.queryItems = [
            URLQueryItem(name: "serviceKey", value: serviceKey),
            URLQueryItem(name: "prdlstNm", value: productName),
            URLQueryItem(name: "returnType", value: "json"),
            URLQueryItem(name: "pageNo", value: "1"),
            URLQueryItem(name: "numOfRows", value: "100")
        ]
        
        var request = URLRequest(url: components.url!)
        
        URLSession.shared.dataTask(with: request) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let data = data else {
                completion(.failure(APIError.noData))
                return
            }
            
            do {
                let response = try JSONDecoder().decode(FoodSafetyResponse.self, from: data)
                let safetyInfos = response.body.items.map { item in
                    FoodSafetyInfo(
                        productName: item.prdlstNm,
                        manufacturer: item.manufacture,
                        reportNumber: item.prdlstReportNo,
                        category: item.prdkind,
                        rawMaterials: item.rawmtrl,
                        nutrients: self.parseNutrients(item.nutrient),
                        allergyInfo: item.allergy,
                        expirationDate: item.pog,
                        barcode: item.barcode
                    )
                }
                completion(.success(safetyInfos))
            } catch {
                completion(.failure(error))
            }
        }.resume()
    }
    
    private func parseNutrients(_ nutrientString: String) -> [String: String] {
        // 영양 정보 파싱
        var nutrients: [String: String] = [:]
        let components = nutrientString.components(separatedBy: ",")
        for component in components {
            let parts = component.components(separatedBy: ":")
            if parts.count == 2 {
                nutrients[parts[0].trimmingCharacters(in: .whitespaces)] = parts[1].trimmingCharacters(in: .whitespaces)
            }
        }
        return nutrients
    }
    
    // MARK: - 농산물 가격 정보
    func getAgriculturalPrices(
        itemName: String,
        completion: @escaping (Result<[AgriculturalPrice], Error>) -> Void
    ) {
        let urlString = "http://www.kamis.or.kr/service/price/xml.do"
        var components = URLComponents(string: urlString)!
        components.queryItems = [
            URLQueryItem(name: "action", value: "dailyPriceByCategoryList"),
            URLQueryItem(name: "p_cert_key", value: serviceKey),
            URLQueryItem(name: "p_cert_id", value: "YOUR_CERT_ID"),
            URLQueryItem(name: "p_returntype", value: "json"),
            URLQueryItem(name: "p_product_cls_code", value: "01"), // 소매
            URLQueryItem(name: "p_item_category_code", value: "100"), // 채소류
            URLQueryItem(name: "p_country_code", value: "1101"), // 서울
            URLQueryItem(name: "p_regday", value: DateFormatter.yyyyMMdd.string(from: Date()))
        ]
        
        var request = URLRequest(url: components.url!)
        
        URLSession.shared.dataTask(with: request) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let data = data else {
                completion(.failure(APIError.noData))
                return
            }
            
            do {
                let response = try JSONDecoder().decode(KamisResponse.self, from: data)
                let prices = response.data.item.map { item in
                    AgriculturalPrice(
                        itemName: item.itemName,
                        kindName: item.kindName,
                        unit: item.unit,
                        price: Int(item.dpr1) ?? 0,
                        previousPrice: Int(item.dpr2) ?? 0,
                        yearAgoPrice: Int(item.dpr3) ?? 0,
                        marketName: item.countyname
                    )
                }
                completion(.success(prices))
            } catch {
                completion(.failure(error))
            }
        }.resume()
    }
    
    // MARK: - 식당 위생 등급
    func getRestaurantHygieneGrade(
        restaurantName: String,
        completion: @escaping (Result<[RestaurantHygiene], Error>) -> Void
    ) {
        let urlString = "http://apis.data.go.kr/B552584/RestaurantService/getRestaurantList"
        var components = URLComponents(string: urlString)!
        components.queryItems = [
            URLQueryItem(name: "serviceKey", value: serviceKey),
            URLQueryItem(name: "bplcNm", value: restaurantName),
            URLQueryItem(name: "returnType", value: "json"),
            URLQueryItem(name: "pageNo", value: "1"),
            URLQueryItem(name: "numOfRows", value: "100")
        ]
        
        var request = URLRequest(url: components.url!)
        
        URLSession.shared.dataTask(with: request) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let data = data else {
                completion(.failure(APIError.noData))
                return
            }
            
            do {
                let response = try JSONDecoder().decode(RestaurantHygieneResponse.self, from: data)
                let hygieneInfos = response.body.items.map { item in
                    RestaurantHygiene(
                        name: item.bplcNm,
                        address: item.siteTel,
                        grade: item.grade,
                        businessType: item.uptaeNm,
                        inspectionDate: item.crtYmd,
                        phoneNumber: item.siteTel
                    )
                }
                completion(.success(hygieneInfos))
            } catch {
                completion(.failure(error))
            }
        }.resume()
    }
}

// MARK: - 통합 API View
import SwiftUI

struct KoreanAPIIntegrationView: View {
    @State private var selectedAPI: APIType = .kakaoMap
    @State private var searchQuery = ""
    @State private var searchResults: [Any] = []
    @State private var isLoading = false
    @State private var currentLocation: CLLocationCoordinate2D?
    
    enum APIType: String, CaseIterable {
        case kakaoMap = "카카오 지도"
        case naverShopping = "네이버 쇼핑"
        case foodSafety = "식품 안전"
        case agriculturalPrice = "농산물 가격"
        case translation = "번역"
        case blog = "블로그 검색"
        
        var icon: String {
            switch self {
            case .kakaoMap: return "map"
            case .naverShopping: return "cart"
            case .foodSafety: return "checkmark.shield"
            case .agriculturalPrice: return "leaf"
            case .translation: return "globe"
            case .blog: return "text.justify"
            }
        }
    }
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 0) {
                // API Selector
                ScrollView(.horizontal, showsIndicators: false) {
                    HStack(spacing: 16) {
                        ForEach(APIType.allCases, id: \.self) { api in
                            APITypeButton(
                                type: api,
                                isSelected: selectedAPI == api
                            ) {
                                selectedAPI = api
                                searchResults = []
                            }
                        }
                    }
                    .padding()
                }
                
                // Search Bar
                HStack {
                    Image(systemName: "magnifyingglass")
                        .foregroundColor(.secondary)
                    
                    TextField(searchPlaceholder, text: $searchQuery)
                        .textFieldStyle(RoundedBorderTextFieldStyle())
                        .onSubmit {
                            performSearch()
                        }
                    
                    if isLoading {
                        ProgressView()
                    }
                }
                .padding(.horizontal)
                
                // Results
                ScrollView {
                    if searchResults.isEmpty && !isLoading {
                        EmptyStateView(apiType: selectedAPI)
                            .padding(.top, 100)
                    } else {
                        LazyVStack(spacing: 16) {
                            ForEach(0..<searchResults.count, id: \.self) { index in
                                resultView(for: searchResults[index])
                            }
                        }
                        .padding()
                    }
                }
            }
            .navigationTitle("한국 API 통합")
            .onAppear {
                requestLocationPermission()
            }
        }
    }
    
    var searchPlaceholder: String {
        switch selectedAPI {
        case .kakaoMap: return "주변 음식점 검색..."
        case .naverShopping: return "재료 가격 검색..."
        case .foodSafety: return "식품 안전 정보 검색..."
        case .agriculturalPrice: return "농산물 가격 검색..."
        case .translation: return "번역할 텍스트 입력..."
        case .blog: return "레시피 블로그 검색..."
        }
    }
    
    func performSearch() {
        isLoading = true
        
        switch selectedAPI {
        case .kakaoMap:
            searchNearbyRestaurants()
        case .naverShopping:
            searchShoppingItems()
        case .foodSafety:
            searchFoodSafety()
        case .agriculturalPrice:
            searchAgriculturalPrices()
        case .translation:
            translateText()
        case .blog:
            searchBlogs()
        }
    }
    
    func searchNearbyRestaurants() {
        guard let location = currentLocation else {
            isLoading = false
            return
        }
        
        KakaoManager.shared.searchNearbyRestaurants(
            latitude: location.latitude,
            longitude: location.longitude
        ) { result in
            DispatchQueue.main.async {
                switch result {
                case .success(let restaurants):
                    self.searchResults = restaurants
                case .failure(let error):
                    print("Error: \(error)")
                }
                self.isLoading = false
            }
        }
    }
    
    func searchShoppingItems() {
        NaverAPIManager.shared.searchIngredientPrices(query: searchQuery) { result in
            DispatchQueue.main.async {
                switch result {
                case .success(let items):
                    self.searchResults = items
                case .failure(let error):
                    print("Error: \(error)")
                }
                self.isLoading = false
            }
        }
    }
    
    func searchFoodSafety() {
        PublicDataManager.shared.getFoodSafetyInfo(productName: searchQuery) { result in
            DispatchQueue.main.async {
                switch result {
                case .success(let infos):
                    self.searchResults = infos
                case .failure(let error):
                    print("Error: \(error)")
                }
                self.isLoading = false
            }
        }
    }
    
    func searchAgriculturalPrices() {
        PublicDataManager.shared.getAgriculturalPrices(itemName: searchQuery) { result in
            DispatchQueue.main.async {
                switch result {
                case .success(let prices):
                    self.searchResults = prices
                case .failure(let error):
                    print("Error: \(error)")
                }
                self.isLoading = false
            }
        }
    }
    
    func translateText() {
        NaverAPIManager.shared.translateText(text: searchQuery) { result in
            DispatchQueue.main.async {
                switch result {
                case .success(let translated):
                    self.searchResults = [TranslationResult(original: self.searchQuery, translated: translated)]
                case .failure(let error):
                    print("Error: \(error)")
                }
                self.isLoading = false
            }
        }
    }
    
    func searchBlogs() {
        NaverAPIManager.shared.searchRecipeBlog(query: searchQuery) { result in
            DispatchQueue.main.async {
                switch result {
                case .success(let posts):
                    self.searchResults = posts
                case .failure(let error):
                    print("Error: \(error)")
                }
                self.isLoading = false
            }
        }
    }
    
    @ViewBuilder
    func resultView(for item: Any) -> some View {
        if let restaurant = item as? Restaurant {
            RestaurantCard(restaurant: restaurant)
        } else if let shoppingItem = item as? ShoppingItem {
            ShoppingItemCard(item: shoppingItem)
        } else if let foodSafety = item as? FoodSafetyInfo {
            FoodSafetyCard(info: foodSafety)
        } else if let price = item as? AgriculturalPrice {
            AgriculturalPriceCard(price: price)
        } else if let translation = item as? TranslationResult {
            TranslationCard(result: translation)
        } else if let blogPost = item as? BlogPost {
            BlogPostCard(post: blogPost)
        }
    }
    
    func requestLocationPermission() {
        // Location permission request
    }
}

// MARK: - Result Cards
struct RestaurantCard: View {
    let restaurant: Restaurant
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            HStack {
                VStack(alignment: .leading, spacing: 4) {
                    Text(restaurant.name)
                        .font(.headline)
                    
                    Text(restaurant.category)
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
                
                Spacer()
                
                Text("\(restaurant.distance)m")
                    .font(.caption)
                    .padding(.horizontal, 8)
                    .padding(.vertical, 4)
                    .background(Color.blue.opacity(0.1))
                    .foregroundColor(.blue)
                    .cornerRadius(8)
            }
            
            Text(restaurant.roadAddress)
                .font(.subheadline)
                .foregroundColor(.secondary)
            
            if !restaurant.phone.isEmpty {
                HStack {
                    Image(systemName: "phone")
                        .font(.caption)
                    Text(restaurant.phone)
                        .font(.caption)
                }
                .foregroundColor(.blue)
            }
            
            HStack {
                Button {
                    // Navigate
                    let location = Location(
                        name: restaurant.name,
                        latitude: restaurant.latitude,
                        longitude: restaurant.longitude
                    )
                    KakaoManager.shared.navigateToPlace(destination: location)
                } label: {
                    Label("길찾기", systemImage: "location.fill")
                        .font(.caption)
                        .padding(.horizontal, 12)
                        .padding(.vertical, 6)
                        .background(Color.blue)
                        .foregroundColor(.white)
                        .cornerRadius(8)
                }
                
                Spacer()
                
                if let url = URL(string: restaurant.placeUrl) {
                    Link(destination: url) {
                        Label("상세보기", systemImage: "safari")
                            .font(.caption)
                            .padding(.horizontal, 12)
                            .padding(.vertical, 6)
                            .background(Color(.systemGray6))
                            .foregroundColor(.primary)
                            .cornerRadius(8)
                    }
                }
            }
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(radius: 2)
    }
}

struct ShoppingItemCard: View {
    let item: ShoppingItem
    
    var body: some View {
        HStack(spacing: 12) {
            AsyncImage(url: URL(string: item.image)) { image in
                image
                    .resizable()
                    .aspectRatio(contentMode: .fill)
            } placeholder: {
                RoundedRectangle(cornerRadius: 8)
                    .fill(Color(.systemGray5))
            }
            .frame(width: 80, height: 80)
            .cornerRadius(8)
            
            VStack(alignment: .leading, spacing: 4) {
                Text(item.title)
                    .font(.subheadline)
                    .lineLimit(2)
                
                Text("₩\(item.lprice.formatted())")
                    .font(.headline)
                    .foregroundColor(.blue)
                
                HStack {
                    Text(item.mallName)
                        .font(.caption)
                        .foregroundColor(.secondary)
                    
                    Spacer()
                    
                    if !item.brand.isEmpty {
                        Text(item.brand)
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }
                }
            }
            
            Spacer()
            
            Link(destination: URL(string: item.link)!) {
                Image(systemName: "cart.fill")
                    .foregroundColor(.white)
                    .padding(8)
                    .background(Color.green)
                    .cornerRadius(8)
            }
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(radius: 2)
    }
}

struct FoodSafetyCard: View {
    let info: FoodSafetyInfo
    @State private var isExpanded = false
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            HStack {
                VStack(alignment: .leading, spacing: 4) {
                    Text(info.productName)
                        .font(.headline)
                    
                    Text(info.manufacturer)
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
                
                Spacer()
                
                Image(systemName: "checkmark.shield.fill")
                    .foregroundColor(.green)
            }
            
            LabeledContent("품목", value: info.category)
                .font(.subheadline)
            
            LabeledContent("신고번호", value: info.reportNumber)
                .font(.caption)
                .foregroundColor(.secondary)
            
            if isExpanded {
                VStack(alignment: .leading, spacing: 8) {
                    Text("원재료")
                        .font(.caption)
                        .bold()
                    Text(info.rawMaterials)
                        .font(.caption)
                    
                    if !info.allergyInfo.isEmpty {
                        Text("알레르기 정보")
                            .font(.caption)
                            .bold()
                            .padding(.top, 4)
                        Text(info.allergyInfo)
                            .font(.caption)
                            .foregroundColor(.red)
                    }
                    
                    if !info.nutrients.isEmpty {
                        Text("영양 정보")
                            .font(.caption)
                            .bold()
                            .padding(.top, 4)
                        ForEach(Array(info.nutrients.keys), id: \.self) { key in
                            HStack {
                                Text(key)
                                Spacer()
                                Text(info.nutrients[key] ?? "")
                            }
                            .font(.caption)
                        }
                    }
                }
                .padding(.top)
            }
            
            Button {
                withAnimation {
                    isExpanded.toggle()
                }
            } label: {
                HStack {
                    Text(isExpanded ? "접기" : "더보기")
                        .font(.caption)
                    Image(systemName: isExpanded ? "chevron.up" : "chevron.down")
                        .font(.caption)
                }
                .foregroundColor(.blue)
            }
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(radius: 2)
    }
}

struct AgriculturalPriceCard: View {
    let price: AgriculturalPrice
    
    var priceChange: Double {
        Double(price.price - price.previousPrice) / Double(price.previousPrice) * 100
    }
    
    var priceChangeColor: Color {
        if priceChange > 0 {
            return .red
        } else if priceChange < 0 {
            return .blue
        } else {
            return .gray
        }
    }
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            HStack {
                VStack(alignment: .leading, spacing: 4) {
                    Text(price.itemName)
                        .font(.headline)
                    
                    Text(price.kindName)
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
                
                Spacer()
                
                VStack(alignment: .trailing, spacing: 2) {
                    Text("₩\(price.price)")
                        .font(.title3)
                        .bold()
                    
                    HStack(spacing: 2) {
                        Image(systemName: priceChange > 0 ? "arrow.up" : "arrow.down")
                            .font(.caption)
                        Text(String(format: "%.1f%%", abs(priceChange)))
                            .font(.caption)
                    }
                    .foregroundColor(priceChangeColor)
                }
            }
            
            HStack {
                VStack(alignment: .leading, spacing: 2) {
                    Text("전일")
                        .font(.caption2)
                        .foregroundColor(.secondary)
                    Text("₩\(price.previousPrice)")
                        .font(.caption)
                }
                
                Spacer()
                
                VStack(alignment: .leading, spacing: 2) {
                    Text("전년")
                        .font(.caption2)
                        .foregroundColor(.secondary)
                    Text("₩\(price.yearAgoPrice)")
                        .font(.caption)
                }
                
                Spacer()
                
                VStack(alignment: .trailing, spacing: 2) {
                    Text("단위")
                        .font(.caption2)
                        .foregroundColor(.secondary)
                    Text(price.unit)
                        .font(.caption)
                }
            }
            .padding(.top, 8)
            
            Text(price.marketName)
                .font(.caption)
                .padding(.horizontal, 8)
                .padding(.vertical, 4)
                .background(Color(.systemGray6))
                .cornerRadius(8)
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(radius: 2)
    }
}

struct TranslationCard: View {
    let result: TranslationResult
    
    var body: some View {
        VStack(alignment: .leading, spacing: 16) {
            VStack(alignment: .leading, spacing: 8) {
                HStack {
                    Image(systemName: "flag.fill")
                        .foregroundColor(.blue)
                    Text("한국어")
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
                
                Text(result.original)
                    .font(.body)
            }
            
            Divider()
            
            VStack(alignment: .leading, spacing: 8) {
                HStack {
                    Image(systemName: "globe")
                        .foregroundColor(.green)
                    Text("영어")
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
                
                Text(result.translated)
                    .font(.body)
            }
            
            HStack {
                Button {
                    UIPasteboard.general.string = result.translated
                } label: {
                    Label("복사", systemImage: "doc.on.doc")
                        .font(.caption)
                        .padding(.horizontal, 12)
                        .padding(.vertical, 6)
                        .background(Color(.systemGray6))
                        .cornerRadius(8)
                }
                
                Spacer()
            }
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(radius: 2)
    }
}

struct BlogPostCard: View {
    let post: BlogPost
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text(post.title)
                .font(.headline)
                .lineLimit(2)
            
            Text(post.description)
                .font(.subheadline)
                .foregroundColor(.secondary)
                .lineLimit(3)
            
            HStack {
                Text(post.bloggername)
                    .font(.caption)
                    .foregroundColor(.blue)
                
                Spacer()
                
                Text(formatDate(post.postdate))
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            
            Link(destination: URL(string: post.link)!) {
                Text("블로그 보기")
                    .font(.caption)
                    .foregroundColor(.white)
                    .padding(.horizontal, 16)
                    .padding(.vertical, 8)
                    .background(Color.green)
                    .cornerRadius(8)
            }
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(12)
        .shadow(radius: 2)
    }
    
    func formatDate(_ dateString: String) -> String {
        // Format: 20231225 -> 2023.12.25
        guard dateString.count == 8 else { return dateString }
        let year = String(dateString.prefix(4))
        let month = String(dateString.dropFirst(4).prefix(2))
        let day = String(dateString.suffix(2))
        return "\(year).\(month).\(day)"
    }
}

// MARK: - Supporting Views
struct APITypeButton: View {
    let type: KoreanAPIIntegrationView.APIType
    let isSelected: Bool
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            VStack(spacing: 8) {
                Image(systemName: type.icon)
                    .font(.title2)
                
                Text(type.rawValue)
                    .font(.caption)
            }
            .foregroundColor(isSelected ? .white : .primary)
            .frame(width: 80, height: 80)
            .background(isSelected ? Color.blue : Color(.systemGray6))
            .cornerRadius(12)
        }
    }
}

struct EmptyStateView: View {
    let apiType: KoreanAPIIntegrationView.APIType
    
    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: apiType.icon)
                .font(.system(size: 60))
                .foregroundColor(.secondary)
            
            Text(emptyMessage)
                .font(.headline)
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)
        }
    }
    
    var emptyMessage: String {
        switch apiType {
        case .kakaoMap:
            return "위치 권한을 허용하면\n주변 음식점을 찾을 수 있어요"
        case .naverShopping:
            return "재료 이름을 검색하면\n최저가를 찾아드려요"
        case .foodSafety:
            return "식품명을 검색하면\n안전 정보를 확인할 수 있어요"
        case .agriculturalPrice:
            return "농산물 이름을 검색하면\n시세를 확인할 수 있어요"
        case .translation:
            return "번역할 텍스트를 입력하세요"
        case .blog:
            return "레시피를 검색하면\n블로그 포스트를 찾아드려요"
        }
    }
}

// MARK: - Models
struct Restaurant {
    let id: String
    let name: String
    let category: String
    let address: String
    let roadAddress: String
    let phone: String
    let latitude: Double
    let longitude: Double
    let distance: Int
    let placeUrl: String
}

struct ShoppingItem {
    let title: String
    let link: String
    let image: String
    let lprice: Int
    let hprice: Int
    let mallName: String
    let productId: String
    let productType: String
    let brand: String
    let maker: String
    let category: String
}

struct FoodSafetyInfo {
    let productName: String
    let manufacturer: String
    let reportNumber: String
    let category: String
    let rawMaterials: String
    let nutrients: [String: String]
    let allergyInfo: String
    let expirationDate: String
    let barcode: String
}

struct AgriculturalPrice {
    let itemName: String
    let kindName: String
    let unit: String
    let price: Int
    let previousPrice: Int
    let yearAgoPrice: Int
    let marketName: String
}

struct BlogPost {
    let title: String
    let link: String
    let description: String
    let bloggername: String
    let bloggerlink: String
    let postdate: String
}

struct TranslationResult {
    let original: String
    let translated: String
}

struct Location {
    let name: String
    let latitude: Double
    let longitude: Double
}

struct RestaurantHygiene {
    let name: String
    let address: String
    let grade: String
    let businessType: String
    let inspectionDate: String
    let phoneNumber: String
}

// MARK: - Response Models
struct KakaoPlaceResponse: Codable {
    let documents: [PlaceDocument]
    
    struct PlaceDocument: Codable {
        let id: String
        let placeName: String
        let categoryName: String
        let addressName: String
        let roadAddressName: String
        let phone: String
        let x: String
        let y: String
        let distance: String
        let placeUrl: String
        
        enum CodingKeys: String, CodingKey {
            case id
            case placeName = "place_name"
            case categoryName = "category_name"
            case addressName = "address_name"
            case roadAddressName = "road_address_name"
            case phone
            case x, y
            case distance
            case placeUrl = "place_url"
        }
    }
}

struct NaverShoppingResponse: Codable {
    let items: [ShoppingItemResponse]
    
    struct ShoppingItemResponse: Codable {
        let title: String
        let link: String
        let image: String
        let lprice: String
        let hprice: String
        let mallName: String
        let productId: String
        let productType: String
        let brand: String
        let maker: String
        let category1: String
        let category2: String
        let category3: String
        let category4: String
    }
}

struct NaverBlogResponse: Codable {
    let items: [BlogItemResponse]
    
    struct BlogItemResponse: Codable {
        let title: String
        let link: String
        let description: String
        let bloggername: String
        let bloggerlink: String
        let postdate: String
    }
}

struct PapagoResponse: Codable {
    let message: Message
    
    struct Message: Codable {
        let result: Result
        
        struct Result: Codable {
            let translatedText: String
        }
    }
}

struct FoodSafetyResponse: Codable {
    let body: Body
    
    struct Body: Codable {
        let items: [FoodSafetyItem]
        
        struct FoodSafetyItem: Codable {
            let prdlstNm: String
            let manufacture: String
            let prdlstReportNo: String
            let prdkind: String
            let rawmtrl: String
            let nutrient: String
            let allergy: String
            let pog: String
            let barcode: String
        }
    }
}

struct KamisResponse: Codable {
    let data: DataResponse
    
    struct DataResponse: Codable {
        let item: [KamisItem]
        
        struct KamisItem: Codable {
            let itemName: String
            let kindName: String
            let unit: String
            let dpr1: String
            let dpr2: String
            let dpr3: String
            let countyname: String
        }
    }
}

struct RestaurantHygieneResponse: Codable {
    let body: Body
    
    struct Body: Codable {
        let items: [HygieneItem]
        
        struct HygieneItem: Codable {
            let bplcNm: String
            let siteTel: String
            let grade: String
            let uptaeNm: String
            let crtYmd: String
        }
    }
}

// MARK: - Errors
enum APIError: Error {
    case noData
    case invalidResponse
    case networkError
}

// MARK: - Extensions
import CoreLocation

extension NaverAPIManager: NaverThirdPartyLoginConnectionDelegate {
    func oauth20ConnectionDidFinishRequestACTokenWithAuthCode() {
        // 로그인 성공
    }
    
    func oauth20ConnectionDidFinishRequestACTokenWithRefreshToken() {
        // 토큰 갱신 성공
    }
    
    func oauth20ConnectionDidFinishDeleteToken() {
        // 로그아웃 성공
    }
    
    func oauth20Connection(_ oauthConnection: NaverThirdPartyLoginConnection!, didFailWithError error: Error!) {
        // 에러 처리
    }
}

// Placeholder imports - 실제 앱에서는 적절한 SDK 임포트 필요
struct NaverThirdPartyLoginConnection {
    static func getSharedInstance() -> NaverThirdPartyLoginConnection? { nil }
    var delegate: NaverThirdPartyLoginConnectionDelegate? = nil
    func requestThirdPartyLogin() {}
}

protocol NaverThirdPartyLoginConnectionDelegate {
    func oauth20ConnectionDidFinishRequestACTokenWithAuthCode()
    func oauth20ConnectionDidFinishRequestACTokenWithRefreshToken()
    func oauth20ConnectionDidFinishDeleteToken()
    func oauth20Connection(_ oauthConnection: NaverThirdPartyLoginConnection!, didFailWithError error: Error!)
}
```