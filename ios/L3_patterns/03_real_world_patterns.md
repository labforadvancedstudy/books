# L3: 실전 패턴 - 프로덕션 레벨 SwiftUI

## 핵심: 이론과 현실 사이의 다리

> "In theory, there's no difference between theory and practice. In practice, there is." - Yogi Berra

L3에서 배운 패턴들을 실제 프로덕션 환경에서 어떻게 적용하는지 살펴봅시다.

## 1. 복잡한 폼 관리 패턴

### Form State Machine 패턴

```swift
// MARK: - Form State를 명시적으로 관리
enum FormState: Equatable {
    case editing
    case validating
    case submitting
    case submitted(result: Result<String, FormError>)
    case error(FormError)
    
    var isLoading: Bool {
        switch self {
        case .validating, .submitting: return true
        default: return false
        }
    }
    
    var canEdit: Bool {
        switch self {
        case .editing, .error: return true
        default: return false
        }
    }
}

enum FormError: LocalizedError, Equatable {
    case invalidEmail
    case passwordTooShort
    case phoneInvalid
    case networkError
    case serverError(String)
    
    var errorDescription: String? {
        switch self {
        case .invalidEmail: return "올바른 이메일을 입력하세요"
        case .passwordTooShort: return "비밀번호는 8자 이상이어야 합니다"
        case .phoneInvalid: return "올바른 전화번호를 입력하세요"
        case .networkError: return "네트워크 연결을 확인하세요"
        case .serverError(let message): return message
        }
    }
}

// MARK: - Observable Form ViewModel
@Observable
class UserRegistrationForm {
    var email = ""
    var password = ""
    var confirmPassword = ""
    var phone = ""
    var agreeToTerms = false
    
    var state = FormState.editing
    var validationErrors: Set<FormError> = []
    
    // Computed validation
    var isEmailValid: Bool {
        email.isValidEmail
    }
    
    var isPasswordValid: Bool {
        password.count >= 8
    }
    
    var doPasswordsMatch: Bool {
        password == confirmPassword && !password.isEmpty
    }
    
    var isPhoneValid: Bool {
        phone.isValidKoreanPhone
    }
    
    var isFormValid: Bool {
        isEmailValid && 
        isPasswordValid && 
        doPasswordsMatch && 
        isPhoneValid && 
        agreeToTerms &&
        validationErrors.isEmpty
    }
    
    // Validation methods
    func validateEmail() {
        if !email.isEmpty && !isEmailValid {
            validationErrors.insert(.invalidEmail)
        } else {
            validationErrors.remove(.invalidEmail)
        }
    }
    
    func validatePassword() {
        if !password.isEmpty && !isPasswordValid {
            validationErrors.insert(.passwordTooShort)
        } else {
            validationErrors.remove(.passwordTooShort)
        }
    }
    
    func validatePhone() {
        if !phone.isEmpty && !isPhoneValid {
            validationErrors.insert(.phoneInvalid)
        } else {
            validationErrors.remove(.phoneInvalid)
        }
    }
    
    // 실시간 validation
    func validateAll() {
        state = .validating
        
        Task { @MainActor in
            // 각 필드 검증
            validateEmail()
            validatePassword()
            validatePhone()
            
            // 추가 검증 (중복 이메일 체크 등)
            if isEmailValid {
                do {
                    let isAvailable = try await UserAPI.checkEmailAvailability(email)
                    if !isAvailable {
                        validationErrors.insert(.invalidEmail)
                    }
                } catch {
                    validationErrors.insert(.networkError)
                }
            }
            
            state = .editing
        }
    }
    
    // 실제 제출
    func submit() async {
        guard isFormValid else { return }
        
        state = .submitting
        
        do {
            let user = try await UserAPI.register(
                email: email,
                password: password,
                phone: phone
            )
            state = .submitted(.success(user.id))
        } catch let error as APIError {
            state = .error(.serverError(error.message))
        } catch {
            state = .error(.networkError)
        }
    }
    
    func reset() {
        email = ""
        password = ""
        confirmPassword = ""
        phone = ""
        agreeToTerms = false
        state = .editing
        validationErrors.removeAll()
    }
}

// MARK: - Form View Implementation
struct UserRegistrationView: View {
    @State private var form = UserRegistrationForm()
    @FocusState private var focusedField: FormField?
    
    enum FormField {
        case email, password, confirmPassword, phone
    }
    
    var body: some View {
        NavigationStack {
            Form {
                Section("계정 정보") {
                    FormTextField(
                        title: "이메일",
                        text: $form.email,
                        isValid: form.isEmailValid,
                        errorMessage: form.validationErrors.contains(.invalidEmail) ? 
                            FormError.invalidEmail.errorDescription : nil
                    )
                    .focused($focusedField, equals: .email)
                    .textContentType(.emailAddress)
                    .keyboardType(.emailAddress)
                    .autocapitalization(.none)
                    .onChange(of: form.email) { _, _ in
                        form.validateEmail()
                    }
                    .onSubmit {
                        focusedField = .password
                    }
                    
                    FormSecureField(
                        title: "비밀번호",
                        text: $form.password,
                        isValid: form.isPasswordValid,
                        errorMessage: form.validationErrors.contains(.passwordTooShort) ?
                            FormError.passwordTooShort.errorDescription : nil
                    )
                    .focused($focusedField, equals: .password)
                    .textContentType(.newPassword)
                    .onChange(of: form.password) { _, _ in
                        form.validatePassword()
                    }
                    .onSubmit {
                        focusedField = .confirmPassword
                    }
                    
                    FormSecureField(
                        title: "비밀번호 확인",
                        text: $form.confirmPassword,
                        isValid: form.doPasswordsMatch,
                        errorMessage: !form.doPasswordsMatch && !form.confirmPassword.isEmpty ?
                            "비밀번호가 일치하지 않습니다" : nil
                    )
                    .focused($focusedField, equals: .confirmPassword)
                    .textContentType(.newPassword)
                    .onSubmit {
                        focusedField = .phone
                    }
                }
                
                Section("연락처") {
                    FormTextField(
                        title: "전화번호",
                        text: $form.phone,
                        isValid: form.isPhoneValid,
                        errorMessage: form.validationErrors.contains(.phoneInvalid) ?
                            FormError.phoneInvalid.errorDescription : nil
                    )
                    .focused($focusedField, equals: .phone)
                    .textContentType(.telephoneNumber)
                    .keyboardType(.phonePad)
                    .onChange(of: form.phone) { _, _ in
                        form.validatePhone()
                    }
                    .onSubmit {
                        focusedField = nil
                    }
                }
                
                Section {
                    Toggle("이용약관에 동의합니다", isOn: $form.agreeToTerms)
                }
                
                Section {
                    AsyncButton("가입하기") {
                        await form.submit()
                    }
                    .disabled(!form.isFormValid || form.state.isLoading)
                    .buttonStyle(.borderedProminent)
                }
            }
            .navigationTitle("회원가입")
            .navigationBarTitleDisplayMode(.large)
            .disabled(!form.state.canEdit)
            .overlay(alignment: .center) {
                if form.state.isLoading {
                    ProgressView()
                        .scaleEffect(1.5)
                        .padding()
                        .background(.regularMaterial)
                        .cornerRadius(10)
                }
            }
            .alert("가입 완료", isPresented: .constant(
                if case .submitted(.success) = form.state { true } else { false }
            )) {
                Button("확인") {
                    // 성공 후 처리
                }
            } message: {
                Text("회원가입이 완료되었습니다.")
            }
            .alert("오류", isPresented: .constant(
                if case .error = form.state { true } else { false }
            )) {
                Button("확인") {
                    form.state = .editing
                }
            } message: {
                if case .error(let error) = form.state {
                    Text(error.localizedDescription)
                }
            }
        }
    }
}

// MARK: - Custom Form Components
struct FormTextField: View {
    let title: String
    @Binding var text: String
    let isValid: Bool
    let errorMessage: String?
    
    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            HStack {
                TextField(title, text: $text)
                
                if !text.isEmpty {
                    Image(systemName: isValid ? "checkmark.circle.fill" : "exclamationmark.circle.fill")
                        .foregroundColor(isValid ? .green : .red)
                        .animation(.easeInOut, value: isValid)
                }
            }
            
            if let errorMessage = errorMessage {
                Text(errorMessage)
                    .font(.caption)
                    .foregroundColor(.red)
                    .animation(.easeInOut, value: errorMessage)
            }
        }
    }
}

struct FormSecureField: View {
    let title: String
    @Binding var text: String
    let isValid: Bool
    let errorMessage: String?
    @State private var isSecure = true
    
    var body: some View {
        VStack(alignment: .leading, spacing: 4) {
            HStack {
                if isSecure {
                    SecureField(title, text: $text)
                } else {
                    TextField(title, text: $text)
                }
                
                Button {
                    isSecure.toggle()
                } label: {
                    Image(systemName: isSecure ? "eye.slash" : "eye")
                        .foregroundColor(.secondary)
                }
                
                if !text.isEmpty {
                    Image(systemName: isValid ? "checkmark.circle.fill" : "exclamationmark.circle.fill")
                        .foregroundColor(isValid ? .green : .red)
                        .animation(.easeInOut, value: isValid)
                }
            }
            
            if let errorMessage = errorMessage {
                Text(errorMessage)
                    .font(.caption)
                    .foregroundColor(.red)
                    .animation(.easeInOut, value: errorMessage)
            }
        }
    }
}

struct AsyncButton<Label: View>: View {
    let action: () async -> Void
    @ViewBuilder let label: () -> Label
    
    @State private var isLoading = false
    
    init(
        _ title: String,
        action: @escaping () async -> Void
    ) where Label == Text {
        self.action = action
        self.label = { Text(title) }
    }
    
    init(
        action: @escaping () async -> Void,
        @ViewBuilder label: @escaping () -> Label
    ) {
        self.action = action
        self.label = label
    }
    
    var body: some View {
        Button {
            Task {
                isLoading = true
                await action()
                isLoading = false
            }
        } label: {
            HStack {
                if isLoading {
                    ProgressView()
                        .scaleEffect(0.8)
                }
                label()
            }
        }
        .disabled(isLoading)
    }
}
```

