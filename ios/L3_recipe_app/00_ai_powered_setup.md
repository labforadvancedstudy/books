# L3-00: AI 기반 레시피 앱 설정
## Core ML, Vision, CreateML로 스마트 요리 도우미

---

> **"Technology should enhance the human experience, not replace it."**

AI와 머신러닝으로 요리를 더 쉽고 즐겁게 만드는 혁신적인 레시피 앱을 구축합니다.

---

## 🎯 목표

**완성 후 결과물**:
- 재료 인식 시스템
- 영양 자동 계산
- 레시피 추천 엔진
- 음성 요리 가이드
- AR 요리 도우미

---

## 🚀 프로젝트 아키텍처

### 전체 구조

```
RecipeBox/
├── App/
│   ├── RecipeBoxApp.swift
│   ├── AppEnvironment.swift
│   └── Configuration/
├── Core/
│   ├── AI/
│   │   ├── Models/
│   │   │   ├── IngredientClassifier.mlmodel
│   │   │   ├── NutritionPredictor.mlmodel
│   │   │   └── RecipeRecommender.mlmodel
│   │   ├── Vision/
│   │   │   ├── IngredientRecognizer.swift
│   │   │   ├── FoodVolumeEstimator.swift
│   │   │   └── PlateAnalyzer.swift
│   │   └── NLP/
│   │       ├── RecipeParser.swift
│   │       └── CookingAssistant.swift
│   ├── Data/
│   │   ├── Models/
│   │   ├── Repositories/
│   │   └── Services/
│   └── Domain/
│       ├── Entities/
│       ├── UseCases/
│       └── Interfaces/
├── Features/
│   ├── Camera/
│   ├── Recipes/
│   ├── MealPlanning/
│   ├── Shopping/
│   └── Social/
└── Resources/
```

---

## 🤖 Core ML 모델 구현

### 1단계: 재료 인식 모델

**Core/AI/Vision/IngredientRecognizer.swift**:

