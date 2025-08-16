# RecipeBox: 레시피 관리 시스템

## 레시피 CRUD with SwiftData

```swift
import SwiftUI
import SwiftData
import PhotosUI

// MARK: - Data Models
@Model
final class Recipe {
    var id: UUID = UUID()
    var title: String = ""
    var koreanTitle: String = ""
    var description_text: String = ""
    var cuisine: CuisineType = .korean
    var difficulty: Difficulty = .medium
    var prepTime: Int = 0
    var cookTime: Int = 0
    var servings: Int = 4
    var calories: Int = 0
    var imageData: Data?
    var createdAt: Date = Date()
    var updatedAt: Date = Date()
    var isFavorite: Bool = false
    var rating: Double = 0.0
    var viewCount: Int = 0
    
    @Relationship(deleteRule: .cascade)
    var ingredients: [Ingredient] = []
    
    @Relationship(deleteRule: .cascade)
    var instructions: [Instruction] = []
    
    @Relationship(deleteRule: .cascade)
    var nutritionFacts: NutritionFacts?
    
    @Relationship(inverse: \RecipeCollection.recipes)
    var collections: [RecipeCollection] = []
    
    @Relationship(deleteRule: .cascade)
    var reviews: [Review] = []
    
    @Relationship
    var tags: [Tag] = []
    
    @Relationship
    var author: ChefProfile?
    
    var totalTime: Int {
        prepTime + cookTime
    }
    
    var averageRating: Double {
        guard !reviews.isEmpty else { return 0.0 }
        let total = reviews.reduce(0) { $0 + $1.rating }
        return total / Double(reviews.count)
    }
    
    init(title: String, koreanTitle: String = "", cuisine: CuisineType = .korean) {
        self.title = title
        self.koreanTitle = koreanTitle
        self.cuisine = cuisine
    }
}

@Model
final class Ingredient {
    var id: UUID = UUID()
    var name: String = ""
    var koreanName: String = ""
    var amount: Double = 0.0
    var unit: MeasurementUnit = .gram
    var category: IngredientCategory = .vegetable
    var isOptional: Bool = false
    var substitutes: [String] = []
    var price: Int = 0 // 원화
    var calories: Int = 0
    
    init(name: String, amount: Double, unit: MeasurementUnit) {
        self.name = name
        self.amount = amount
        self.unit = unit
    }
}

@Model
final class Instruction {
    var id: UUID = UUID()
    var stepNumber: Int = 0
    var text: String = ""
    var imageData: Data?
    var videoURL: String?
    var duration: Int = 0 // 분
    var tips: [String] = []
    var temperature: Int? // 섭씨
    
    init(stepNumber: Int, text: String) {
        self.stepNumber = stepNumber
        self.text = text
    }
}

@Model
final class NutritionFacts {
    var calories: Int = 0
    var protein: Double = 0.0
    var carbs: Double = 0.0
    var fat: Double = 0.0
    var fiber: Double = 0.0
    var sugar: Double = 0.0
    var sodium: Int = 0
    var cholesterol: Int = 0
    var vitamins: [String: Double] = [:]
    var minerals: [String: Double] = [:]
    
    init() {}
}

@Model
final class RecipeCollection {
    var id: UUID = UUID()
    var name: String = ""
    var description_text: String = ""
    var imageData: Data?
    var isPublic: Bool = false
    var createdAt: Date = Date()
    
    @Relationship
    var recipes: [Recipe] = []
    
    @Relationship
    var owner: ChefProfile?
    
    init(name: String) {
        self.name = name
    }
}

@Model
final class Tag {
    var id: UUID = UUID()
    var name: String = ""
    var koreanName: String = ""
    var category: TagCategory = .diet
    var colorHex: String = "#FF6B6B"
    
    init(name: String, koreanName: String = "", category: TagCategory = .diet) {
        self.name = name
        self.koreanName = koreanName
        self.category = category
    }
}

// MARK: - Enums
enum CuisineType: String, CaseIterable, Codable {
    case korean = "한식"
    case chinese = "중식"
    case japanese = "일식"
    case western = "양식"
    case italian = "이탈리안"
    case mexican = "멕시칸"
    case thai = "태국"
    case vietnamese = "베트남"
    case indian = "인도"
    case fusion = "퓨전"
    
    var icon: String {
        switch self {
        case .korean: return "🇰🇷"
        case .chinese: return "🇨🇳"
        case .japanese: return "🇯🇵"
        case .western: return "🍔"
        case .italian: return "🇮🇹"
        case .mexican: return "🇲🇽"
        case .thai: return "🇹🇭"
        case .vietnamese: return "🇻🇳"
        case .indian: return "🇮🇳"
        case .fusion: return "🌏"
        }
    }
}

enum Difficulty: String, CaseIterable, Codable {
    case easy = "쉬움"
    case medium = "보통"
    case hard = "어려움"
    case expert = "전문가"
    
    var color: Color {
        switch self {
        case .easy: return .green
        case .medium: return .orange
        case .hard: return .red
        case .expert: return .purple
        }
    }
}

enum MeasurementUnit: String, CaseIterable, Codable {
    case gram = "g"
    case kilogram = "kg"
    case milliliter = "ml"
    case liter = "L"
    case tablespoon = "큰술"
    case teaspoon = "작은술"
    case cup = "컵"
    case piece = "개"
    case slice = "조각"
    case pinch = "꼬집"
    case handful = "줌"
    case clove = "쪽"
    
    var conversionToGram: Double? {
        switch self {
        case .gram: return 1.0
        case .kilogram: return 1000.0
        case .milliliter: return 1.0 // 물 기준
        case .liter: return 1000.0
        case .tablespoon: return 15.0
        case .teaspoon: return 5.0
        case .cup: return 240.0
        default: return nil
        }
    }
}

enum IngredientCategory: String, CaseIterable, Codable {
    case meat = "육류"
    case seafood = "해산물"
    case vegetable = "채소"
    case fruit = "과일"
    case dairy = "유제품"
    case grain = "곡물"
    case seasoning = "양념"
    case sauce = "소스"
    case oil = "기름"
    case other = "기타"
}

enum TagCategory: String, CaseIterable, Codable {
    case diet = "다이어트"
    case allergy = "알레르기"
    case mealType = "식사유형"
    case season = "계절"
    case special = "특별한날"
}

// MARK: - Recipe Manager
@Observable
final class RecipeManager {
    private let modelContext: ModelContext
    
    init(modelContext: ModelContext) {
        self.modelContext = modelContext
    }
    
    // MARK: - CRUD Operations
    func createRecipe(_ recipe: Recipe) async throws {
        recipe.createdAt = Date()
        recipe.updatedAt = Date()
        modelContext.insert(recipe)
        try modelContext.save()
    }
    
    func updateRecipe(_ recipe: Recipe) async throws {
        recipe.updatedAt = Date()
        try modelContext.save()
    }
    
    func deleteRecipe(_ recipe: Recipe) async throws {
        modelContext.delete(recipe)
        try modelContext.save()
    }
    
    func duplicateRecipe(_ recipe: Recipe) async throws -> Recipe {
        let newRecipe = Recipe(
            title: "\(recipe.title) (복사본)",
            koreanTitle: "\(recipe.koreanTitle) (복사본)",
            cuisine: recipe.cuisine
        )
        
        newRecipe.description_text = recipe.description_text
        newRecipe.difficulty = recipe.difficulty
        newRecipe.prepTime = recipe.prepTime
        newRecipe.cookTime = recipe.cookTime
        newRecipe.servings = recipe.servings
        newRecipe.calories = recipe.calories
        newRecipe.imageData = recipe.imageData
        
        // 재료 복사
        for ingredient in recipe.ingredients {
            let newIngredient = Ingredient(
                name: ingredient.name,
                amount: ingredient.amount,
                unit: ingredient.unit
            )
            newIngredient.koreanName = ingredient.koreanName
            newIngredient.category = ingredient.category
            newIngredient.isOptional = ingredient.isOptional
            newIngredient.substitutes = ingredient.substitutes
            newRecipe.ingredients.append(newIngredient)
        }
        
        // 조리법 복사
        for instruction in recipe.instructions {
            let newInstruction = Instruction(
                stepNumber: instruction.stepNumber,
                text: instruction.text
            )
            newInstruction.imageData = instruction.imageData
            newInstruction.videoURL = instruction.videoURL
            newInstruction.duration = instruction.duration
            newInstruction.tips = instruction.tips
            newInstruction.temperature = instruction.temperature
            newRecipe.instructions.append(newInstruction)
        }
        
        modelContext.insert(newRecipe)
        try modelContext.save()
        return newRecipe
    }
    
    // MARK: - Recipe Import/Export
    func importFromURL(_ url: URL) async throws -> Recipe {
        let (data, _) = try await URLSession.shared.data(from: url)
        let html = String(data: data, encoding: .utf8) ?? ""
        
        // Parse recipe from HTML using structured data
        let recipe = try parseRecipeFromHTML(html)
        modelContext.insert(recipe)
        try modelContext.save()
        return recipe
    }
    
    func importFromText(_ text: String) throws -> Recipe {
        let recipe = Recipe(title: "", koreanTitle: "")
        let lines = text.components(separatedBy: .newlines)
        
        var isIngredientsSection = false
        var isInstructionsSection = false
        var stepNumber = 1
        
        for line in lines {
            let trimmed = line.trimmingCharacters(in: .whitespacesAndNewlines)
            guard !trimmed.isEmpty else { continue }
            
            if trimmed.lowercased().contains("재료") || trimmed.lowercased().contains("ingredient") {
                isIngredientsSection = true
                isInstructionsSection = false
                continue
            }
            
            if trimmed.lowercased().contains("조리") || trimmed.lowercased().contains("instruction") {
                isIngredientsSection = false
                isInstructionsSection = true
                continue
            }
            
            if recipe.title.isEmpty {
                recipe.title = trimmed
                recipe.koreanTitle = trimmed
            } else if isIngredientsSection {
                let ingredient = parseIngredientLine(trimmed)
                recipe.ingredients.append(ingredient)
            } else if isInstructionsSection {
                let instruction = Instruction(stepNumber: stepNumber, text: trimmed)
                recipe.instructions.append(instruction)
                stepNumber += 1
            }
        }
        
        modelContext.insert(recipe)
        try modelContext.save()
        return recipe
    }
    
    func exportToPDF(_ recipe: Recipe) -> Data {
        let pdfMetaData = [
            kCGPDFContextCreator: "RecipeBox",
            kCGPDFContextAuthor: recipe.author?.name ?? "RecipeBox",
            kCGPDFContextTitle: recipe.title
        ]
        
        let format = UIGraphicsPDFRendererFormat()
        format.documentInfo = pdfMetaData as [String: Any]
        
        let pageRect = CGRect(x: 0, y: 0, width: 595, height: 842) // A4
        let renderer = UIGraphicsPDFRenderer(bounds: pageRect, format: format)
        
        let data = renderer.pdfData { context in
            context.beginPage()
            
            // Title
            let titleAttributes: [NSAttributedString.Key: Any] = [
                .font: UIFont.systemFont(ofSize: 24, weight: .bold)
            ]
            recipe.title.draw(at: CGPoint(x: 50, y: 50), withAttributes: titleAttributes)
            
            // Korean Title
            if !recipe.koreanTitle.isEmpty {
                let koreanTitleAttributes: [NSAttributedString.Key: Any] = [
                    .font: UIFont.systemFont(ofSize: 18),
                    .foregroundColor: UIColor.gray
                ]
                recipe.koreanTitle.draw(at: CGPoint(x: 50, y: 80), withAttributes: koreanTitleAttributes)
            }
            
            var yPosition: CGFloat = 120
            
            // Recipe Info
            let infoAttributes: [NSAttributedString.Key: Any] = [
                .font: UIFont.systemFont(ofSize: 12)
            ]
            
            let info = """
            요리 시간: \(recipe.totalTime)분 | 난이도: \(recipe.difficulty.rawValue) | \(recipe.servings)인분
            칼로리: \(recipe.calories)kcal
            """
            info.draw(at: CGPoint(x: 50, y: yPosition), withAttributes: infoAttributes)
            yPosition += 40
            
            // Ingredients
            let sectionAttributes: [NSAttributedString.Key: Any] = [
                .font: UIFont.systemFont(ofSize: 16, weight: .semibold)
            ]
            "재료".draw(at: CGPoint(x: 50, y: yPosition), withAttributes: sectionAttributes)
            yPosition += 25
            
            let itemAttributes: [NSAttributedString.Key: Any] = [
                .font: UIFont.systemFont(ofSize: 11)
            ]
            
            for ingredient in recipe.ingredients {
                let text = "• \(ingredient.koreanName.isEmpty ? ingredient.name : ingredient.koreanName): \(ingredient.amount) \(ingredient.unit.rawValue)"
                text.draw(at: CGPoint(x: 60, y: yPosition), withAttributes: itemAttributes)
                yPosition += 20
                
                if yPosition > 750 {
                    context.beginPage()
                    yPosition = 50
                }
            }
            
            yPosition += 20
            
            // Instructions
            "조리법".draw(at: CGPoint(x: 50, y: yPosition), withAttributes: sectionAttributes)
            yPosition += 25
            
            for instruction in recipe.instructions {
                let text = "\(instruction.stepNumber). \(instruction.text)"
                let textRect = CGRect(x: 60, y: yPosition, width: 485, height: 100)
                text.draw(in: textRect, withAttributes: itemAttributes)
                yPosition += min(100, text.boundingRect(
                    with: CGSize(width: 485, height: .infinity),
                    options: .usesLineFragmentOrigin,
                    attributes: itemAttributes,
                    context: nil
                ).height + 10)
                
                if yPosition > 750 {
                    context.beginPage()
                    yPosition = 50
                }
            }
        }
        
        return data
    }
    
    private func parseRecipeFromHTML(_ html: String) throws -> Recipe {
        // JSON-LD 파싱
        if let jsonLDRange = html.range(of: "<script type=\"application/ld+json\">") {
            let startIndex = html.index(jsonLDRange.upperBound, offsetBy: 0)
            if let endRange = html[startIndex...].range(of: "</script>") {
                let jsonString = String(html[startIndex..<endRange.lowerBound])
                if let data = jsonString.data(using: .utf8) {
                    let decoder = JSONDecoder()
                    if let schema = try? decoder.decode(RecipeSchema.self, from: data) {
                        return schema.toRecipe()
                    }
                }
            }
        }
        
        // Fallback HTML 파싱
        let recipe = Recipe(title: "Imported Recipe", koreanTitle: "가져온 레시피")
        // 간단한 HTML 파싱 로직
        return recipe
    }
    
    private func parseIngredientLine(_ line: String) -> Ingredient {
        // 재료 라인 파싱 로직
        let components = line.components(separatedBy: CharacterSet(charactersIn: ":-"))
        let name = components.first?.trimmingCharacters(in: .whitespaces) ?? line
        
        let ingredient = Ingredient(name: name, amount: 1, unit: .piece)
        
        // 숫자와 단위 추출
        let pattern = #"(\d+(?:\.\d+)?)\s*([가-힣a-zA-Z]+)"#
        if let regex = try? NSRegularExpression(pattern: pattern) {
            let matches = regex.matches(in: line, range: NSRange(line.startIndex..., in: line))
            if let match = matches.first {
                if let amountRange = Range(match.range(at: 1), in: line) {
                    ingredient.amount = Double(line[amountRange]) ?? 1
                }
                if let unitRange = Range(match.range(at: 2), in: line) {
                    let unitString = String(line[unitRange])
                    ingredient.unit = MeasurementUnit.allCases.first { $0.rawValue == unitString } ?? .piece
                }
            }
        }
        
        return ingredient
    }
}

// MARK: - Recipe Search & Filter
struct RecipeSearchFilter {
    var searchText: String = ""
    var cuisines: Set<CuisineType> = []
    var difficulties: Set<Difficulty> = []
    var maxTime: Int?
    var minRating: Double = 0
    var tags: Set<Tag> = []
    var isVegetarian: Bool = false
    var isVegan: Bool = false
    var isGlutenFree: Bool = false
    var maxCalories: Int?
    var servings: Int?
    
    func matches(_ recipe: Recipe) -> Bool {
        // 검색어 확인
        if !searchText.isEmpty {
            let searchLower = searchText.lowercased()
            let titleMatch = recipe.title.lowercased().contains(searchLower)
            let koreanTitleMatch = recipe.koreanTitle.lowercased().contains(searchLower)
            let descriptionMatch = recipe.description_text.lowercased().contains(searchLower)
            
            if !titleMatch && !koreanTitleMatch && !descriptionMatch {
                return false
            }
        }
        
        // 요리 종류
        if !cuisines.isEmpty && !cuisines.contains(recipe.cuisine) {
            return false
        }
        
        // 난이도
        if !difficulties.isEmpty && !difficulties.contains(recipe.difficulty) {
            return false
        }
        
        // 시간
        if let maxTime = maxTime, recipe.totalTime > maxTime {
            return false
        }
        
        // 평점
        if recipe.averageRating < minRating {
            return false
        }
        
        // 칼로리
        if let maxCalories = maxCalories, recipe.calories > maxCalories {
            return false
        }
        
        // 태그
        if !tags.isEmpty {
            let recipeTags = Set(recipe.tags)
            if tags.intersection(recipeTags).isEmpty {
                return false
            }
        }
        
        return true
    }
}

// MARK: - Recipe Search View
struct RecipeSearchView: View {
    @Environment(\.modelContext) private var modelContext
    @Query private var allRecipes: [Recipe]
    @State private var filter = RecipeSearchFilter()
    @State private var showFilter = false
    @State private var sortOption: SortOption = .newest
    
    enum SortOption: String, CaseIterable {
        case newest = "최신순"
        case popular = "인기순"
        case rating = "평점순"
        case fastest = "조리시간순"
        
        func sort(_ recipes: [Recipe]) -> [Recipe] {
            switch self {
            case .newest:
                return recipes.sorted { $0.createdAt > $1.createdAt }
            case .popular:
                return recipes.sorted { $0.viewCount > $1.viewCount }
            case .rating:
                return recipes.sorted { $0.averageRating > $1.averageRating }
            case .fastest:
                return recipes.sorted { $0.totalTime < $1.totalTime }
            }
        }
    }
    
    var filteredRecipes: [Recipe] {
        let filtered = allRecipes.filter { filter.matches($0) }
        return sortOption.sort(filtered)
    }
    
    var body: some View {
        NavigationStack {
            ScrollView {
                VStack(spacing: 16) {
                    // Search Bar
                    HStack {
                        Image(systemName: "magnifyingglass")
                            .foregroundColor(.secondary)
                        
                        TextField("레시피 검색...", text: $filter.searchText)
                            .textFieldStyle(RoundedBorderTextFieldStyle())
                        
                        Button {
                            showFilter.toggle()
                        } label: {
                            Image(systemName: "slider.horizontal.3")
                                .foregroundColor(hasActiveFilters ? .blue : .secondary)
                        }
                    }
                    .padding(.horizontal)
                    
                    // Sort Options
                    ScrollView(.horizontal, showsIndicators: false) {
                        HStack(spacing: 12) {
                            ForEach(SortOption.allCases, id: \.self) { option in
                                Button {
                                    sortOption = option
                                } label: {
                                    Text(option.rawValue)
                                        .font(.subheadline)
                                        .padding(.horizontal, 16)
                                        .padding(.vertical, 8)
                                        .background(sortOption == option ? Color.blue : Color(.systemGray6))
                                        .foregroundColor(sortOption == option ? .white : .primary)
                                        .cornerRadius(20)
                                }
                            }
                        }
                        .padding(.horizontal)
                    }
                    
                    // Active Filters
                    if hasActiveFilters {
                        ScrollView(.horizontal, showsIndicators: false) {
                            HStack(spacing: 8) {
                                ForEach(Array(filter.cuisines), id: \.self) { cuisine in
                                    FilterChip(text: cuisine.rawValue) {
                                        filter.cuisines.remove(cuisine)
                                    }
                                }
                                
                                ForEach(Array(filter.difficulties), id: \.self) { difficulty in
                                    FilterChip(text: difficulty.rawValue) {
                                        filter.difficulties.remove(difficulty)
                                    }
                                }
                                
                                if let maxTime = filter.maxTime {
                                    FilterChip(text: "~\(maxTime)분") {
                                        filter.maxTime = nil
                                    }
                                }
                                
                                if filter.minRating > 0 {
                                    FilterChip(text: "★\(String(format: "%.1f", filter.minRating))+") {
                                        filter.minRating = 0
                                    }
                                }
                            }
                            .padding(.horizontal)
                        }
                    }
                    
                    // Results
                    if filteredRecipes.isEmpty {
                        VStack(spacing: 20) {
                            Image(systemName: "magnifyingglass")
                                .font(.system(size: 60))
                                .foregroundColor(.secondary)
                            
                            Text("검색 결과가 없습니다")
                                .font(.headline)
                            
                            Text("다른 검색어나 필터를 시도해보세요")
                                .font(.subheadline)
                                .foregroundColor(.secondary)
                        }
                        .padding(.top, 100)
                    } else {
                        LazyVStack(spacing: 16) {
                            ForEach(filteredRecipes) { recipe in
                                NavigationLink(destination: RecipeDetailView(recipe: recipe)) {
                                    RecipeCard(recipe: recipe)
                                }
                                .buttonStyle(PlainButtonStyle())
                            }
                        }
                        .padding(.horizontal)
                    }
                }
            }
            .navigationTitle("레시피 검색")
            .sheet(isPresented: $showFilter) {
                FilterView(filter: $filter)
            }
        }
    }
    
    var hasActiveFilters: Bool {
        !filter.cuisines.isEmpty ||
        !filter.difficulties.isEmpty ||
        filter.maxTime != nil ||
        filter.minRating > 0 ||
        !filter.tags.isEmpty ||
        filter.isVegetarian ||
        filter.isVegan ||
        filter.isGlutenFree ||
        filter.maxCalories != nil
    }
}

// MARK: - Filter View
struct FilterView: View {
    @Binding var filter: RecipeSearchFilter
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        NavigationStack {
            Form {
                Section("요리 종류") {
                    LazyVGrid(columns: [GridItem(.adaptive(minimum: 100))], spacing: 12) {
                        ForEach(CuisineType.allCases, id: \.self) { cuisine in
                            FilterToggle(
                                text: "\(cuisine.icon) \(cuisine.rawValue)",
                                isSelected: filter.cuisines.contains(cuisine)
                            ) {
                                if filter.cuisines.contains(cuisine) {
                                    filter.cuisines.remove(cuisine)
                                } else {
                                    filter.cuisines.insert(cuisine)
                                }
                            }
                        }
                    }
                }
                
                Section("난이도") {
                    HStack(spacing: 12) {
                        ForEach(Difficulty.allCases, id: \.self) { difficulty in
                            FilterToggle(
                                text: difficulty.rawValue,
                                isSelected: filter.difficulties.contains(difficulty)
                            ) {
                                if filter.difficulties.contains(difficulty) {
                                    filter.difficulties.remove(difficulty)
                                } else {
                                    filter.difficulties.insert(difficulty)
                                }
                            }
                        }
                    }
                }
                
                Section("조리 시간") {
                    HStack {
                        Text("최대")
                        Picker("시간", selection: Binding(
                            get: { filter.maxTime ?? 60 },
                            set: { filter.maxTime = $0 }
                        )) {
                            Text("15분").tag(15)
                            Text("30분").tag(30)
                            Text("45분").tag(45)
                            Text("1시간").tag(60)
                            Text("2시간").tag(120)
                            Text("제한없음").tag(nil as Int?)
                        }
                        .pickerStyle(SegmentedPickerStyle())
                    }
                }
                
                Section("평점") {
                    HStack {
                        Text("최소")
                        HStack(spacing: 4) {
                            ForEach(1...5, id: \.self) { star in
                                Button {
                                    filter.minRating = Double(star)
                                } label: {
                                    Image(systemName: filter.minRating >= Double(star) ? "star.fill" : "star")
                                        .foregroundColor(.yellow)
                                }
                            }
                        }
                        Spacer()
                        if filter.minRating > 0 {
                            Button("초기화") {
                                filter.minRating = 0
                            }
                            .font(.caption)
                        }
                    }
                }
                
                Section("식단 유형") {
                    Toggle("채식주의자", isOn: $filter.isVegetarian)
                    Toggle("비건", isOn: $filter.isVegan)
                    Toggle("글루텐 프리", isOn: $filter.isGlutenFree)
                }
                
                Section("칼로리") {
                    HStack {
                        Text("최대")
                        TextField("칼로리", value: $filter.maxCalories, format: .number)
                            .keyboardType(.numberPad)
                        Text("kcal")
                    }
                }
            }
            .navigationTitle("필터")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .navigationBarLeading) {
                    Button("초기화") {
                        filter = RecipeSearchFilter()
                    }
                }
                
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("적용") {
                        dismiss()
                    }
                }
            }
        }
    }
}

// MARK: - Meal Planning
@Model
final class MealPlan {
    var id: UUID = UUID()
    var date: Date = Date()
    var mealType: MealType = .lunch
    var recipe: Recipe?
    var servings: Int = 1
    var notes: String = ""
    var isCompleted: Bool = false
    
    init(date: Date, mealType: MealType) {
        self.date = date
        self.mealType = mealType
    }
}

enum MealType: String, CaseIterable, Codable {
    case breakfast = "아침"
    case lunch = "점심"
    case dinner = "저녁"
    case snack = "간식"
    
    var icon: String {
        switch self {
        case .breakfast: return "☀️"
        case .lunch: return "🌤"
        case .dinner: return "🌙"
        case .snack: return "🍪"
        }
    }
    
    var defaultTime: Date {
        let calendar = Calendar.current
        let now = Date()
        let components = calendar.dateComponents([.year, .month, .day], from: now)
        
        switch self {
        case .breakfast:
            return calendar.date(bySettingHour: 7, minute: 0, second: 0, of: now) ?? now
        case .lunch:
            return calendar.date(bySettingHour: 12, minute: 0, second: 0, of: now) ?? now
        case .dinner:
            return calendar.date(bySettingHour: 18, minute: 0, second: 0, of: now) ?? now
        case .snack:
            return calendar.date(bySettingHour: 15, minute: 0, second: 0, of: now) ?? now
        }
    }
}

// MARK: - Meal Planning Calendar View
struct MealPlanningView: View {
    @Environment(\.modelContext) private var modelContext
    @Query private var mealPlans: [MealPlan]
    @Query private var recipes: [Recipe]
    @State private var selectedDate = Date()
    @State private var showAddMeal = false
    @State private var draggedRecipe: Recipe?
    
    var calendar: Calendar {
        Calendar.current
    }
    
    var currentWeek: [Date] {
        guard let weekInterval = calendar.dateInterval(of: .weekOfYear, for: selectedDate) else {
            return []
        }
        
        return (0..<7).compactMap { dayOffset in
            calendar.date(byAdding: .day, value: dayOffset, to: weekInterval.start)
        }
    }
    
    func mealsForDate(_ date: Date) -> [MealPlan] {
        mealPlans.filter { calendar.isDate($0.date, inSameDayAs: date) }
            .sorted { $0.mealType.defaultTime < $1.mealType.defaultTime }
    }
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 0) {
                // Week Selector
                WeekSelectorView(selectedDate: $selectedDate)
                    .padding()
                
                // Calendar Grid
                ScrollView {
                    LazyVGrid(columns: [GridItem(.flexible())], spacing: 20) {
                        ForEach(currentWeek, id: \.self) { date in
                            VStack(alignment: .leading, spacing: 12) {
                                // Date Header
                                HStack {
                                    Text(date, format: .dateTime.month(.abbreviated).day())
                                        .font(.headline)
                                    Text(date, format: .dateTime.weekday(.wide))
                                        .font(.subheadline)
                                        .foregroundColor(.secondary)
                                    Spacer()
                                    
                                    if calendar.isDateInToday(date) {
                                        Text("오늘")
                                            .font(.caption)
                                            .padding(.horizontal, 8)
                                            .padding(.vertical, 4)
                                            .background(Color.blue)
                                            .foregroundColor(.white)
                                            .cornerRadius(12)
                                    }
                                }
                                
                                // Meal Slots
                                ForEach(MealType.allCases, id: \.self) { mealType in
                                    MealSlotView(
                                        date: date,
                                        mealType: mealType,
                                        mealPlan: mealsForDate(date).first { $0.mealType == mealType }
                                    )
                                    .onDrop(of: [.text], isTargeted: nil) { providers in
                                        handleDrop(providers: providers, date: date, mealType: mealType)
                                    }
                                }
                            }
                            .padding()
                            .background(Color(.systemGray6))
                            .cornerRadius(12)
                        }
                    }
                    .padding()
                }
                
                // Recipe Drawer
                RecipeDrawer(recipes: recipes, draggedRecipe: $draggedRecipe)
            }
            .navigationTitle("식단 계획")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button {
                        generateShoppingList()
                    } label: {
                        Image(systemName: "cart")
                    }
                }
            }
        }
    }
    
    func handleDrop(providers: [NSItemProvider], date: Date, mealType: MealType) -> Bool {
        // Drop handling for drag and drop
        return true
    }
    
    func generateShoppingList() {
        // Generate shopping list from meal plans
        let startOfWeek = currentWeek.first ?? Date()
        let endOfWeek = currentWeek.last ?? Date()
        
        let weekMealPlans = mealPlans.filter { plan in
            plan.date >= startOfWeek && plan.date <= endOfWeek
        }
        
        var shoppingList: [String: (amount: Double, unit: MeasurementUnit)] = [:]
        
        for plan in weekMealPlans {
            guard let recipe = plan.recipe else { continue }
            
            let scaleFactor = Double(plan.servings) / Double(recipe.servings)
            
            for ingredient in recipe.ingredients {
                let key = ingredient.koreanName.isEmpty ? ingredient.name : ingredient.koreanName
                let scaledAmount = ingredient.amount * scaleFactor
                
                if let existing = shoppingList[key] {
                    // 같은 단위면 합치기
                    if existing.unit == ingredient.unit {
                        shoppingList[key] = (existing.amount + scaledAmount, existing.unit)
                    } else {
                        // 다른 단위면 그램으로 변환 시도
                        if let existingInGram = existing.unit.conversionToGram,
                           let newInGram = ingredient.unit.conversionToGram {
                            let totalGrams = existing.amount * existingInGram + scaledAmount * newInGram
                            shoppingList[key] = (totalGrams, .gram)
                        }
                    }
                } else {
                    shoppingList[key] = (scaledAmount, ingredient.unit)
                }
            }
        }
        
        // Create shopping list document
        createShoppingListDocument(shoppingList)
    }
    
    func createShoppingListDocument(_ list: [String: (amount: Double, unit: MeasurementUnit)]) {
        // Implementation for creating shopping list
    }
}

// MARK: - Nutrition Calculator
struct NutritionCalculator {
    static func calculate(for recipe: Recipe) -> NutritionFacts {
        let nutrition = NutritionFacts()
        
        for ingredient in recipe.ingredients {
            // 재료별 영양 정보 계산
            let ingredientNutrition = getNutritionForIngredient(ingredient)
            nutrition.calories += ingredientNutrition.calories
            nutrition.protein += ingredientNutrition.protein
            nutrition.carbs += ingredientNutrition.carbs
            nutrition.fat += ingredientNutrition.fat
            nutrition.fiber += ingredientNutrition.fiber
            nutrition.sugar += ingredientNutrition.sugar
            nutrition.sodium += ingredientNutrition.sodium
            nutrition.cholesterol += ingredientNutrition.cholesterol
        }
        
        // 1인분 기준으로 계산
        let servings = Double(recipe.servings)
        nutrition.calories = Int(Double(nutrition.calories) / servings)
        nutrition.protein /= servings
        nutrition.carbs /= servings
        nutrition.fat /= servings
        nutrition.fiber /= servings
        nutrition.sugar /= servings
        nutrition.sodium = Int(Double(nutrition.sodium) / servings)
        nutrition.cholesterol = Int(Double(nutrition.cholesterol) / servings)
        
        return nutrition
    }
    
    static func getNutritionForIngredient(_ ingredient: Ingredient) -> NutritionFacts {
        // 한국 식품 영양 데이터베이스 연동
        // 실제로는 API나 로컬 DB에서 가져옴
        let nutrition = NutritionFacts()
        
        // 예시 데이터
        switch ingredient.category {
        case .meat:
            nutrition.calories = Int(ingredient.amount * 2.5)
            nutrition.protein = ingredient.amount * 0.26
            nutrition.fat = ingredient.amount * 0.15
        case .vegetable:
            nutrition.calories = Int(ingredient.amount * 0.25)
            nutrition.carbs = ingredient.amount * 0.05
            nutrition.fiber = ingredient.amount * 0.02
        case .grain:
            nutrition.calories = Int(ingredient.amount * 3.5)
            nutrition.carbs = ingredient.amount * 0.75
            nutrition.protein = ingredient.amount * 0.08
        default:
            nutrition.calories = Int(ingredient.amount * 1.0)
        }
        
        return nutrition
    }
}

// MARK: - Recipe Scaling
struct RecipeScaler {
    static func scale(recipe: Recipe, to servings: Int) -> Recipe {
        let scaleFactor = Double(servings) / Double(recipe.servings)
        
        let scaledRecipe = Recipe(
            title: recipe.title,
            koreanTitle: recipe.koreanTitle,
            cuisine: recipe.cuisine
        )
        
        scaledRecipe.servings = servings
        scaledRecipe.description_text = recipe.description_text
        scaledRecipe.difficulty = recipe.difficulty
        scaledRecipe.prepTime = recipe.prepTime
        scaledRecipe.cookTime = recipe.cookTime
        
        // 재료 조정
        for ingredient in recipe.ingredients {
            let scaledIngredient = Ingredient(
                name: ingredient.name,
                amount: ingredient.amount * scaleFactor,
                unit: ingredient.unit
            )
            scaledIngredient.koreanName = ingredient.koreanName
            scaledIngredient.category = ingredient.category
            scaledRecipe.ingredients.append(scaledIngredient)
        }
        
        // 조리법은 그대로
        scaledRecipe.instructions = recipe.instructions
        
        // 영양 정보 재계산
        scaledRecipe.nutritionFacts = NutritionCalculator.calculate(for: scaledRecipe)
        
        return scaledRecipe
    }
}

// MARK: - Supporting Views
struct RecipeCard: View {
    let recipe: Recipe
    
    var body: some View {
        HStack(spacing: 16) {
            // Image
            if let imageData = recipe.imageData,
               let uiImage = UIImage(data: imageData) {
                Image(uiImage: uiImage)
                    .resizable()
                    .aspectRatio(contentMode: .fill)
                    .frame(width: 100, height: 100)
                    .cornerRadius(12)
            } else {
                RoundedRectangle(cornerRadius: 12)
                    .fill(Color(.systemGray5))
                    .frame(width: 100, height: 100)
                    .overlay(
                        Image(systemName: "photo")
                            .foregroundColor(.secondary)
                    )
            }
            
            // Info
            VStack(alignment: .leading, spacing: 8) {
                Text(recipe.koreanTitle.isEmpty ? recipe.title : recipe.koreanTitle)
                    .font(.headline)
                    .lineLimit(1)
                
                if !recipe.koreanTitle.isEmpty {
                    Text(recipe.title)
                        .font(.caption)
                        .foregroundColor(.secondary)
                        .lineLimit(1)
                }
                
                HStack(spacing: 12) {
                    Label("\(recipe.totalTime)분", systemImage: "clock")
                    Label(recipe.difficulty.rawValue, systemImage: "chart.bar")
                    if recipe.averageRating > 0 {
                        Label(String(format: "%.1f", recipe.averageRating), systemImage: "star.fill")
                            .foregroundColor(.yellow)
                    }
                }
                .font(.caption)
                .foregroundColor(.secondary)
                
                Text(recipe.cuisine.icon + " " + recipe.cuisine.rawValue)
                    .font(.caption)
                    .padding(.horizontal, 8)
                    .padding(.vertical, 4)
                    .background(Color(.systemGray6))
                    .cornerRadius(8)
            }
            
            Spacer()
            
            if recipe.isFavorite {
                Image(systemName: "heart.fill")
                    .foregroundColor(.red)
            }
        }
        .padding()
        .background(Color(.systemBackground))
        .cornerRadius(16)
        .shadow(radius: 2)
    }
}

struct FilterChip: View {
    let text: String
    let onRemove: () -> Void
    
    var body: some View {
        HStack(spacing: 4) {
            Text(text)
                .font(.caption)
            
            Button(action: onRemove) {
                Image(systemName: "xmark.circle.fill")
                    .font(.caption)
            }
        }
        .padding(.horizontal, 12)
        .padding(.vertical, 6)
        .background(Color.blue.opacity(0.1))
        .foregroundColor(.blue)
        .cornerRadius(16)
    }
}

struct FilterToggle: View {
    let text: String
    let isSelected: Bool
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            Text(text)
                .font(.subheadline)
                .padding(.horizontal, 16)
                .padding(.vertical, 8)
                .background(isSelected ? Color.blue : Color(.systemGray6))
                .foregroundColor(isSelected ? .white : .primary)
                .cornerRadius(20)
        }
    }
}

// MARK: - Helper Views
struct WeekSelectorView: View {
    @Binding var selectedDate: Date
    
    var body: some View {
        HStack {
            Button {
                selectedDate = Calendar.current.date(byAdding: .weekOfYear, value: -1, to: selectedDate) ?? selectedDate
            } label: {
                Image(systemName: "chevron.left")
            }
            
            Spacer()
            
            Text(weekRangeText)
                .font(.headline)
            
            Spacer()
            
            Button {
                selectedDate = Calendar.current.date(byAdding: .weekOfYear, value: 1, to: selectedDate) ?? selectedDate
            } label: {
                Image(systemName: "chevron.right")
            }
        }
    }
    
    var weekRangeText: String {
        let calendar = Calendar.current
        guard let weekInterval = calendar.dateInterval(of: .weekOfYear, for: selectedDate) else {
            return ""
        }
        
        let formatter = DateFormatter()
        formatter.locale = Locale(identifier: "ko_KR")
        formatter.dateFormat = "M월 d일"
        
        let start = formatter.string(from: weekInterval.start)
        let end = formatter.string(from: calendar.date(byAdding: .day, value: 6, to: weekInterval.start) ?? weekInterval.start)
        
        return "\(start) - \(end)"
    }
}

struct MealSlotView: View {
    let date: Date
    let mealType: MealType
    let mealPlan: MealPlan?
    
    var body: some View {
        HStack {
            Text(mealType.icon)
            Text(mealType.rawValue)
                .font(.subheadline)
            
            if let mealPlan = mealPlan, let recipe = mealPlan.recipe {
                Spacer()
                Text(recipe.koreanTitle.isEmpty ? recipe.title : recipe.koreanTitle)
                    .font(.caption)
                    .lineLimit(1)
            } else {
                Spacer()
                Text("+ 추가")
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
        }
        .padding(8)
        .background(mealPlan != nil ? Color.blue.opacity(0.1) : Color(.systemGray6))
        .cornerRadius(8)
    }
}

struct RecipeDrawer: View {
    let recipes: [Recipe]
    @Binding var draggedRecipe: Recipe?
    @State private var isExpanded = false
    
    var body: some View {
        VStack {
            // Handle
            HStack {
                Capsule()
                    .fill(Color.secondary)
                    .frame(width: 40, height: 4)
            }
            .padding(.vertical, 8)
            .onTapGesture {
                withAnimation {
                    isExpanded.toggle()
                }
            }
            
            if isExpanded {
                ScrollView(.horizontal, showsIndicators: false) {
                    HStack(spacing: 12) {
                        ForEach(recipes) { recipe in
                            RecipeThumbnail(recipe: recipe)
                                .onDrag {
                                    draggedRecipe = recipe
                                    return NSItemProvider(object: recipe.id.uuidString as NSString)
                                }
                        }
                    }
                    .padding(.horizontal)
                }
                .frame(height: 100)
            }
        }
        .background(Color(.systemBackground))
        .cornerRadius(20, corners: [.topLeft, .topRight])
        .shadow(radius: 5)
    }
}

struct RecipeThumbnail: View {
    let recipe: Recipe
    
    var body: some View {
        VStack {
            if let imageData = recipe.imageData,
               let uiImage = UIImage(data: imageData) {
                Image(uiImage: uiImage)
                    .resizable()
                    .aspectRatio(contentMode: .fill)
                    .frame(width: 80, height: 60)
                    .cornerRadius(8)
            } else {
                RoundedRectangle(cornerRadius: 8)
                    .fill(Color(.systemGray5))
                    .frame(width: 80, height: 60)
            }
            
            Text(recipe.koreanTitle.isEmpty ? recipe.title : recipe.koreanTitle)
                .font(.caption2)
                .lineLimit(1)
        }
    }
}

// MARK: - Recipe Detail View (Placeholder)
struct RecipeDetailView: View {
    let recipe: Recipe
    
    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 20) {
                Text(recipe.title)
                    .font(.largeTitle)
                    .bold()
                
                if !recipe.koreanTitle.isEmpty {
                    Text(recipe.koreanTitle)
                        .font(.title2)
                        .foregroundColor(.secondary)
                }
                
                // Additional detail implementation
            }
            .padding()
        }
    }
}

// MARK: - Recipe Schema for Import
struct RecipeSchema: Codable {
    let name: String
    let description: String?
    let prepTime: String?
    let cookTime: String?
    let recipeYield: String?
    let recipeIngredient: [String]?
    let recipeInstructions: [InstructionSchema]?
    
    struct InstructionSchema: Codable {
        let text: String
    }
    
    func toRecipe() -> Recipe {
        let recipe = Recipe(title: name, koreanTitle: "")
        recipe.description_text = description ?? ""
        
        // Parse times
        if let prepTime = prepTime {
            recipe.prepTime = parseISO8601Duration(prepTime)
        }
        if let cookTime = cookTime {
            recipe.cookTime = parseISO8601Duration(cookTime)
        }
        
        // Parse servings
        if let yield = recipeYield {
            recipe.servings = Int(yield.filter { $0.isNumber }) ?? 4
        }
        
        // Parse ingredients
        if let ingredients = recipeIngredient {
            for (index, ingredientText) in ingredients.enumerated() {
                let ingredient = Ingredient(name: ingredientText, amount: 1, unit: .piece)
                recipe.ingredients.append(ingredient)
            }
        }
        
        // Parse instructions
        if let instructions = recipeInstructions {
            for (index, instruction) in instructions.enumerated() {
                let step = Instruction(stepNumber: index + 1, text: instruction.text)
                recipe.instructions.append(step)
            }
        }
        
        return recipe
    }
    
    func parseISO8601Duration(_ duration: String) -> Int {
        // PT15M -> 15 minutes
        let pattern = #"PT(?:(\d+)H)?(?:(\d+)M)?"#
        guard let regex = try? NSRegularExpression(pattern: pattern) else { return 0 }
        
        let matches = regex.matches(in: duration, range: NSRange(duration.startIndex..., in: duration))
        guard let match = matches.first else { return 0 }
        
        var totalMinutes = 0
        
        if let hoursRange = Range(match.range(at: 1), in: duration) {
            totalMinutes += (Int(duration[hoursRange]) ?? 0) * 60
        }
        
        if let minutesRange = Range(match.range(at: 2), in: duration) {
            totalMinutes += Int(duration[minutesRange]) ?? 0
        }
        
        return totalMinutes
    }
}

// MARK: - Extensions
extension View {
    func cornerRadius(_ radius: CGFloat, corners: UIRectCorner) -> some View {
        clipShape(RoundedCorner(radius: radius, corners: corners))
    }
}

struct RoundedCorner: Shape {
    var radius: CGFloat = .infinity
    var corners: UIRectCorner = .allCorners
    
    func path(in rect: CGRect) -> Path {
        let path = UIBezierPath(
            roundedRect: rect,
            byRoundingCorners: corners,
            cornerRadii: CGSize(width: radius, height: radius)
        )
        return Path(path.cgPath)
    }
}
```