## 2. 무한 스크롤과 페이징 패턴

### LazyLoading + Pagination

```swift
// MARK: - Paginated Data Source
@Observable
class PaginatedDataSource<T: Identifiable & Sendable> {
    private(set) var items: [T] = []
    private(set) var isLoading = false
    private(set) var isLoadingMore = false
    private(set) var hasMorePages = true
    private(set) var error: Error?
    
    private var currentPage = 0
    private let pageSize: Int
    private let loadData: (Int, Int) async throws -> PaginationResponse<T>
    
    init(
        pageSize: Int = 20,
        loadData: @escaping (Int, Int) async throws -> PaginationResponse<T>
    ) {
        self.pageSize = pageSize
        self.loadData = loadData
    }
    
    func loadFirstPage() async {
        guard !isLoading else { return }
        
        isLoading = true
        error = nil
        currentPage = 0
        
        do {
            let response = try await loadData(currentPage, pageSize)
            items = response.items
            hasMorePages = response.hasMore
            currentPage += 1
        } catch {
            self.error = error
        }
        
        isLoading = false
    }
    
    func loadMoreIfNeeded(for item: T) async {
        guard !isLoadingMore,
              hasMorePages,
              items.last?.id == item.id else { return }
        
        await loadNextPage()
    }
    
    private func loadNextPage() async {
        isLoadingMore = true
        
        do {
            let response = try await loadData(currentPage, pageSize)
            items.append(contentsOf: response.items)
            hasMorePages = response.hasMore
            currentPage += 1
        } catch {
            self.error = error
        }
        
        isLoadingMore = false
    }
    
    func refresh() async {
        await loadFirstPage()
    }
}

struct PaginationResponse<T> {
    let items: [T]
    let hasMore: Bool
    let totalCount: Int?
}

// MARK: - Infinite Scroll View
struct InfiniteScrollView<Item: Identifiable & Sendable, Content: View>: View {
    @State private var dataSource: PaginatedDataSource<Item>
    private let content: (Item) -> Content
    private let onRefresh: (() async -> Void)?
    
    init(
        pageSize: Int = 20,
        loadData: @escaping (Int, Int) async throws -> PaginationResponse<Item>,
        onRefresh: (() async -> Void)? = nil,
        @ViewBuilder content: @escaping (Item) -> Content
    ) {
        self._dataSource = State(wrappedValue: PaginatedDataSource(
            pageSize: pageSize,
            loadData: loadData
        ))
        self.onRefresh = onRefresh
        self.content = content
    }
    
    var body: some View {
        ScrollView {
            LazyVStack(spacing: 12) {
                ForEach(dataSource.items) { item in
                    content(item)
                        .task {
                            await dataSource.loadMoreIfNeeded(for: item)
                        }
                }
                
                if dataSource.isLoadingMore {
                    HStack {
                        ProgressView()
                            .scaleEffect(0.8)
                        Text("더 불러오는 중...")
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }
                    .padding()
                }
                
                if !dataSource.hasMorePages && !dataSource.items.isEmpty {
                    Text("모든 항목을 불러왔습니다")
                        .font(.caption)
                        .foregroundColor(.secondary)
                        .padding()
                }
            }
            .padding()
        }
        .refreshable {
            if let onRefresh = onRefresh {
                await onRefresh()
            } else {
                await dataSource.refresh()
            }
        }
        .overlay {
            if dataSource.isLoading {
                ProgressView("불러오는 중...")
                    .frame(maxWidth: .infinity, maxHeight: .infinity)
                    .background(.regularMaterial)
            }
        }
        .overlay {
            if dataSource.items.isEmpty && !dataSource.isLoading {
                ContentUnavailableView(
                    "데이터가 없습니다",
                    systemImage: "list.bullet",
                    description: Text("새로고침하여 다시 시도해보세요")
                )
            }
        }
        .alert("오류 발생", isPresented: .constant(dataSource.error != nil)) {
            Button("재시도") {
                Task {
                    await dataSource.refresh()
                }
            }
            Button("취소", role: .cancel) {
                // dataSource.error = nil로 하면 binding이 깨짐
            }
        } message: {
            if let error = dataSource.error {
                Text(error.localizedDescription)
            }
        }
        .task {
            if dataSource.items.isEmpty {
                await dataSource.loadFirstPage()
            }
        }
    }
}

// MARK: - 실제 사용 예시
struct OrderListView: View {
    var body: some View {
        NavigationStack {
            InfiniteScrollView(
                pageSize: 15,
                loadData: { page, size in
                    try await OrderAPI.fetchOrders(page: page, pageSize: size)
                }
            ) { order in
                OrderCardView(order: order)
                    .cardStyle()
            }
            .navigationTitle("주문 내역")
        }
    }
}
```