```swift
import Vision
import CoreML
import UIKit
import Combine

class IngredientRecognizer: ObservableObject {
    @Published var recognizedIngredients: [RecognizedIngredient] = []
    @Published var isProcessing = false
    @Published var confidence: Double = 0.0
    
    private var requests = [VNRequest]()
    private let ingredientModel: VNCoreMLModel
    private let nutritionPredictor: NutritionPredictor
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        // Core ML 모델 로드
        guard let model = try? VNCoreMLModel(for: IngredientClassifier().model) else {
            fatalError("재료 분류 모델 로드 실패")
        }
        self.ingredientModel = model
        self.nutritionPredictor = NutritionPredictor()
        
        setupVision()
    }
    
    private func setupVision() {
        let ingredientRequest = VNCoreMLRequest(model: ingredientModel) { [weak self] request, error in
            self?.processIngredientResults(request.results)
        }
        
        ingredientRequest.imageCropAndScaleOption = .centerCrop
        
        // 텍스트 인식 요청 추가 (라벨 읽기)
        let textRequest = VNRecognizeTextRequest { [weak self] request, error in
            self?.processTextResults(request.results)
        }
        textRequest.recognitionLevel = .accurate
        textRequest.recognitionLanguages = ["ko-KR", "en-US"]
        
        // 바코드 인식 요청
        let barcodeRequest = VNDetectBarcodesRequest { [weak self] request, error in
            self?.processBarcodeResults(request.results)
        }
        
        requests = [ingredientRequest, textRequest, barcodeRequest]
    }
    
    func recognizeIngredients(from image: UIImage) async throws -> [RecognizedIngredient] {
        guard let ciImage = CIImage(image: image) else {
            throw RecognitionError.invalidImage
        }
        
        isProcessing = true
        defer { isProcessing = false }
        
        return try await withCheckedThrowingContinuation { continuation in
            let handler = VNImageRequestHandler(ciImage: ciImage, options: [:])
            
            DispatchQueue.global(qos: .userInitiated).async { [weak self] in
                do {
                    try handler.perform(self?.requests ?? [])
                    
                    DispatchQueue.main.async {
                        continuation.resume(returning: self?.recognizedIngredients ?? [])
                    }
                } catch {
                    continuation.resume(throwing: error)
                }
            }
        }
    }
    
    private func processIngredientResults(_ results: [Any]?) {
        guard let results = results as? [VNClassificationObservation] else { return }
        
        let topResults = results.prefix(5).compactMap { observation -> RecognizedIngredient? in
            guard observation.confidence > 0.5 else { return nil }
            
            return RecognizedIngredient(
                id: UUID(),
                name: translateIngredientName(observation.identifier),
                confidence: Double(observation.confidence),
                category: categorizeIngredient(observation.identifier),
                boundingBox: nil,
                nutrition: predictNutrition(for: observation.identifier)
            )
        }
        
        DispatchQueue.main.async {
            self.recognizedIngredients = topResults
            self.confidence = Double(results.first?.confidence ?? 0)
        }
    }
    
    private func processTextResults(_ results: [Any]?) {
        guard let results = results as? [VNRecognizedTextObservation] else { return }
        
        for observation in results {
            guard let topCandidate = observation.topCandidates(1).first else { continue }
            
            // 제품명이나 브랜드 인식
            if let productInfo = extractProductInfo(from: topCandidate.string) {
                let ingredient = RecognizedIngredient(
                    id: UUID(),
                    name: productInfo.name,
                    confidence: Double(topCandidate.confidence),
                    category: .packaged,
                    boundingBox: observation.boundingBox,
                    nutrition: productInfo.nutrition,
                    brand: productInfo.brand
                )
                
                DispatchQueue.main.async {
                    self.recognizedIngredients.append(ingredient)
                }
            }
        }
    }
    
    private func processBarcodeResults(_ results: [Any]?) {
        guard let results = results as? [VNBarcodeObservation] else { return }
        
        for observation in results {
            if let barcode = observation.payloadStringValue {
                Task {
                    if let product = await fetchProductByBarcode(barcode) {
                        let ingredient = RecognizedIngredient(
                            id: UUID(),
                            name: product.name,
                            confidence: 1.0,
                            category: .packaged,
                            boundingBox: observation.boundingBox,
                            nutrition: product.nutrition,
                            brand: product.brand,
                            barcode: barcode
                        )
                        
                        await MainActor.run {
                            self.recognizedIngredients.append(ingredient)
                        }
                    }
                }
            }
        }
    }
    
    private func translateIngredientName(_ identifier: String) -> String {
        // 영어 식별자를 한글로 변환
        let translations: [String: String] = [
            "apple": "사과",
            "banana": "바나나",
            "carrot": "당근",
            "chicken": "닭고기",
            "rice": "쌀",
            "tomato": "토마토",
            "onion": "양파",
            "garlic": "마늘",
            "beef": "소고기",
            "pork": "돼지고기",
            "egg": "계란",
            "milk": "우유",
            "cheese": "치즈",
            "bread": "빵",
            "pasta": "파스타"
        ]
        
        return translations[identifier.lowercased()] ?? identifier
    }
    
    private func categorizeIngredient(_ identifier: String) -> IngredientCategory {
        let categories: [String: IngredientCategory] = [
            "apple": .fruit,
            "banana": .fruit,
            "carrot": .vegetable,
            "chicken": .protein,
            "rice": .grain,
            "milk": .dairy
        ]
        
        return categories[identifier.lowercased()] ?? .other
    }
    
    private func predictNutrition(for ingredient: String) -> NutritionInfo {
        // ML 모델로 영양 정보 예측
        return nutritionPredictor.predict(for: ingredient)
    }
    
    private func fetchProductByBarcode(_ barcode: String) async -> Product? {
        // 바코드로 제품 정보 조회 (API 호출)
        do {
            return try await ProductAPI.shared.fetchProduct(barcode: barcode)
        } catch {
            print("바코드 조회 실패: \(error)")
            return nil
        }
    }
}

// MARK: - Supporting Types

struct RecognizedIngredient: Identifiable {
    let id: UUID
    let name: String
    let confidence: Double
    let category: IngredientCategory
    let boundingBox: CGRect?
    let nutrition: NutritionInfo?
    var brand: String?
    var barcode: String?
    var quantity: Measurement<UnitMass>?
}

enum IngredientCategory: String, CaseIterable {
    case fruit = "과일"
    case vegetable = "채소"
    case protein = "단백질"
    case dairy = "유제품"
    case grain = "곡물"
    case packaged = "가공식품"
    case condiment = "조미료"
    case other = "기타"
    
    var icon: String {
        switch self {
        case .fruit: return "🍎"
        case .vegetable: return "🥬"
        case .protein: return "🥩"
        case .dairy: return "🥛"
        case .grain: return "🌾"
        case .packaged: return "📦"
        case .condiment: return "🧂"
        case .other: return "🍽"
        }
    }
}

struct NutritionInfo: Codable {
    let calories: Double
    let protein: Double
    let carbs: Double
    let fat: Double
    let fiber: Double?
    let sugar: Double?
    let sodium: Double?
    let vitamins: [String: Double]?
    let minerals: [String: Double]?
}

enum RecognitionError: Error {
    case invalidImage
    case modelNotLoaded
    case processingFailed
}
```

