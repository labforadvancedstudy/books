# RecipeBox: 소셜 기능 완벽 구현

## 사용자 프로필 & 팔로우 시스템

```swift
import SwiftUI
import SwiftData
import PhotosUI
import Combine

// MARK: - Chef Profile Model
@Model
final class ChefProfile {
    var id: UUID = UUID()
    var username: String = ""
    var displayName: String = ""
    var bio: String = ""
    var profileImageData: Data?
    var coverImageData: Data?
    var isVerified: Bool = false
    var verificationBadge: VerificationBadge?
    var joinedDate: Date = Date()
    var lastActiveDate: Date = Date()
    
    // Statistics
    var followersCount: Int = 0
    var followingCount: Int = 0
    var recipesCount: Int = 0
    var totalLikes: Int = 0
    var totalViews: Int = 0
    var averageRating: Double = 0.0
    
    // Social Links
    var instagramHandle: String = ""
    var youtubeChannel: String = ""
    var websiteURL: String = ""
    var kakaoTalkID: String = ""
    
    // Preferences
    var preferredCuisines: [CuisineType] = []
    var dietaryRestrictions: [DietaryRestriction] = []
    var cookingLevel: CookingLevel = .intermediate
    var notificationSettings: NotificationSettings = NotificationSettings()
    
    // Relationships
    @Relationship(deleteRule: .cascade)
    var recipes: [Recipe] = []
    
    @Relationship(deleteRule: .cascade)
    var collections: [RecipeCollection] = []
    
    @Relationship(deleteRule: .cascade)
    var reviews: [Review] = []
    
    @Relationship
    var followers: [ChefProfile] = []
    
    @Relationship
    var following: [ChefProfile] = []
    
    @Relationship
    var blockedUsers: [ChefProfile] = []
    
    @Relationship(deleteRule: .cascade)
    var achievements: [Achievement] = []
    
    @Relationship(deleteRule: .cascade)
    var cookingStreams: [CookingStream] = []
    
    @Relationship
    var participatedChallenges: [RecipeChallenge] = []
    
    init(username: String, displayName: String) {
        self.username = username
        self.displayName = displayName
    }
    
    func isFollowing(_ chef: ChefProfile) -> Bool {
        following.contains { $0.id == chef.id }
    }
    
    func isFollowedBy(_ chef: ChefProfile) -> Bool {
        followers.contains { $0.id == chef.id }
    }
}

// MARK: - Verification System
enum VerificationBadge: String, Codable {
    case professionalChef = "전문 셰프"
    case celebrityChef = "유명 셰프"
    case influencer = "인플루언서"
    case educator = "요리 교육자"
    case restaurant = "레스토랑"
    case brand = "브랜드"
    
    var icon: String {
        switch self {
        case .professionalChef: return "👨‍🍳"
        case .celebrityChef: return "⭐️"
        case .influencer: return "📱"
        case .educator: return "👩‍🏫"
        case .restaurant: return "🍽"
        case .brand: return "🏢"
        }
    }
    
    var color: Color {
        switch self {
        case .professionalChef: return .blue
        case .celebrityChef: return .yellow
        case .influencer: return .purple
        case .educator: return .green
        case .restaurant: return .orange
        case .brand: return .red
        }
    }
}

enum CookingLevel: String, CaseIterable, Codable {
    case beginner = "초보"
    case intermediate = "중급"
    case advanced = "고급"
    case professional = "전문가"
}

enum DietaryRestriction: String, CaseIterable, Codable {
    case vegetarian = "채식주의"
    case vegan = "비건"
    case glutenFree = "글루텐 프리"
    case dairyFree = "유제품 프리"
    case nutFree = "견과류 프리"
    case halal = "할랄"
    case kosher = "코셔"
    case lowCarb = "저탄수화물"
    case keto = "키토"
    case paleo = "팔레오"
}

// MARK: - Review & Rating System
@Model
final class Review {
    var id: UUID = UUID()
    var rating: Int = 5
    var title: String = ""
    var comment: String = ""
    var createdAt: Date = Date()
    var updatedAt: Date = Date()
    var helpfulCount: Int = 0
    var isVerifiedPurchase: Bool = false
    
    @Relationship
    var author: ChefProfile?
    
    @Relationship
    var recipe: Recipe?
    
    @Relationship(deleteRule: .cascade)
    var photos: [ReviewPhoto] = []
    
    @Relationship(deleteRule: .cascade)
    var replies: [ReviewReply] = []
    
    init(rating: Int, title: String, comment: String) {
        self.rating = rating
        self.title = title
        self.comment = comment
    }
}

@Model
final class ReviewPhoto {
    var id: UUID = UUID()
    var imageData: Data?
    var caption: String = ""
    var uploadedAt: Date = Date()
    
    init(imageData: Data?, caption: String = "") {
        self.imageData = imageData
        self.caption = caption
    }
}

@Model
final class ReviewReply {
    var id: UUID = UUID()
    var text: String = ""
    var createdAt: Date = Date()
    var isFromRecipeOwner: Bool = false
    
    @Relationship
    var author: ChefProfile?
    
    init(text: String) {
        self.text = text
    }
}

// MARK: - Social Features Manager
@Observable
final class SocialManager {
    private let modelContext: ModelContext
    private let networkManager = NetworkManager.shared
    
    init(modelContext: ModelContext) {
        self.modelContext = modelContext
    }
    
    // MARK: - Follow System
    func follow(_ chef: ChefProfile, by follower: ChefProfile) async throws {
        guard !follower.isFollowing(chef) else { return }
        
        follower.following.append(chef)
        chef.followers.append(follower)
        chef.followersCount += 1
        follower.followingCount += 1
        
        try modelContext.save()
        
        // Send notification
        await sendFollowNotification(to: chef, from: follower)
        
        // Analytics
        Analytics.track(.userFollowed, properties: [
            "followed_user_id": chef.id.uuidString,
            "follower_id": follower.id.uuidString
        ])
    }
    
    func unfollow(_ chef: ChefProfile, by follower: ChefProfile) async throws {
        guard follower.isFollowing(chef) else { return }
        
        follower.following.removeAll { $0.id == chef.id }
        chef.followers.removeAll { $0.id == follower.id }
        chef.followersCount = max(0, chef.followersCount - 1)
        follower.followingCount = max(0, follower.followingCount - 1)
        
        try modelContext.save()
    }
    
    func blockUser(_ userToBlock: ChefProfile, by blocker: ChefProfile) async throws {
        // Unfollow if following
        if blocker.isFollowing(userToBlock) {
            try await unfollow(userToBlock, by: blocker)
        }
        if userToBlock.isFollowing(blocker) {
            try await unfollow(blocker, by: userToBlock)
        }
        
        blocker.blockedUsers.append(userToBlock)
        try modelContext.save()
    }
    
    // MARK: - Review System
    func submitReview(
        for recipe: Recipe,
        by author: ChefProfile,
        rating: Int,
        title: String,
        comment: String,
        photos: [UIImage]
    ) async throws {
        let review = Review(rating: rating, title: title, comment: comment)
        review.author = author
        review.recipe = recipe
        
        // Process photos
        for photo in photos {
            if let imageData = photo.jpegData(compressionQuality: 0.8) {
                let reviewPhoto = ReviewPhoto(imageData: imageData)
                review.photos.append(reviewPhoto)
            }
        }
        
        recipe.reviews.append(review)
        
        // Update recipe average rating
        updateRecipeRating(recipe)
        
        modelContext.insert(review)
        try modelContext.save()
        
        // Notify recipe owner
        if let recipeOwner = recipe.author, recipeOwner.id != author.id {
            await sendReviewNotification(to: recipeOwner, review: review, recipe: recipe)
        }
        
        // Check for achievements
        await checkReviewAchievements(for: author)
    }
    
    func markReviewHelpful(_ review: Review, by user: ChefProfile) async throws {
        review.helpfulCount += 1
        try modelContext.save()
    }
    
    func replyToReview(_ review: Review, text: String, by author: ChefProfile) async throws {
        let reply = ReviewReply(text: text)
        reply.author = author
        reply.isFromRecipeOwner = review.recipe?.author?.id == author.id
        
        review.replies.append(reply)
        try modelContext.save()
        
        // Notify review author
        if let reviewAuthor = review.author, reviewAuthor.id != author.id {
            await sendReplyNotification(to: reviewAuthor, reply: reply, review: review)
        }
    }
    
    private func updateRecipeRating(_ recipe: Recipe) {
        let totalRating = recipe.reviews.reduce(0) { $0 + $1.rating }
        recipe.averageRating = Double(totalRating) / Double(recipe.reviews.count)
    }
    
    // MARK: - Achievements
    func checkReviewAchievements(for user: ChefProfile) async {
        let reviewCount = user.reviews.count
        
        // First Review Achievement
        if reviewCount == 1 {
            let achievement = Achievement(
                type: .firstReview,
                name: "첫 리뷰",
                description: "첫 번째 리뷰를 작성했습니다",
                iconName: "star.bubble"
            )
            user.achievements.append(achievement)
        }
        
        // Review Milestones
        let milestones = [10, 50, 100, 500, 1000]
        for milestone in milestones {
            if reviewCount == milestone {
                let achievement = Achievement(
                    type: .reviewMilestone,
                    name: "\(milestone)개 리뷰",
                    description: "\(milestone)개의 리뷰를 작성했습니다",
                    iconName: "text.bubble.fill"
                )
                user.achievements.append(achievement)
            }
        }
    }
    
    // MARK: - Notifications
    private func sendFollowNotification(to chef: ChefProfile, from follower: ChefProfile) async {
        guard chef.notificationSettings.followNotifications else { return }
        
        let notification = NotificationRequest(
            title: "새로운 팔로워",
            body: "\(follower.displayName)님이 팔로우하기 시작했습니다",
            userInfo: [
                "type": "follow",
                "follower_id": follower.id.uuidString
            ]
        )
        
        await NotificationManager.shared.schedule(notification)
    }
    
    private func sendReviewNotification(to chef: ChefProfile, review: Review, recipe: Recipe) async {
        guard chef.notificationSettings.reviewNotifications else { return }
        
        let notification = NotificationRequest(
            title: "새로운 리뷰",
            body: "\(review.author?.displayName ?? "누군가")님이 '\(recipe.title)'에 \(review.rating)점 리뷰를 남겼습니다",
            userInfo: [
                "type": "review",
                "review_id": review.id.uuidString,
                "recipe_id": recipe.id.uuidString
            ]
        )
        
        await NotificationManager.shared.schedule(notification)
    }
    
    private func sendReplyNotification(to user: ChefProfile, reply: ReviewReply, review: Review) async {
        guard user.notificationSettings.replyNotifications else { return }
        
        let notification = NotificationRequest(
            title: "리뷰 답글",
            body: "\(reply.author?.displayName ?? "누군가")님이 답글을 남겼습니다",
            userInfo: [
                "type": "reply",
                "reply_id": reply.id.uuidString,
                "review_id": review.id.uuidString
            ]
        )
        
        await NotificationManager.shared.schedule(notification)
    }
}

// MARK: - Profile View
struct ChefProfileView: View {
    let chef: ChefProfile
    @Environment(\.modelContext) private var modelContext
    @StateObject private var socialManager: SocialManagerWrapper
    @State private var selectedTab = 0
    @State private var isFollowing = false
    @State private var showingSettings = false
    @State private var showingShare = false
    
    init(chef: ChefProfile) {
        self.chef = chef
        self._socialManager = StateObject(wrappedValue: SocialManagerWrapper())
    }
    
    var body: some View {
        ScrollView {
            VStack(spacing: 0) {
                // Cover Photo
                ZStack(alignment: .bottom) {
                    if let coverData = chef.coverImageData,
                       let uiImage = UIImage(data: coverData) {
                        Image(uiImage: uiImage)
                            .resizable()
                            .aspectRatio(contentMode: .fill)
                            .frame(height: 200)
                            .clipped()
                    } else {
                        LinearGradient(
                            colors: [.blue, .purple],
                            startPoint: .topLeading,
                            endPoint: .bottomTrailing
                        )
                        .frame(height: 200)
                    }
                    
                    // Profile Image
                    HStack {
                        ProfileImageView(imageData: chef.profileImageData, size: 100)
                            .overlay(
                                Circle()
                                    .stroke(Color.white, lineWidth: 4)
                            )
                            .offset(y: 50)
                        
                        Spacer()
                    }
                    .padding(.horizontal)
                }
                
                // Profile Info
                VStack(alignment: .leading, spacing: 16) {
                    HStack {
                        VStack(alignment: .leading, spacing: 4) {
                            HStack {
                                Text(chef.displayName)
                                    .font(.title)
                                    .bold()
                                
                                if chef.isVerified {
                                    Image(systemName: "checkmark.seal.fill")
                                        .foregroundColor(.blue)
                                }
                                
                                if let badge = chef.verificationBadge {
                                    Text(badge.icon)
                                        .font(.title3)
                                }
                            }
                            
                            Text("@\(chef.username)")
                                .font(.subheadline)
                                .foregroundColor(.secondary)
                        }
                        
                        Spacer()
                        
                        // Action Buttons
                        HStack(spacing: 12) {
                            Button {
                                Task {
                                    if isFollowing {
                                        try await socialManager.manager.unfollow(chef, by: getCurrentUser())
                                    } else {
                                        try await socialManager.manager.follow(chef, by: getCurrentUser())
                                    }
                                    isFollowing.toggle()
                                }
                            } label: {
                                Text(isFollowing ? "팔로잉" : "팔로우")
                                    .font(.subheadline)
                                    .bold()
                                    .padding(.horizontal, 20)
                                    .padding(.vertical, 8)
                                    .background(isFollowing ? Color(.systemGray5) : Color.blue)
                                    .foregroundColor(isFollowing ? .primary : .white)
                                    .cornerRadius(20)
                            }
                            
                            Button {
                                // Message
                            } label: {
                                Image(systemName: "paperplane")
                                    .padding(8)
                                    .background(Color(.systemGray5))
                                    .cornerRadius(20)
                            }
                            
                            Menu {
                                Button {
                                    showingShare = true
                                } label: {
                                    Label("프로필 공유", systemImage: "square.and.arrow.up")
                                }
                                
                                Button {
                                    // Report
                                } label: {
                                    Label("신고하기", systemImage: "exclamationmark.triangle")
                                }
                                
                                Button(role: .destructive) {
                                    Task {
                                        try await socialManager.manager.blockUser(chef, by: getCurrentUser())
                                    }
                                } label: {
                                    Label("차단하기", systemImage: "hand.raised")
                                }
                            } label: {
                                Image(systemName: "ellipsis")
                                    .padding(8)
                                    .background(Color(.systemGray5))
                                    .cornerRadius(20)
                            }
                        }
                    }
                    .padding(.top, 60)
                    
                    // Bio
                    if !chef.bio.isEmpty {
                        Text(chef.bio)
                            .font(.body)
                    }
                    
                    // Stats
                    HStack(spacing: 30) {
                        StatView(value: chef.recipesCount, label: "레시피")
                        StatView(value: chef.followersCount, label: "팔로워")
                        StatView(value: chef.followingCount, label: "팔로잉")
                        
                        if chef.averageRating > 0 {
                            VStack {
                                HStack(spacing: 2) {
                                    Image(systemName: "star.fill")
                                        .foregroundColor(.yellow)
                                        .font(.caption)
                                    Text(String(format: "%.1f", chef.averageRating))
                                        .font(.headline)
                                }
                                Text("평점")
                                    .font(.caption)
                                    .foregroundColor(.secondary)
                            }
                        }
                    }
                    
                    // Social Links
                    if !chef.instagramHandle.isEmpty || !chef.youtubeChannel.isEmpty {
                        HStack(spacing: 16) {
                            if !chef.instagramHandle.isEmpty {
                                Link(destination: URL(string: "https://instagram.com/\(chef.instagramHandle)")!) {
                                    Image("instagram")
                                        .resizable()
                                        .frame(width: 24, height: 24)
                                }
                            }
                            
                            if !chef.youtubeChannel.isEmpty {
                                Link(destination: URL(string: chef.youtubeChannel)!) {
                                    Image("youtube")
                                        .resizable()
                                        .frame(width: 24, height: 24)
                                }
                            }
                            
                            if !chef.websiteURL.isEmpty {
                                Link(destination: URL(string: chef.websiteURL)!) {
                                    Image(systemName: "globe")
                                        .font(.title3)
                                }
                            }
                        }
                    }
                    
                    // Achievements
                    if !chef.achievements.isEmpty {
                        ScrollView(.horizontal, showsIndicators: false) {
                            HStack(spacing: 12) {
                                ForEach(chef.achievements) { achievement in
                                    AchievementBadge(achievement: achievement)
                                }
                            }
                        }
                    }
                }
                .padding()
                
                // Tab Selection
                Picker("Content", selection: $selectedTab) {
                    Text("레시피").tag(0)
                    Text("컬렉션").tag(1)
                    Text("리뷰").tag(2)
                    Text("라이브").tag(3)
                }
                .pickerStyle(SegmentedPickerStyle())
                .padding(.horizontal)
                
                // Tab Content
                switch selectedTab {
                case 0:
                    RecipesGrid(recipes: chef.recipes)
                case 1:
                    CollectionsGrid(collections: chef.collections)
                case 2:
                    ReviewsList(reviews: chef.reviews)
                case 3:
                    StreamsList(streams: chef.cookingStreams)
                default:
                    EmptyView()
                }
            }
        }
        .navigationBarTitleDisplayMode(.inline)
        .sheet(isPresented: $showingShare) {
            ShareSheet(chef: chef)
        }
        .onAppear {
            isFollowing = getCurrentUser().isFollowing(chef)
        }
    }
    
    func getCurrentUser() -> ChefProfile {
        // Return current logged in user
        ChefProfile(username: "current", displayName: "Current User")
    }
}

// MARK: - Recipe Collections
struct CollectionDetailView: View {
    let collection: RecipeCollection
    @State private var showingAddRecipe = false
    
    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 20) {
                // Header
                if let imageData = collection.imageData,
                   let uiImage = UIImage(data: imageData) {
                    Image(uiImage: uiImage)
                        .resizable()
                        .aspectRatio(contentMode: .fill)
                        .frame(maxHeight: 250)
                        .clipped()
                        .cornerRadius(12)
                }
                
                VStack(alignment: .leading, spacing: 8) {
                    Text(collection.name)
                        .font(.largeTitle)
                        .bold()
                    
                    if !collection.description_text.isEmpty {
                        Text(collection.description_text)
                            .font(.body)
                            .foregroundColor(.secondary)
                    }
                    
                    HStack {
                        if let owner = collection.owner {
                            ProfileChip(chef: owner)
                        }
                        
                        Spacer()
                        
                        Text("\(collection.recipes.count)개 레시피")
                            .font(.subheadline)
                            .foregroundColor(.secondary)
                        
                        if collection.isPublic {
                            Label("공개", systemImage: "globe")
                                .font(.caption)
                                .padding(.horizontal, 8)
                                .padding(.vertical, 4)
                                .background(Color.green.opacity(0.2))
                                .foregroundColor(.green)
                                .cornerRadius(12)
                        } else {
                            Label("비공개", systemImage: "lock")
                                .font(.caption)
                                .padding(.horizontal, 8)
                                .padding(.vertical, 4)
                                .background(Color.orange.opacity(0.2))
                                .foregroundColor(.orange)
                                .cornerRadius(12)
                        }
                    }
                }
                .padding(.horizontal)
                
                // Recipes
                LazyVStack(spacing: 16) {
                    ForEach(collection.recipes) { recipe in
                        NavigationLink(destination: RecipeDetailView(recipe: recipe)) {
                            RecipeCard(recipe: recipe)
                        }
                        .buttonStyle(PlainButtonStyle())
                    }
                }
                .padding(.horizontal)
            }
        }
        .navigationBarTitleDisplayMode(.inline)
        .toolbar {
            if isOwner() {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button {
                        showingAddRecipe = true
                    } label: {
                        Image(systemName: "plus")
                    }
                }
            }
        }
        .sheet(isPresented: $showingAddRecipe) {
            AddRecipeToCollectionView(collection: collection)
        }
    }
    
    func isOwner() -> Bool {
        // Check if current user is collection owner
        true
    }
}

// MARK: - Social Sharing
struct ShareManager {
    static func shareToKakaoTalk(recipe: Recipe) {
        guard let kakaoTalkURL = URL(string: "kakaolink://send") else { return }
        
        if UIApplication.shared.canOpenURL(kakaoTalkURL) {
            let template = """
            {
                "object_type": "feed",
                "content": {
                    "title": "\(recipe.title)",
                    "description": "\(recipe.description_text)",
                    "image_url": "https://recipebox.app/recipe/\(recipe.id)",
                    "link": {
                        "mobile_web_url": "https://recipebox.app/recipe/\(recipe.id)",
                        "web_url": "https://recipebox.app/recipe/\(recipe.id)"
                    }
                },
                "buttons": [
                    {
                        "title": "레시피 보기",
                        "link": {
                            "mobile_web_url": "https://recipebox.app/recipe/\(recipe.id)",
                            "web_url": "https://recipebox.app/recipe/\(recipe.id)"
                        }
                    }
                ]
            }
            """
            
            // Send to KakaoTalk
            // Implementation requires KakaoSDK
        }
    }
    
    static func shareToInstagram(image: UIImage, recipe: Recipe) {
        guard let instagramURL = URL(string: "instagram://library") else { return }
        
        if UIApplication.shared.canOpenURL(instagramURL) {
            // Save image to photos
            UIImageWriteToSavedPhotosAlbum(image, nil, nil, nil)
            
            // Copy recipe text to clipboard
            UIPasteboard.general.string = """
            \(recipe.title)
            
            재료:
            \(recipe.ingredients.map { "• \($0.name): \($0.amount) \($0.unit.rawValue)" }.joined(separator: "\n"))
            
            #RecipeBox #\(recipe.cuisine.rawValue) #요리 #레시피
            """
            
            // Open Instagram
            UIApplication.shared.open(instagramURL)
        }
    }
    
    static func generateShareImage(for recipe: Recipe) -> UIImage {
        let renderer = UIGraphicsImageRenderer(size: CGSize(width: 1080, height: 1080))
        
        return renderer.image { context in
            // Background
            UIColor.systemBackground.setFill()
            context.fill(CGRect(x: 0, y: 0, width: 1080, height: 1080))
            
            // Recipe Image
            if let imageData = recipe.imageData,
               let recipeImage = UIImage(data: imageData) {
                recipeImage.draw(in: CGRect(x: 0, y: 0, width: 1080, height: 600))
            }
            
            // Title
            let titleAttributes: [NSAttributedString.Key: Any] = [
                .font: UIFont.systemFont(ofSize: 48, weight: .bold),
                .foregroundColor: UIColor.label
            ]
            recipe.title.draw(at: CGPoint(x: 50, y: 650), withAttributes: titleAttributes)
            
            // Info
            let infoAttributes: [NSAttributedString.Key: Any] = [
                .font: UIFont.systemFont(ofSize: 32),
                .foregroundColor: UIColor.secondaryLabel
            ]
            let info = "\(recipe.cuisine.icon) \(recipe.cuisine.rawValue) | \(recipe.totalTime)분 | \(recipe.difficulty.rawValue)"
            info.draw(at: CGPoint(x: 50, y: 720), withAttributes: infoAttributes)
            
            // Logo
            let logoText = "RecipeBox"
            let logoAttributes: [NSAttributedString.Key: Any] = [
                .font: UIFont.systemFont(ofSize: 24, weight: .light),
                .foregroundColor: UIColor.tertiaryLabel
            ]
            logoText.draw(at: CGPoint(x: 50, y: 1000), withAttributes: logoAttributes)
        }
    }
}

// MARK: - Live Cooking Streams
@Model
final class CookingStream {
    var id: UUID = UUID()
    var title: String = ""
    var description_text: String = ""
    var thumbnailData: Data?
    var streamURL: String = ""
    var scheduledDate: Date = Date()
    var duration: Int = 60 // minutes
    var isLive: Bool = false
    var viewerCount: Int = 0
    var maxViewers: Int = 0
    var likes: Int = 0
    
    @Relationship
    var host: ChefProfile?
    
    @Relationship
    var recipe: Recipe?
    
    @Relationship(deleteRule: .cascade)
    var comments: [StreamComment] = []
    
    @Relationship
    var viewers: [ChefProfile] = []
    
    init(title: String, host: ChefProfile) {
        self.title = title
        self.host = host
    }
}

@Model
final class StreamComment {
    var id: UUID = UUID()
    var text: String = ""
    var timestamp: Date = Date()
    var isHighlighted: Bool = false
    var isPinned: Bool = false
    
    @Relationship
    var author: ChefProfile?
    
    init(text: String) {
        self.text = text
    }
}

// MARK: - Live Stream View
struct LiveStreamView: View {
    let stream: CookingStream
    @State private var comment = ""
    @State private var showingChat = true
    @State private var isFollowing = false
    @Environment(\.modelContext) private var modelContext
    
    var body: some View {
        GeometryReader { geometry in
            ZStack {
                // Video Player Background
                Color.black
                    .ignoresSafeArea()
                
                // Stream Content
                VStack(spacing: 0) {
                    // Top Bar
                    HStack {
                        // Host Info
                        HStack(spacing: 12) {
                            ProfileImageView(imageData: stream.host?.profileImageData, size: 40)
                            
                            VStack(alignment: .leading, spacing: 2) {
                                Text(stream.host?.displayName ?? "")
                                    .font(.headline)
                                    .foregroundColor(.white)
                                
                                HStack(spacing: 4) {
                                    Circle()
                                        .fill(Color.red)
                                        .frame(width: 8, height: 8)
                                    
                                    Text("LIVE")
                                        .font(.caption)
                                        .bold()
                                    
                                    Text("• \(stream.viewerCount)명 시청중")
                                        .font(.caption)
                                }
                                .foregroundColor(.white.opacity(0.9))
                            }
                        }
                        
                        Spacer()
                        
                        // Action Buttons
                        HStack(spacing: 16) {
                            Button {
                                isFollowing.toggle()
                            } label: {
                                Text(isFollowing ? "팔로잉" : "팔로우")
                                    .font(.caption)
                                    .bold()
                                    .padding(.horizontal, 12)
                                    .padding(.vertical, 6)
                                    .background(isFollowing ? Color.gray : Color.red)
                                    .foregroundColor(.white)
                                    .cornerRadius(15)
                            }
                            
                            Button {
                                // Share
                            } label: {
                                Image(systemName: "square.and.arrow.up")
                                    .foregroundColor(.white)
                            }
                            
                            Button {
                                // Close
                            } label: {
                                Image(systemName: "xmark")
                                    .foregroundColor(.white)
                            }
                        }
                    }
                    .padding()
                    .background(
                        LinearGradient(
                            colors: [Color.black.opacity(0.7), Color.clear],
                            startPoint: .top,
                            endPoint: .bottom
                        )
                    )
                    
                    Spacer()
                    
                    // Bottom Controls
                    VStack(spacing: 0) {
                        // Recipe Info
                        if let recipe = stream.recipe {
                            HStack {
                                VStack(alignment: .leading, spacing: 4) {
                                    Text("지금 만들고 있는 레시피")
                                        .font(.caption)
                                        .foregroundColor(.white.opacity(0.7))
                                    
                                    Text(recipe.title)
                                        .font(.headline)
                                        .foregroundColor(.white)
                                }
                                
                                Spacer()
                                
                                Button {
                                    // View Recipe
                                } label: {
                                    Text("레시피 보기")
                                        .font(.caption)
                                        .bold()
                                        .padding(.horizontal, 16)
                                        .padding(.vertical, 8)
                                        .background(Color.white)
                                        .foregroundColor(.black)
                                        .cornerRadius(20)
                                }
                            }
                            .padding()
                            .background(Color.black.opacity(0.5))
                            .cornerRadius(12)
                            .padding(.horizontal)
                        }
                        
                        // Chat Toggle
                        if showingChat {
                            StreamChatView(stream: stream, comment: $comment)
                                .frame(height: geometry.size.height * 0.3)
                                .transition(.move(edge: .bottom))
                        }
                        
                        // Input Bar
                        HStack(spacing: 12) {
                            TextField("댓글 달기...", text: $comment)
                                .textFieldStyle(RoundedBorderTextFieldStyle())
                            
                            Button {
                                sendComment()
                            } label: {
                                Image(systemName: "paperplane.fill")
                                    .foregroundColor(.white)
                            }
                            .disabled(comment.isEmpty)
                            
                            Button {
                                // Send Heart
                            } label: {
                                Image(systemName: "heart.fill")
                                    .foregroundColor(.red)
                            }
                            
                            Button {
                                withAnimation {
                                    showingChat.toggle()
                                }
                            } label: {
                                Image(systemName: showingChat ? "message.fill" : "message")
                                    .foregroundColor(.white)
                            }
                        }
                        .padding()
                        .background(Color.black.opacity(0.8))
                    }
                }
            }
        }
        .statusBarHidden()
    }
    
    func sendComment() {
        let streamComment = StreamComment(text: comment)
        streamComment.author = getCurrentUser()
        stream.comments.append(streamComment)
        
        do {
            try modelContext.save()
            comment = ""
        } catch {
            print("Failed to send comment: \(error)")
        }
    }
    
    func getCurrentUser() -> ChefProfile {
        // Return current user
        ChefProfile(username: "user", displayName: "User")
    }
}

// MARK: - Recipe Challenges
@Model
final class RecipeChallenge {
    var id: UUID = UUID()
    var title: String = ""
    var description_text: String = ""
    var hashtag: String = ""
    var startDate: Date = Date()
    var endDate: Date = Date()
    var prizeDescription: String = ""
    var rules: [String] = []
    var isActive: Bool = true
    var participantCount: Int = 0
    
    @Relationship
    var sponsor: ChefProfile?
    
    @Relationship
    var judges: [ChefProfile] = []
    
    @Relationship(deleteRule: .cascade)
    var submissions: [ChallengeSubmission] = []
    
    @Relationship
    var winners: [ChallengeSubmission] = []
    
    init(title: String, hashtag: String) {
        self.title = title
        self.hashtag = hashtag
    }
}

@Model
final class ChallengeSubmission {
    var id: UUID = UUID()
    var submittedAt: Date = Date()
    var votes: Int = 0
    var judgeScore: Double = 0.0
    var rank: Int?
    
    @Relationship
    var participant: ChefProfile?
    
    @Relationship
    var recipe: Recipe?
    
    @Relationship(deleteRule: .cascade)
    var photos: [Data] = []
    
    init(participant: ChefProfile, recipe: Recipe) {
        self.participant = participant
        self.recipe = recipe
    }
}

// MARK: - Challenge View
struct ChallengeDetailView: View {
    let challenge: RecipeChallenge
    @State private var showingSubmit = false
    @State private var sortBy: ChallengeSortOption = .popular
    
    enum ChallengeSortOption: String, CaseIterable {
        case popular = "인기순"
        case recent = "최신순"
        case judgeScore = "심사점수순"
    }
    
    var sortedSubmissions: [ChallengeSubmission] {
        switch sortBy {
        case .popular:
            return challenge.submissions.sorted { $0.votes > $1.votes }
        case .recent:
            return challenge.submissions.sorted { $0.submittedAt > $1.submittedAt }
        case .judgeScore:
            return challenge.submissions.sorted { $0.judgeScore > $1.judgeScore }
        }
    }
    
    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 20) {
                // Header
                VStack(alignment: .leading, spacing: 12) {
                    Text(challenge.title)
                        .font(.largeTitle)
                        .bold()
                    
                    Text(challenge.hashtag)
                        .font(.headline)
                        .foregroundColor(.blue)
                    
                    Text(challenge.description_text)
                        .font(.body)
                    
                    // Challenge Info
                    HStack(spacing: 20) {
                        VStack(alignment: .leading) {
                            Text("기간")
                                .font(.caption)
                                .foregroundColor(.secondary)
                            Text("\(challenge.startDate, format: .dateTime.month().day()) - \(challenge.endDate, format: .dateTime.month().day())")
                                .font(.subheadline)
                        }
                        
                        VStack(alignment: .leading) {
                            Text("참가자")
                                .font(.caption)
                                .foregroundColor(.secondary)
                            Text("\(challenge.participantCount)명")
                                .font(.subheadline)
                        }
                        
                        if !challenge.prizeDescription.isEmpty {
                            VStack(alignment: .leading) {
                                Text("상품")
                                    .font(.caption)
                                    .foregroundColor(.secondary)
                                Text(challenge.prizeDescription)
                                    .font(.subheadline)
                            }
                        }
                    }
                    
                    // Rules
                    if !challenge.rules.isEmpty {
                        VStack(alignment: .leading, spacing: 8) {
                            Text("참가 규칙")
                                .font(.headline)
                            
                            ForEach(challenge.rules, id: \.self) { rule in
                                HStack(alignment: .top) {
                                    Text("•")
                                    Text(rule)
                                        .font(.subheadline)
                                }
                            }
                        }
                    }
                    
                    // Judges
                    if !challenge.judges.isEmpty {
                        VStack(alignment: .leading, spacing: 8) {
                            Text("심사위원")
                                .font(.headline)
                            
                            ScrollView(.horizontal, showsIndicators: false) {
                                HStack(spacing: 16) {
                                    ForEach(challenge.judges) { judge in
                                        VStack {
                                            ProfileImageView(imageData: judge.profileImageData, size: 60)
                                            Text(judge.displayName)
                                                .font(.caption)
                                                .lineLimit(1)
                                        }
                                    }
                                }
                            }
                        }
                    }
                    
                    // Action Button
                    if challenge.isActive && Date() < challenge.endDate {
                        Button {
                            showingSubmit = true
                        } label: {
                            Text("챌린지 참가하기")
                                .font(.headline)
                                .foregroundColor(.white)
                                .frame(maxWidth: .infinity)
                                .padding()
                                .background(Color.blue)
                                .cornerRadius(12)
                        }
                    }
                }
                .padding()
                
                // Sort Options
                Picker("정렬", selection: $sortBy) {
                    ForEach(ChallengeSortOption.allCases, id: \.self) { option in
                        Text(option.rawValue).tag(option)
                    }
                }
                .pickerStyle(SegmentedPickerStyle())
                .padding(.horizontal)
                
                // Submissions Grid
                LazyVGrid(columns: [GridItem(.adaptive(minimum: 160))], spacing: 16) {
                    ForEach(sortedSubmissions) { submission in
                        ChallengeSubmissionCard(submission: submission)
                    }
                }
                .padding()
            }
        }
        .navigationBarTitleDisplayMode(.inline)
        .sheet(isPresented: $showingSubmit) {
            SubmitToChallengeView(challenge: challenge)
        }
    }
}

// MARK: - Supporting Views & Models
@Model
final class NotificationSettings {
    var followNotifications: Bool = true
    var reviewNotifications: Bool = true
    var replyNotifications: Bool = true
    var likeNotifications: Bool = true
    var challengeNotifications: Bool = true
    var streamNotifications: Bool = true
    var weeklyDigest: Bool = true
    
    init() {}
}

@Model
final class Achievement {
    var id: UUID = UUID()
    var type: AchievementType
    var name: String = ""
    var description_text: String = ""
    var iconName: String = ""
    var unlockedAt: Date = Date()
    var progress: Int = 0
    var maxProgress: Int = 1
    
    init(type: AchievementType, name: String, description: String, iconName: String) {
        self.type = type
        self.name = name
        self.description_text = description
        self.iconName = iconName
    }
}

enum AchievementType: String, Codable {
    case firstRecipe
    case firstReview
    case firstFollow
    case recipeMilestone
    case reviewMilestone
    case followerMilestone
    case challengeWinner
    case verifiedChef
    case topContributor
}

// MARK: - UI Components
struct ProfileImageView: View {
    let imageData: Data?
    let size: CGFloat
    
    var body: some View {
        if let imageData = imageData,
           let uiImage = UIImage(data: imageData) {
            Image(uiImage: uiImage)
                .resizable()
                .aspectRatio(contentMode: .fill)
                .frame(width: size, height: size)
                .clipShape(Circle())
        } else {
            Circle()
                .fill(Color(.systemGray5))
                .frame(width: size, height: size)
                .overlay(
                    Image(systemName: "person.fill")
                        .foregroundColor(.secondary)
                        .font(.system(size: size * 0.4))
                )
        }
    }
}

struct StatView: View {
    let value: Int
    let label: String
    
    var body: some View {
        VStack(spacing: 4) {
            Text(formatNumber(value))
                .font(.headline)
            Text(label)
                .font(.caption)
                .foregroundColor(.secondary)
        }
    }
    
    func formatNumber(_ number: Int) -> String {
        if number >= 10000 {
            return "\(number / 1000)K"
        } else if number >= 1000 {
            return String(format: "%.1fK", Double(number) / 1000.0)
        }
        return "\(number)"
    }
}

struct ProfileChip: View {
    let chef: ChefProfile
    
    var body: some View {
        HStack(spacing: 8) {
            ProfileImageView(imageData: chef.profileImageData, size: 24)
            
            Text(chef.displayName)
                .font(.subheadline)
            
            if chef.isVerified {
                Image(systemName: "checkmark.seal.fill")
                    .font(.caption)
                    .foregroundColor(.blue)
            }
        }
    }
}

struct AchievementBadge: View {
    let achievement: Achievement
    
    var body: some View {
        VStack(spacing: 4) {
            Image(systemName: achievement.iconName)
                .font(.title2)
                .foregroundColor(.yellow)
            
            Text(achievement.name)
                .font(.caption2)
                .lineLimit(2)
                .multilineTextAlignment(.center)
        }
        .frame(width: 70, height: 70)
        .background(Color(.systemGray6))
        .cornerRadius(12)
    }
}

struct RecipesGrid: View {
    let recipes: [Recipe]
    
    var body: some View {
        LazyVGrid(columns: [GridItem(.adaptive(minimum: 160))], spacing: 16) {
            ForEach(recipes) { recipe in
                NavigationLink(destination: RecipeDetailView(recipe: recipe)) {
                    RecipeGridItem(recipe: recipe)
                }
                .buttonStyle(PlainButtonStyle())
            }
        }
        .padding()
    }
}

struct RecipeGridItem: View {
    let recipe: Recipe
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            if let imageData = recipe.imageData,
               let uiImage = UIImage(data: imageData) {
                Image(uiImage: uiImage)
                    .resizable()
                    .aspectRatio(contentMode: .fill)
                    .frame(height: 160)
                    .clipped()
                    .cornerRadius(12)
            } else {
                RoundedRectangle(cornerRadius: 12)
                    .fill(Color(.systemGray5))
                    .frame(height: 160)
            }
            
            Text(recipe.title)
                .font(.headline)
                .lineLimit(2)
            
            HStack {
                if recipe.averageRating > 0 {
                    HStack(spacing: 2) {
                        Image(systemName: "star.fill")
                            .font(.caption)
                            .foregroundColor(.yellow)
                        Text(String(format: "%.1f", recipe.averageRating))
                            .font(.caption)
                    }
                }
                
                Spacer()
                
                Text("\(recipe.totalTime)분")
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
        }
    }
}

struct CollectionsGrid: View {
    let collections: [RecipeCollection]
    
    var body: some View {
        LazyVStack(spacing: 16) {
            ForEach(collections) { collection in
                NavigationLink(destination: CollectionDetailView(collection: collection)) {
                    CollectionCard(collection: collection)
                }
                .buttonStyle(PlainButtonStyle())
            }
        }
        .padding()
    }
}

struct CollectionCard: View {
    let collection: RecipeCollection
    
    var body: some View {
        HStack(spacing: 16) {
            if let imageData = collection.imageData,
               let uiImage = UIImage(data: imageData) {
                Image(uiImage: uiImage)
                    .resizable()
                    .aspectRatio(contentMode: .fill)
                    .frame(width: 80, height: 80)
                    .cornerRadius(12)
            } else {
                RoundedRectangle(cornerRadius: 12)
                    .fill(Color(.systemGray5))
                    .frame(width: 80, height: 80)
            }
            
            VStack(alignment: .leading, spacing: 4) {
                Text(collection.name)
                    .font(.headline)
                
                Text("\(collection.recipes.count)개 레시피")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
                
                if collection.isPublic {
                    Label("공개", systemImage: "globe")
                        .font(.caption)
                        .foregroundColor(.blue)
                }
            }
            
            Spacer()
        }
        .padding()
        .background(Color(.systemGray6))
        .cornerRadius(16)
    }
}

struct ReviewsList: View {
    let reviews: [Review]
    
    var body: some View {
        LazyVStack(spacing: 16) {
            ForEach(reviews) { review in
                ReviewCard(review: review)
            }
        }
        .padding()
    }
}

struct ReviewCard: View {
    let review: Review
    @State private var showingReplies = false
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            // Header
            HStack {
                if let author = review.author {
                    ProfileChip(chef: author)
                }
                
                Spacer()
                
                HStack(spacing: 2) {
                    ForEach(1...5, id: \.self) { star in
                        Image(systemName: star <= review.rating ? "star.fill" : "star")
                            .font(.caption)
                            .foregroundColor(.yellow)
                    }
                }
            }
            
            // Content
            if !review.title.isEmpty {
                Text(review.title)
                    .font(.headline)
            }
            
            Text(review.comment)
                .font(.body)
            
            // Photos
            if !review.photos.isEmpty {
                ScrollView(.horizontal, showsIndicators: false) {
                    HStack(spacing: 8) {
                        ForEach(review.photos) { photo in
                            if let imageData = photo.imageData,
                               let uiImage = UIImage(data: imageData) {
                                Image(uiImage: uiImage)
                                    .resizable()
                                    .aspectRatio(contentMode: .fill)
                                    .frame(width: 80, height: 80)
                                    .cornerRadius(8)
                            }
                        }
                    }
                }
            }
            
            // Footer
            HStack {
                Text(review.createdAt, format: .dateTime.month().day())
                    .font(.caption)
                    .foregroundColor(.secondary)
                
                if review.isVerifiedPurchase {
                    Label("구매 인증", systemImage: "checkmark.seal")
                        .font(.caption)
                        .foregroundColor(.green)
                }
                
                Spacer()
                
                if review.helpfulCount > 0 {
                    Label("\(review.helpfulCount)", systemImage: "hand.thumbsup")
                        .font(.caption)
                }
                
                if !review.replies.isEmpty {
                    Button {
                        showingReplies.toggle()
                    } label: {
                        Label("\(review.replies.count)", systemImage: "bubble.right")
                            .font(.caption)
                    }
                }
            }
            
            // Replies
            if showingReplies && !review.replies.isEmpty {
                VStack(alignment: .leading, spacing: 8) {
                    ForEach(review.replies) { reply in
                        HStack(alignment: .top, spacing: 8) {
                            if reply.isFromRecipeOwner {
                                Image(systemName: "checkmark.seal.fill")
                                    .font(.caption)
                                    .foregroundColor(.blue)
                            }
                            
                            VStack(alignment: .leading, spacing: 4) {
                                Text(reply.author?.displayName ?? "")
                                    .font(.caption)
                                    .bold()
                                
                                Text(reply.text)
                                    .font(.caption)
                            }
                        }
                        .padding(.leading, 20)
                    }
                }
            }
        }
        .padding()
        .background(Color(.systemGray6))
        .cornerRadius(12)
    }
}

struct StreamsList: View {
    let streams: [CookingStream]
    
    var body: some View {
        LazyVStack(spacing: 16) {
            ForEach(streams) { stream in
                NavigationLink(destination: LiveStreamView(stream: stream)) {
                    StreamCard(stream: stream)
                }
                .buttonStyle(PlainButtonStyle())
            }
        }
        .padding()
    }
}

struct StreamCard: View {
    let stream: CookingStream
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            ZStack(alignment: .topLeading) {
                if let thumbnailData = stream.thumbnailData,
                   let uiImage = UIImage(data: thumbnailData) {
                    Image(uiImage: uiImage)
                        .resizable()
                        .aspectRatio(16/9, contentMode: .fill)
                        .cornerRadius(12)
                } else {
                    RoundedRectangle(cornerRadius: 12)
                        .fill(Color(.systemGray5))
                        .aspectRatio(16/9, contentMode: .fill)
                }
                
                if stream.isLive {
                    HStack(spacing: 4) {
                        Circle()
                            .fill(Color.red)
                            .frame(width: 8, height: 8)
                        
                        Text("LIVE")
                            .font(.caption)
                            .bold()
                    }
                    .padding(.horizontal, 8)
                    .padding(.vertical, 4)
                    .background(Color.red)
                    .foregroundColor(.white)
                    .cornerRadius(8)
                    .padding(8)
                }
            }
            
            VStack(alignment: .leading, spacing: 4) {
                Text(stream.title)
                    .font(.headline)
                    .lineLimit(2)
                
                HStack {
                    if let host = stream.host {
                        ProfileChip(chef: host)
                    }
                    
                    Spacer()
                    
                    if stream.isLive {
                        Text("\(stream.viewerCount)명 시청중")
                            .font(.caption)
                            .foregroundColor(.secondary)
                    } else {
                        Text(stream.scheduledDate, format: .dateTime.month().day().hour().minute())
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }
                }
            }
        }
    }
}

struct StreamChatView: View {
    let stream: CookingStream
    @Binding var comment: String
    
    var body: some View {
        ScrollViewReader { proxy in
            ScrollView {
                LazyVStack(alignment: .leading, spacing: 8) {
                    ForEach(stream.comments) { comment in
                        HStack(alignment: .top, spacing: 8) {
                            if comment.isPinned {
                                Image(systemName: "pin.fill")
                                    .font(.caption)
                                    .foregroundColor(.yellow)
                            }
                            
                            Text(comment.author?.displayName ?? "")
                                .font(.caption)
                                .bold()
                                .foregroundColor(.white)
                            
                            Text(comment.text)
                                .font(.caption)
                                .foregroundColor(.white.opacity(0.9))
                        }
                        .padding(.horizontal, 12)
                        .padding(.vertical, 4)
                        .background(
                            comment.isHighlighted ? 
                            Color.yellow.opacity(0.3) : 
                            Color.black.opacity(0.3)
                        )
                        .cornerRadius(8)
                        .id(comment.id)
                    }
                }
                .padding()
            }
            .onChange(of: stream.comments.count) { _ in
                if let lastComment = stream.comments.last {
                    withAnimation {
                        proxy.scrollTo(lastComment.id, anchor: .bottom)
                    }
                }
            }
        }
    }
}

struct ChallengeSubmissionCard: View {
    let submission: ChallengeSubmission
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            if let firstPhoto = submission.photos.first,
               let uiImage = UIImage(data: firstPhoto) {
                Image(uiImage: uiImage)
                    .resizable()
                    .aspectRatio(contentMode: .fill)
                    .frame(height: 160)
                    .clipped()
                    .cornerRadius(12)
            }
            
            if let participant = submission.participant {
                ProfileChip(chef: participant)
            }
            
            if let recipe = submission.recipe {
                Text(recipe.title)
                    .font(.headline)
                    .lineLimit(2)
            }
            
            HStack {
                Label("\(submission.votes)", systemImage: "heart.fill")
                    .font(.caption)
                    .foregroundColor(.red)
                
                if submission.judgeScore > 0 {
                    Spacer()
                    Text(String(format: "%.1f", submission.judgeScore))
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
            }
            
            if let rank = submission.rank {
                HStack {
                    Spacer()
                    Text("\(rank)위")
                        .font(.caption)
                        .bold()
                        .padding(.horizontal, 8)
                        .padding(.vertical, 4)
                        .background(rankColor(rank))
                        .foregroundColor(.white)
                        .cornerRadius(8)
                }
            }
        }
    }
    
    func rankColor(_ rank: Int) -> Color {
        switch rank {
        case 1: return .yellow
        case 2: return .gray
        case 3: return .orange
        default: return .blue
        }
    }
}

// MARK: - Helper Classes
class SocialManagerWrapper: ObservableObject {
    let manager: SocialManager
    
    init() {
        // Initialize with proper ModelContext
        let container = try! ModelContainer(for: ChefProfile.self, Recipe.self)
        self.manager = SocialManager(modelContext: container.mainContext)
    }
}

struct ShareSheet: View {
    let chef: ChefProfile
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 20) {
                // Profile Preview
                VStack(spacing: 12) {
                    ProfileImageView(imageData: chef.profileImageData, size: 100)
                    
                    Text(chef.displayName)
                        .font(.title2)
                        .bold()
                    
                    Text("@\(chef.username)")
                        .font(.subheadline)
                        .foregroundColor(.secondary)
                }
                .padding()
                
                // Share Options
                VStack(spacing: 16) {
                    ShareButton(title: "카카오톡", icon: "message.fill", color: .yellow) {
                        shareToKakaoTalk()
                    }
                    
                    ShareButton(title: "인스타그램", icon: "camera.fill", color: .purple) {
                        shareToInstagram()
                    }
                    
                    ShareButton(title: "링크 복사", icon: "link", color: .blue) {
                        copyLink()
                    }
                }
                .padding()
                
                Spacer()
            }
            .navigationTitle("프로필 공유")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("닫기") {
                        dismiss()
                    }
                }
            }
        }
    }
    
    func shareToKakaoTalk() {
        // KakaoTalk share implementation
    }
    
    func shareToInstagram() {
        // Instagram share implementation
    }
    
    func copyLink() {
        UIPasteboard.general.string = "https://recipebox.app/chef/\(chef.username)"
    }
}

struct ShareButton: View {
    let title: String
    let icon: String
    let color: Color
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            HStack {
                Image(systemName: icon)
                Text(title)
                Spacer()
                Image(systemName: "chevron.right")
                    .font(.caption)
            }
            .padding()
            .background(color.opacity(0.1))
            .foregroundColor(color)
            .cornerRadius(12)
        }
    }
}

struct AddRecipeToCollectionView: View {
    let collection: RecipeCollection
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        NavigationStack {
            Text("Add Recipe to Collection")
            // Implementation
        }
    }
}

struct SubmitToChallengeView: View {
    let challenge: RecipeChallenge
    @Environment(\.dismiss) private var dismiss
    
    var body: some View {
        NavigationStack {
            Text("Submit to Challenge")
            // Implementation
        }
    }
}

// MARK: - Network & Analytics Helpers
class NetworkManager {
    static let shared = NetworkManager()
    private init() {}
}

struct NotificationRequest {
    let title: String
    let body: String
    let userInfo: [String: String]
}

class NotificationManager {
    static let shared = NotificationManager()
    
    func schedule(_ request: NotificationRequest) async {
        // Schedule notification
    }
}

enum Analytics {
    enum Event {
        case userFollowed
        case recipeViewed
        case reviewSubmitted
    }
    
    static func track(_ event: Event, properties: [String: Any] = [:]) {
        // Track analytics event
    }
}
```