## 3. 복잡한 애니메이션 상태 관리

### Animation State Machine

```swift
// MARK: - 애니메이션 상태 정의
enum AnimationState: CaseIterable {
    case idle
    case loading
    case success
    case error
    case processing
    
    var duration: TimeInterval {
        switch self {
        case .idle: return 0
        case .loading: return 1.0
        case .success: return 0.5
        case .error: return 0.3
        case .processing: return 2.0
        }
    }
    
    var animation: Animation {
        switch self {
        case .idle: return .easeInOut
        case .loading: return .easeInOut(duration: duration).repeatForever()
        case .success: return .spring(response: 0.6, dampingFraction: 0.8)
        case .error: return .easeInOut(duration: duration).repeatCount(3)
        case .processing: return .linear(duration: duration)
        }
    }
}

// MARK: - Animated Button Component
struct AnimatedActionButton: View {
    let title: String
    let action: () async -> Void
    
    @State private var animationState = AnimationState.idle
    @State private var scale: CGFloat = 1.0
    @State private var rotation: Double = 0
    @State private var offset: CGFloat = 0
    @State private var opacity: Double = 1.0
    
    var body: some View {
        Button {
            Task {
                await performAction()
            }
        } label: {
            ZStack {
                // 배경 그라디언트
                RoundedRectangle(cornerRadius: 25)
                    .fill(backgroundGradient)
                    .frame(height: 50)
                
                // 아이콘과 텍스트
                HStack(spacing: 12) {
                    iconView
                    
                    if animationState != .loading {
                        Text(title)
                            .font(.headline)
                            .foregroundColor(.white)
                            .opacity(opacity)
                    }
                }
            }
        }
        .scaleEffect(scale)
        .rotationEffect(.degrees(rotation))
        .offset(x: offset)
        .disabled(animationState == .loading || animationState == .processing)
        .animation(animationState.animation, value: animationState)
        .animation(animationState.animation, value: scale)
        .animation(animationState.animation, value: rotation)
        .animation(animationState.animation, value: offset)
        .animation(.easeInOut, value: opacity)
    }
    
    @ViewBuilder
    private var iconView: some View {
        switch animationState {
        case .idle:
            Image(systemName: "arrow.right.circle.fill")
                .font(.title3)
                .foregroundColor(.white)
                
        case .loading:
            ProgressView()
                .scaleEffect(0.8)
                .progressViewStyle(CircularProgressViewStyle(tint: .white))
                
        case .success:
            Image(systemName: "checkmark.circle.fill")
                .font(.title3)
                .foregroundColor(.white)
                .scaleEffect(1.3)
                
        case .error:
            Image(systemName: "exclamationmark.circle.fill")
                .font(.title3)
                .foregroundColor(.white)
                
        case .processing:
            Image(systemName: "gear")
                .font(.title3)
                .foregroundColor(.white)
                .rotationEffect(.degrees(rotation))
        }
    }
    
    private var backgroundGradient: LinearGradient {
        switch animationState {
        case .idle:
            return LinearGradient(
                colors: [.blue, .purple],
                startPoint: .leading,
                endPoint: .trailing
            )
        case .loading:
            return LinearGradient(
                colors: [.blue.opacity(0.7), .purple.opacity(0.7)],
                startPoint: .leading,
                endPoint: .trailing
            )
        case .success:
            return LinearGradient(
                colors: [.green, .mint],
                startPoint: .leading,
                endPoint: .trailing
            )
        case .error:
            return LinearGradient(
                colors: [.red, .pink],
                startPoint: .leading,
                endPoint: .trailing
            )
        case .processing:
            return LinearGradient(
                colors: [.orange, .yellow],
                startPoint: .leading,
                endPoint: .trailing
            )
        }
    }
    
    private func performAction() async {
        // 1. 로딩 상태 시작
        animationState = .loading
        scale = 0.95
        
        do {
            // 2. 실제 작업 수행
            try await Task.sleep(for: .seconds(1)) // 실제 API 호출 대신
            
            // 3. 성공 애니메이션
            animationState = .success
            scale = 1.1
            
            // 4. 성공 유지
            try await Task.sleep(for: .seconds(0.5))
            
            // 5. 원래 상태로 복원
            await resetToIdle()
            
        } catch {
            // 에러 애니메이션
            animationState = .error
            scale = 1.0
            
            // 흔들기 효과
            for i in 0..<6 {
                offset = i % 2 == 0 ? -10 : 10
                try? await Task.sleep(for: .milliseconds(50))
            }
            offset = 0
            
            // 원래 상태로 복원
            try? await Task.sleep(for: .seconds(1))
            await resetToIdle()
        }
    }
    
    @MainActor
    private func resetToIdle() async {
        animationState = .idle
        scale = 1.0
        rotation = 0
        offset = 0
        opacity = 1.0
    }
}

// MARK: - Card Transition Animation
struct CardTransitionView<Content: View>: View {
    let content: () -> Content
    @State private var isExpanded = false
    @State private var dragOffset = CGSize.zero
    @State private var scale: CGFloat = 1.0
    
    private let maxDragDistance: CGFloat = 150
    
    var body: some View {
        content()
            .scaleEffect(scale)
            .offset(dragOffset)
            .rotation3DEffect(
                .degrees(Double(dragOffset.x) / 10),
                axis: (x: 0, y: 1, z: 0)
            )
            .gesture(
                DragGesture()
                    .onChanged { value in
                        dragOffset = value.translation
                        
                        // 거리에 따른 스케일 조정
                        let distance = sqrt(pow(value.translation.x, 2) + pow(value.translation.y, 2))
                        scale = max(0.8, 1.0 - distance / maxDragDistance * 0.2)
                    }
                    .onEnded { value in
                        let distance = sqrt(pow(value.translation.x, 2) + pow(value.translation.y, 2))
                        
                        if distance > maxDragDistance {
                            // 카드 제거
                            withAnimation(.spring(response: 0.4, dampingFraction: 0.6)) {
                                dragOffset = CGSize(
                                    width: value.translation.x * 3,
                                    height: value.translation.y * 3
                                )
                                scale = 0.1
                            }
                        } else {
                            // 원래 위치로 복원
                            withAnimation(.spring(response: 0.4, dampingFraction: 0.8)) {
                                dragOffset = .zero
                                scale = 1.0
                            }
                        }
                    }
            )
    }
}
```