### 2단계: 영양 예측 모델

**Core/AI/Models/NutritionPredictor.swift**:

```swift
import CoreML
import TabularData

class NutritionPredictor {
    private let model: MLModel
    private let nutritionDatabase: NutritionDatabase
    
    init() {
        // Core ML 모델 로드
        guard let modelURL = Bundle.main.url(forResource: "NutritionPredictor", withExtension: "mlmodelc"),
              let model = try? MLModel(contentsOf: modelURL) else {
            fatalError("영양 예측 모델 로드 실패")
        }
        
        self.model = model
        self.nutritionDatabase = NutritionDatabase()
    }
    
    func predict(for ingredient: String, quantity: Measurement<UnitMass>? = nil) -> NutritionInfo {
        // 먼저 데이터베이스에서 검색
        if let dbNutrition = nutritionDatabase.getNutrition(for: ingredient) {
            if let quantity = quantity {
                return scaleNutrition(dbNutrition, to: quantity)
            }
            return dbNutrition
        }
        
        // ML 모델로 예측
        do {
            let input = NutritionPredictorInput(
                ingredientName: ingredient,
                category: categorize(ingredient)
            )
            
            let prediction = try model.prediction(from: input)
            
            return NutritionInfo(
                calories: prediction.calories,
                protein: prediction.protein,
                carbs: prediction.carbs,
                fat: prediction.fat,
                fiber: prediction.fiber,
                sugar: prediction.sugar,
                sodium: prediction.sodium,
                vitamins: extractVitamins(from: prediction),
                minerals: extractMinerals(from: prediction)
            )
        } catch {
            // 기본값 반환
            return NutritionInfo(
                calories: 0,
                protein: 0,
                carbs: 0,
                fat: 0,
                fiber: nil,
                sugar: nil,
                sodium: nil,
                vitamins: nil,
                minerals: nil
            )
        }
    }
    
    func predictForRecipe(_ recipe: Recipe) -> NutritionInfo {
        var totalNutrition = NutritionInfo(
            calories: 0,
            protein: 0,
            carbs: 0,
            fat: 0,
            fiber: 0,
            sugar: 0,
            sodium: 0,
            vitamins: [:],
            minerals: [:]
        )
        
        for ingredient in recipe.ingredients {
            let nutrition = predict(
                for: ingredient.name,
                quantity: ingredient.quantity
            )
            
            totalNutrition = combineNutrition(totalNutrition, nutrition)
        }
        
        // 1인분 기준으로 나누기
        return scaleNutrition(totalNutrition, servings: recipe.servings)
    }
    
    private func scaleNutrition(_ nutrition: NutritionInfo, to quantity: Measurement<UnitMass>) -> NutritionInfo {
        let scaleFactor = quantity.converted(to: .grams).value / 100.0 // 100g 기준
        
        return NutritionInfo(
            calories: nutrition.calories * scaleFactor,
            protein: nutrition.protein * scaleFactor,
            carbs: nutrition.carbs * scaleFactor,
            fat: nutrition.fat * scaleFactor,
            fiber: nutrition.fiber.map { $0 * scaleFactor },
            sugar: nutrition.sugar.map { $0 * scaleFactor },
            sodium: nutrition.sodium.map { $0 * scaleFactor },
            vitamins: nutrition.vitamins?.mapValues { $0 * scaleFactor },
            minerals: nutrition.minerals?.mapValues { $0 * scaleFactor }
        )
    }
    
    private func scaleNutrition(_ nutrition: NutritionInfo, servings: Int) -> NutritionInfo {
        let scaleFactor = 1.0 / Double(servings)
        
        return NutritionInfo(
            calories: nutrition.calories * scaleFactor,
            protein: nutrition.protein * scaleFactor,
            carbs: nutrition.carbs * scaleFactor,
            fat: nutrition.fat * scaleFactor,
            fiber: nutrition.fiber.map { $0 * scaleFactor },
            sugar: nutrition.sugar.map { $0 * scaleFactor },
            sodium: nutrition.sodium.map { $0 * scaleFactor },
            vitamins: nutrition.vitamins?.mapValues { $0 * scaleFactor },
            minerals: nutrition.minerals?.mapValues { $0 * scaleFactor }
        )
    }
}
```

### 3단계: 레시피 추천 엔진

**Core/AI/RecipeRecommender.swift**:

```swift
import CoreML
import CreateML
import Combine

class RecipeRecommendationEngine: ObservableObject {
    @Published var recommendations: [Recipe] = []
    @Published var personalizedSuggestions: [RecipeSuggestion] = []
    
    private let userPreferences: UserPreferences
    private let recipeDatabase: RecipeDatabase
    private let mlRecommender: MLRecommender<String>?
    private var cancellables = Set<AnyCancellable>()
    
    init(userPreferences: UserPreferences = .shared) {
        self.userPreferences = userPreferences
        self.recipeDatabase = RecipeDatabase.shared
        
        // 추천 모델 로드
        if let modelURL = Bundle.main.url(forResource: "RecipeRecommender", withExtension: "mlmodelc") {
            self.mlRecommender = try? MLRecommender<String>(contentsOf: modelURL)
        } else {
            self.mlRecommender = nil
        }
        
        setupBindings()
    }
    
    private func setupBindings() {
        // 사용자 선호도 변경 감지
        userPreferences.$dietaryRestrictions
            .combineLatest(
                userPreferences.$allergies,
                userPreferences.$cuisinePreferences
            )
            .debounce(for: .seconds(1), scheduler: RunLoop.main)
            .sink { [weak self] _ in
                Task {
                    await self?.updateRecommendations()
                }
            }
            .store(in: &cancellables)
    }
    
    func recommendRecipes(basedOn ingredients: [RecognizedIngredient]) async -> [Recipe] {
        var recommendations: [Recipe] = []
        
        // 1. 재료 기반 매칭
        let ingredientNames = ingredients.map { $0.name }
        let matchingRecipes = await recipeDatabase.findRecipes(withIngredients: ingredientNames)
        
        // 2. ML 모델 예측
        if let recommender = mlRecommender {
            let mlRecommendations = getMLRecommendations(
                for: ingredientNames,
                userHistory: userPreferences.cookingHistory
            )
            recommendations.append(contentsOf: mlRecommendations)
        }
        
        // 3. 영양 균형 고려
        let nutritionBalanced = filterByNutrition(
            recipes: matchingRecipes,
            targetNutrition: userPreferences.nutritionGoals
        )
        recommendations.append(contentsOf: nutritionBalanced)
        
        // 4. 난이도 및 시간 필터링
        recommendations = recommendations.filter { recipe in
            recipe.difficulty <= userPreferences.maxDifficulty &&
            recipe.cookingTime <= userPreferences.maxCookingTime
        }
        
        // 5. 알레르기 및 식단 제한 필터링
        recommendations = filterByDietaryRestrictions(recommendations)
        
        // 6. 점수 계산 및 정렬
        let scoredRecipes = recommendations.map { recipe -> (Recipe, Double) in
            let score = calculateRecommendationScore(
                recipe: recipe,
                availableIngredients: ingredients,
                userPreferences: userPreferences
            )
            return (recipe, score)
        }
        
        return scoredRecipes
            .sorted { $0.1 > $1.1 }
            .map { $0.0 }
            .prefix(20)
            .map { $0 }
    }
    
    private func getMLRecommendations(
        for ingredients: [String],
        userHistory: [CookingHistory]
    ) -> [Recipe] {
        guard let recommender = mlRecommender else { return [] }
        
        // 사용자 히스토리를 학습 데이터로 변환
        let trainingData = userHistory.map { history in
            return [
                "userId": userPreferences.userId,
                "recipeId": history.recipeId,
                "rating": history.rating,
                "cookingDate": history.date.timeIntervalSince1970
            ]
        }
        
        // 예측 수행
        do {
            let predictions = try recommender.recommendations(
                from: trainingData,
                k: 10
            )
            
            return predictions.compactMap { prediction in
                recipeDatabase.getRecipe(by: prediction.item)
            }
        } catch {
            print("ML 추천 실패: \(error)")
            return []
        }
    }
    
    private func calculateRecommendationScore(
        recipe: Recipe,
        availableIngredients: [RecognizedIngredient],
        userPreferences: UserPreferences
    ) -> Double {
        var score = 0.0
        
        // 재료 매칭 점수 (40%)
        let ingredientMatchRatio = calculateIngredientMatch(
            recipe: recipe,
            available: availableIngredients
        )
        score += ingredientMatchRatio * 40
        
        // 선호 요리 스타일 점수 (20%)
        if userPreferences.cuisinePreferences.contains(recipe.cuisine) {
            score += 20
        }
        
        // 영양 목표 적합도 (20%)
        let nutritionScore = calculateNutritionScore(
            recipe: recipe,
            goals: userPreferences.nutritionGoals
        )
        score += nutritionScore * 20
        
        // 인기도 점수 (10%)
        score += min(recipe.rating * 2, 10)
        
        // 계절성 점수 (10%)
        if isSeasonalRecipe(recipe) {
            score += 10
        }
        
        return score
    }
    
    private func calculateIngredientMatch(
        recipe: Recipe,
        available: [RecognizedIngredient]
    ) -> Double {
        let recipeIngredients = Set(recipe.ingredients.map { $0.name.lowercased() })
        let availableIngredients = Set(available.map { $0.name.lowercased() })
        
        let intersection = recipeIngredients.intersection(availableIngredients)
        
        return Double(intersection.count) / Double(recipeIngredients.count)
    }
    
    private func filterByDietaryRestrictions(_ recipes: [Recipe]) -> [Recipe] {
        return recipes.filter { recipe in
            // 알레르기 체크
            let hasAllergen = userPreferences.allergies.contains { allergen in
                recipe.allergens.contains(allergen)
            }
            if hasAllergen { return false }
            
            // 식단 제한 체크
            for restriction in userPreferences.dietaryRestrictions {
                switch restriction {
                case .vegetarian:
                    if recipe.tags.contains("meat") { return false }
                case .vegan:
                    if recipe.tags.contains("dairy") || recipe.tags.contains("egg") { return false }
                case .glutenFree:
                    if recipe.tags.contains("gluten") { return false }
                case .keto:
                    if recipe.nutrition.carbs > 20 { return false }
                case .halal:
                    if recipe.tags.contains("pork") || recipe.tags.contains("alcohol") { return false }
                default:
                    break
                }
            }
            
            return true
        }
    }
    
    private func isSeasonalRecipe(_ recipe: Recipe) -> Bool {
        let currentMonth = Calendar.current.component(.month, from: Date())
        let season = getSeason(for: currentMonth)
        
        return recipe.tags.contains(season.rawValue)
    }
    
    private func getSeason(for month: Int) -> Season {
        switch month {
        case 3...5: return .spring
        case 6...8: return .summer
        case 9...11: return .fall
        default: return .winter
        }
    }
}

// MARK: - Supporting Types

enum Season: String {
    case spring = "봄"
    case summer = "여름"
    case fall = "가을"
    case winter = "겨울"
}

struct RecipeSuggestion {
    let recipe: Recipe
    let reason: String
    let matchPercentage: Double
    let missingIngredients: [String]
    let estimatedCost: Double
}

struct CookingHistory {
    let recipeId: String
    let date: Date
    let rating: Double
    let notes: String?
}

enum DietaryRestriction: String, CaseIterable {
    case vegetarian = "채식"
    case vegan = "비건"
    case glutenFree = "글루텐프리"
    case dairyFree = "유제품프리"
    case keto = "키토"
    case paleo = "팔레오"
    case halal = "할랄"
    case kosher = "코셔"
}
```