## 4. 글로벌 에러 처리 패턴

### Error Handling Middleware

```swift
// MARK: - 글로벌 에러 타입
enum AppError: LocalizedError, Equatable {
    case networkUnavailable
    case serverError(code: Int, message: String)
    case authenticationRequired
    case permissionDenied
    case rateLimitExceeded
    case maintenanceMode
    case unknownError(String)
    
    var errorDescription: String? {
        switch self {
        case .networkUnavailable:
            return "인터넷 연결을 확인해주세요"
        case .serverError(_, let message):
            return message
        case .authenticationRequired:
            return "로그인이 필요합니다"
        case .permissionDenied:
            return "접근 권한이 없습니다"
        case .rateLimitExceeded:
            return "잠시 후 다시 시도해주세요"
        case .maintenanceMode:
            return "서버 점검 중입니다"
        case .unknownError(let message):
            return message
        }
    }
    
    var recoveryActions: [ErrorRecoveryAction] {
        switch self {
        case .networkUnavailable:
            return [.retry, .goOffline]
        case .serverError:
            return [.retry, .reportBug]
        case .authenticationRequired:
            return [.login, .signUp]
        case .permissionDenied:
            return [.requestPermission, .contactSupport]
        case .rateLimitExceeded:
            return [.waitAndRetry]
        case .maintenanceMode:
            return [.checkStatus]
        case .unknownError:
            return [.retry, .reportBug]
        }
    }
    
    var shouldShowInToast: Bool {
        switch self {
        case .networkUnavailable, .rateLimitExceeded:
            return true
        default:
            return false
        }
    }
}

enum ErrorRecoveryAction: CaseIterable {
    case retry
    case login
    case signUp
    case goOffline
    case reportBug
    case requestPermission
    case contactSupport
    case waitAndRetry
    case checkStatus
    
    var title: String {
        switch self {
        case .retry: return "다시 시도"
        case .login: return "로그인"
        case .signUp: return "회원가입"
        case .goOffline: return "오프라인 모드"
        case .reportBug: return "버그 신고"
        case .requestPermission: return "권한 요청"
        case .contactSupport: return "고객센터"
        case .waitAndRetry: return "잠시 후 재시도"
        case .checkStatus: return "상태 확인"
        }
    }
}

// MARK: - Error Handler Service
@Observable
class ErrorHandlerService {
    private(set) var currentError: AppError?
    private(set) var errorQueue: [AppError] = []
    private(set) var isShowingError = false
    
    private var errorTimer: Timer?
    
    func handle(_ error: Error) {
        let appError = mapToAppError(error)
        
        if appError.shouldShowInToast {
            showToast(appError)
        } else {
            showModal(appError)
        }
    }
    
    private func mapToAppError(_ error: Error) -> AppError {
        switch error {
        case let apiError as APIError:
            switch apiError {
            case .unauthorized:
                return .authenticationRequired
            case .forbidden:
                return .permissionDenied
            case .serverError(let code, let message):
                return .serverError(code: code, message: message)
            case .rateLimited:
                return .rateLimitExceeded
            case .maintenance:
                return .maintenanceMode
            }
        case let urlError as URLError:
            switch urlError.code {
            case .notConnectedToInternet, .networkConnectionLost:
                return .networkUnavailable
            default:
                return .unknownError(urlError.localizedDescription)
            }
        default:
            return .unknownError(error.localizedDescription)
        }
    }
    
    private func showModal(_ error: AppError) {
        currentError = error
        isShowingError = true
    }
    
    private func showToast(_ error: AppError) {
        errorQueue.append(error)
        processErrorQueue()
    }
    
    private func processErrorQueue() {
        guard !errorQueue.isEmpty, errorTimer == nil else { return }
        
        errorTimer = Timer.scheduledTimer(withTimeInterval: 3.0, repeats: true) { _ in
            if !self.errorQueue.isEmpty {
                self.errorQueue.removeFirst()
            }
            
            if self.errorQueue.isEmpty {
                self.errorTimer?.invalidate()
                self.errorTimer = nil
            }
        }
    }
    
    func performRecoveryAction(_ action: ErrorRecoveryAction) async {
        switch action {
        case .retry:
            // 재시도 로직
            break
        case .login:
            // 로그인 화면으로 이동
            break
        case .signUp:
            // 회원가입 화면으로 이동
            break
        case .goOffline:
            // 오프라인 모드 활성화
            break
        case .reportBug:
            // 버그 리포트 전송
            break
        case .requestPermission:
            // 권한 요청
            break
        case .contactSupport:
            // 고객센터 연결
            break
        case .waitAndRetry:
            // 30초 후 자동 재시도
            try? await Task.sleep(for: .seconds(30))
            // 재시도 로직
            break
        case .checkStatus:
            // 서버 상태 확인
            break
        }
        
        dismissCurrentError()
    }
    
    func dismissCurrentError() {
        currentError = nil
        isShowingError = false
    }
}

// MARK: - Error Handling Modifiers
extension View {
    func globalErrorHandling(_ errorHandler: ErrorHandlerService) -> some View {
        self
            .alert("오류 발생", isPresented: $errorHandler.isShowingError) {
                if let error = errorHandler.currentError {
                    ForEach(error.recoveryActions, id: \.self) { action in
                        Button(action.title) {
                            Task {
                                await errorHandler.performRecoveryAction(action)
                            }
                        }
                    }
                    
                    Button("취소", role: .cancel) {
                        errorHandler.dismissCurrentError()
                    }
                }
            } message: {
                if let error = errorHandler.currentError {
                    Text(error.localizedDescription)
                }
            }
            .overlay(alignment: .top) {
                ErrorToastView(errors: errorHandler.errorQueue)
            }
    }
}

struct ErrorToastView: View {
    let errors: [AppError]
    
    var body: some View {
        VStack {
            ForEach(Array(errors.enumerated()), id: \.offset) { index, error in
                HStack {
                    Image(systemName: "exclamationmark.triangle.fill")
                        .foregroundColor(.white)
                    
                    Text(error.localizedDescription)
                        .font(.subheadline)
                        .foregroundColor(.white)
                    
                    Spacer()
                }
                .padding()
                .background(.red)
                .cornerRadius(10)
                .transition(.move(edge: .top).combined(with: .opacity))
                .offset(y: CGFloat(index * 60))
            }
        }
        .padding()
        .animation(.spring(), value: errors.count)
    }
}
```

## 5. 데이터 동기화 패턴

### 온라인/오프라인 싱크

```swift
// MARK: - 동기화 상태 관리
enum SyncState {
    case idle
    case syncing
    case conflict(local: Any, remote: Any)
    case failed(Error)
    case completed
}

@Observable
class DataSyncManager<T: Identifiable & Codable & Sendable> {
    private(set) var syncState = SyncState.idle
    private(set) var lastSyncDate: Date?
    private(set) var pendingChanges: [T] = []
    
    private let localStore: LocalStoreProtocol<T>
    private let remoteAPI: RemoteAPIProtocol<T>
    private let conflictResolver: ConflictResolverProtocol<T>
    
    init(
        localStore: LocalStoreProtocol<T>,
        remoteAPI: RemoteAPIProtocol<T>,
        conflictResolver: ConflictResolverProtocol<T>
    ) {
        self.localStore = localStore
        self.remoteAPI = remoteAPI
        self.conflictResolver = conflictResolver
    }
    
    func sync() async {
        guard syncState != .syncing else { return }
        
        syncState = .syncing
        
        do {
            // 1. 로컬 변경사항 업로드
            try await uploadLocalChanges()
            
            // 2. 원격 변경사항 다운로드
            try await downloadRemoteChanges()
            
            // 3. 충돌 해결
            try await resolveConflicts()
            
            lastSyncDate = Date()
            syncState = .completed
            
        } catch {
            syncState = .failed(error)
        }
    }
    
    private func uploadLocalChanges() async throws {
        let localChanges = try await localStore.getPendingChanges()
        
        for change in localChanges {
            do {
                let updated = try await remoteAPI.update(change)
                try await localStore.markAsSynced(updated)
            } catch {
                // 충돌 발생 시 나중에 처리
                pendingChanges.append(change)
            }
        }
    }
    
    private func downloadRemoteChanges() async throws {
        let lastSync = lastSyncDate ?? Date.distantPast
        let remoteChanges = try await remoteAPI.getChangesSince(lastSync)
        
        for remoteItem in remoteChanges {
            if let localItem = try await localStore.find(id: remoteItem.id) {
                // 로컬에 존재하는 경우 충돌 검사
                if localItem.lastModified > remoteItem.lastModified {
                    // 로컬이 더 최신 - 충돌
                    syncState = .conflict(local: localItem, remote: remoteItem)
                    return
                }
            }
            
            try await localStore.save(remoteItem)
        }
    }
    
    private func resolveConflicts() async throws {
        for pendingChange in pendingChanges {
            let resolution = await conflictResolver.resolve(
                local: pendingChange,
                remote: try await remoteAPI.get(id: pendingChange.id)
            )
            
            switch resolution {
            case .useLocal:
                try await remoteAPI.forceUpdate(pendingChange)
            case .useRemote(let remoteItem):
                try await localStore.save(remoteItem)
            case .merge(let mergedItem):
                try await remoteAPI.update(mergedItem)
                try await localStore.save(mergedItem)
            }
        }
        
        pendingChanges.removeAll()
    }
}

enum ConflictResolution<T> {
    case useLocal
    case useRemote(T)
    case merge(T)
}

protocol ConflictResolverProtocol<T> {
    associatedtype T
    func resolve(local: T, remote: T) async -> ConflictResolution<T>
}
```