### 4단계: 음성 요리 어시스턴트

**Core/AI/CookingAssistant.swift**:

```swift
import Speech
import AVFoundation
import NaturalLanguage

class CookingAssistant: NSObject, ObservableObject {
    @Published var isListening = false
    @Published var currentStep: Int = 0
    @Published var spokenCommand: String = ""
    @Published var assistantResponse: String = ""
    
    private let speechRecognizer = SFSpeechRecognizer(locale: Locale(identifier: "ko-KR"))
    private let audioEngine = AVAudioEngine()
    private var recognitionRequest: SFSpeechAudioBufferRecognitionRequest?
    private var recognitionTask: SFSpeechRecognitionTask?
    private let synthesizer = AVSpeechSynthesizer()
    
    private var recipe: Recipe?
    private var timer: Timer?
    private var activeTimers: [CookingTimer] = []
    
    override init() {
        super.init()
        synthesizer.delegate = self
        requestSpeechAuthorization()
    }
    
    func startCookingMode(with recipe: Recipe) {
        self.recipe = recipe
        self.currentStep = 0
        
        speak("요리를 시작합니다. \(recipe.name) 레시피입니다. 준비되면 '시작'이라고 말씀해주세요.")
        startListening()
    }
    
    func startListening() {
        guard !isListening else { return }
        
        do {
            try startAudioEngine()
            isListening = true
        } catch {
            print("음성 인식 시작 실패: \(error)")
        }
    }
    
    private func startAudioEngine() throws {
        // 오디오 세션 설정
        let audioSession = AVAudioSession.sharedInstance()
        try audioSession.setCategory(.playAndRecord, mode: .default)
        try audioSession.setActive(true, options: .notifyOthersOnDeactivation)
        
        recognitionRequest = SFSpeechAudioBufferRecognitionRequest()
        guard let recognitionRequest = recognitionRequest else { return }
        
        recognitionRequest.shouldReportPartialResults = true
        recognitionRequest.requiresOnDeviceRecognition = false
        
        let inputNode = audioEngine.inputNode
        let recordingFormat = inputNode.outputFormat(forBus: 0)
        
        inputNode.installTap(onBus: 0, bufferSize: 1024, format: recordingFormat) { buffer, _ in
            recognitionRequest.append(buffer)
        }
        
        audioEngine.prepare()
        try audioEngine.start()
        
        recognitionTask = speechRecognizer?.recognitionTask(with: recognitionRequest) { [weak self] result, error in
            if let result = result {
                self?.processCommand(result.bestTranscription.formattedString)
            }
            
            if error != nil || result?.isFinal == true {
                self?.stopListening()
            }
        }
    }
    
    private func processCommand(_ command: String) {
        spokenCommand = command.lowercased()
        
        // NLP로 명령 분석
        let tagger = NLTagger(tagSchemes: [.lexicalClass])
        tagger.string = spokenCommand
        
        if spokenCommand.contains("시작") || spokenCommand.contains("다음") {
            nextStep()
        } else if spokenCommand.contains("이전") || spokenCommand.contains("전") {
            previousStep()
        } else if spokenCommand.contains("반복") || spokenCommand.contains("다시") {
            repeatCurrentStep()
        } else if spokenCommand.contains("타이머") {
            handleTimerCommand()
        } else if spokenCommand.contains("얼마나") || spokenCommand.contains("몇") {
            handleQuantityQuestion()
        } else if spokenCommand.contains("도움") || spokenCommand.contains("뭐") {
            provideHelp()
        } else if spokenCommand.contains("정지") || spokenCommand.contains("멈춰") {
            pauseCooking()
        } else if spokenCommand.contains("재료") {
            listIngredients()
        } else if spokenCommand.contains("온도") {
            handleTemperatureQuery()
        }
    }
    
    private func nextStep() {
        guard let recipe = recipe else { return }
        
        if currentStep < recipe.steps.count - 1 {
            currentStep += 1
            let step = recipe.steps[currentStep]
            
            var announcement = "단계 \(currentStep + 1). \(step.description)"
            
            // 타이머가 필요한 경우
            if let duration = step.duration {
                announcement += " 시간은 \(formatDuration(duration))입니다."
                
                if step.requiresTimer {
                    announcement += " 타이머를 시작하려면 '타이머 시작'이라고 말씀해주세요."
                }
            }
            
            // 온도 정보
            if let temperature = step.temperature {
                announcement += " 온도는 \(temperature)도입니다."
            }
            
            speak(announcement)
            
            // 중요 단계 알림
            if step.isCritical {
                sendNotification(title: "중요 단계", body: step.description)
            }
        } else {
            speak("요리가 완료되었습니다. 맛있게 드세요!")
            stopCookingMode()
        }
    }
    
    private func handleTimerCommand() {
        guard let recipe = recipe,
              currentStep < recipe.steps.count else { return }
        
        let step = recipe.steps[currentStep]
        
        if spokenCommand.contains("시작") {
            if let duration = step.duration {
                startTimer(duration: duration, name: step.timerName ?? "타이머")
                speak("\(formatDuration(duration)) 타이머를 시작했습니다.")
            }
        } else if spokenCommand.contains("확인") || spokenCommand.contains("남은") {
            checkTimers()
        } else if spokenCommand.contains("취소") || spokenCommand.contains("중지") {
            cancelTimer()
        }
    }
    
    private func startTimer(duration: TimeInterval, name: String) {
        let timer = CookingTimer(
            id: UUID(),
            name: name,
            duration: duration,
            startTime: Date()
        )
        
        activeTimers.append(timer)
        
        // 타이머 종료 스케줄
        DispatchQueue.main.asyncAfter(deadline: .now() + duration) { [weak self] in
            self?.timerCompleted(timer)
        }
    }
    
    private func timerCompleted(_ timer: CookingTimer) {
        speak("\(timer.name) 타이머가 완료되었습니다.")
        
        // 햅틱 피드백
        let generator = UINotificationFeedbackGenerator()
        generator.notificationOccurred(.success)
        
        // 알림
        sendNotification(
            title: "타이머 완료",
            body: "\(timer.name) 시간이 끝났습니다"
        )
        
        activeTimers.removeAll { $0.id == timer.id }
    }
    
    private func speak(_ text: String) {
        assistantResponse = text
        
        let utterance = AVSpeechUtterance(string: text)
        utterance.voice = AVSpeechSynthesisVoice(language: "ko-KR")
        utterance.rate = 0.5
        utterance.pitchMultiplier = 1.0
        utterance.volume = 1.0
        
        synthesizer.speak(utterance)
    }
    
    private func stopListening() {
        audioEngine.stop()
        audioEngine.inputNode.removeTap(onBus: 0)
        recognitionRequest?.endAudio()
        recognitionTask?.cancel()
        
        recognitionRequest = nil
        recognitionTask = nil
        isListening = false
    }
}

// MARK: - AVSpeechSynthesizerDelegate

extension CookingAssistant: AVSpeechSynthesizerDelegate {
    func speechSynthesizer(_ synthesizer: AVSpeechSynthesizer, didFinish utterance: AVSpeechUtterance) {
        // 말하기 완료 후 다시 듣기 시작
        if recipe != nil {
            startListening()
        }
    }
}

// MARK: - Supporting Types

struct CookingTimer: Identifiable {
    let id: UUID
    let name: String
    let duration: TimeInterval
    let startTime: Date
    
    var remainingTime: TimeInterval {
        max(0, duration - Date().timeIntervalSince(startTime))
    }
    
    var isExpired: Bool {
        remainingTime <= 0
    }
}

extension Recipe {
    struct Step {
        let number: Int
        let description: String
        let duration: TimeInterval?
        let temperature: Int?
        let requiresTimer: Bool
        let isCritical: Bool
        let timerName: String?
        let tips: [String]
        let mediaURL: URL?
    }
}
```

---

## 🎯 여기서 배운 것

### 1. **Core ML 통합**
- Vision 프레임워크
- 이미지 분류
- 객체 인식
- 텍스트 추출

### 2. **머신러닝 모델**
- CreateML 활용
- 추천 시스템
- 영양 예측
- 패턴 학습

### 3. **음성 인터페이스**
- Speech Recognition
- 자연어 처리
- 음성 합성
- 핸즈프리 인터랙션

### 4. **AI 기반 UX**
- 스마트 제안
- 개인화 추천
- 컨텍스트 인식
- 적응형 인터페이스

---

## 🎉 성공 확인

**AI 기능 체크리스트**:
- [ ] 카메라로 재료 인식 가능
- [ ] 영양 정보 자동 계산
- [ ] 개인 맞춤 레시피 추천
- [ ] 음성으로 요리 단계 안내
- [ ] 스마트 타이머 작동

---

**완벽합니다! AI 기반 스마트 레시피 앱의 기초가 완성되었습니다.**