## 정리: 실전 패턴의 핵심 원칙

1. **상태를 명시적으로 모델링하라**
   - Form state, animation state, sync state 등을 enum으로 정의
   - 불가능한 상태를 컴파일 타임에 차단

2. **에러를 타입으로 표현하라**
   - 복구 가능한 에러와 복구 방법을 함께 정의
   - 사용자에게 의미있는 메시지와 액션 제공

3. **성능을 처음부터 고려하라**
   - 무한 스크롤과 페이징으로 메모리 관리
   - 필요할 때만 로드하는 lazy loading

4. **오프라인을 일급 시민으로 대우하라**
   - 동기화 로직을 명시적으로 설계
   - 충돌 해결 전략을 미리 정의

5. **애니메이션으로 상태를 표현하라**
   - 로딩, 성공, 실패 상태를 시각적으로 전달
   - 사용자의 인지 부하를 줄이는 피드백

이제 다음 레벨로 가서 이런 패턴들을 더 큰 아키텍처 안에서 조직하는 방법을 배워봅시다.

→ [L4: 시스템 아키텍처 - Clean Architecture in SwiftUI](../L4_architecture/00_clean_architecture.md)

---

*"패턴은 반복되는 문제에 대한 재사용 가능한 해답이다. 하지만 모든 문제가 못은 아니므로, 모든 해답이 망치일 필요는 없다." - Gang of Four*