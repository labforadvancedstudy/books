# CarPlay와 외부 디스플레이: 모바일 경험의 확장

## Core Insight
CarPlay는 단순한 화면 미러링이 아닌, 운전 중 안전성을 위해 특별히 설계된 적응형 인터페이스로, 앱의 본질을 자동차 환경에 맞게 재해석한다.

CarPlay의 철학은 제약을 통한 혁신이다. 운전 중에는 복잡한 인터페이스가 생명을 위험에 빠뜨릴 수 있기 때문에, Apple은 의도적으로 기능을 제한하고 단순화했다. 이 제약 안에서 최고의 사용자 경험을 만드는 것이 CarPlay 앱 개발의 핵심이다.

**CarPlay 앱 카테고리와 템플릿:**

**1. 기본 CarPlay 설정:**
```swift
import CarPlay

class CarPlaySceneDelegate: UIResponder, CPTemplateApplicationSceneDelegate {
    var interfaceController: CPInterfaceController?
    
    // MARK: - CPTemplateApplicationSceneDelegate
    func templateApplicationScene(_ templateApplicationScene: CPTemplateApplicationScene,
                                 didConnect interfaceController: CPInterfaceController) {
        self.interfaceController = interfaceController
        setupRootTemplate()
    }
    
    func templateApplicationScene(_ templateApplicationScene: CPTemplateApplicationScene,
                                 didDisconnectInterfaceController interfaceController: CPInterfaceController) {
        self.interfaceController = nil
    }
    
    private func setupRootTemplate() {
        // 탭 바 기반 네비게이션
        let homeTab = createHomeTab()
        let navigationTab = createNavigationTab()
        let musicTab = createMusicTab()
        
        let tabBarTemplate = CPTabBarTemplate(templates: [homeTab, navigationTab, musicTab])
        interfaceController?.setRootTemplate(tabBarTemplate, animated: true, completion: nil)
    }
    
    private func createHomeTab() -> CPListTemplate {
        let recentTrips = CPListItem(text: "최근 여행", detailText: "3개의 저장된 경로")
        let favorites = CPListItem(text: "즐겨찾기", detailText: "집, 회사, 카페")
        let settings = CPListItem(text: "설정", detailText: "앱 환경설정")
        
        recentTrips.setHandler { [weak self] item, completion in
            self?.showRecentTrips()
            completion()
        }
        
        let section = CPListSection(items: [recentTrips, favorites, settings])
        let listTemplate = CPListTemplate(title: "홈", sections: [section])
        listTemplate.tabTitle = "홈"
        listTemplate.tabImage = UIImage(systemName: "house.fill")
        
        return listTemplate
    }
}
```

**2. 네비게이션 앱 구현:**
```swift
class NavigationCarPlayManager {
    private var interfaceController: CPInterfaceController?
    private var mapTemplate: CPMapTemplate?
    
    func setupNavigationTemplate() {
        mapTemplate = CPMapTemplate()
        mapTemplate?.showPanningInterface = false
        
        // 네비게이션 바 버튼
        let searchButton = CPBarButton(title: "검색") { [weak self] _ in
            self?.showSearchInterface()
        }
        
        let voiceButton = CPBarButton(title: "음성") { [weak self] _ in
            self?.startVoiceInput()
        }
        
        mapTemplate?.leadingNavigationBarButtons = [searchButton]
        mapTemplate?.trailingNavigationBarButtons = [voiceButton]
        
        // 여행 추정 정보
        let travelEstimates = CPTravelEstimates(distanceRemaining: Measurement(value: 5.2, unit: UnitLength.kilometers),
                                              timeRemaining: 420) // 7분
        mapTemplate?.updateEstimates(travelEstimates, for: currentTrip)
        
        interfaceController?.pushTemplate(mapTemplate!, animated: true, completion: nil)
    }
    
    private func showSearchInterface() {
        let searchTemplate = CPSearchTemplate()
        searchTemplate.delegate = self
        
        interfaceController?.presentTemplate(searchTemplate, animated: true, completion: nil)
    }
    
    private func startNavigation(to destination: CLLocationCoordinate2D, name: String) {
        let maneuvers = generateManeuvers(to: destination)
        let trip = CPTrip(origin: MKMapItem.forCurrentLocation(), 
                         destination: MKMapItem(placemark: MKPlacemark(coordinate: destination)),
                         routeChoices: [])
        
        trip.userInfo = ["destinationName": name]
        
        mapTemplate?.startNavigation(using: trip)
        
        // 첫 번째 안내 표시
        if let firstManeuver = maneuvers.first {
            mapTemplate?.showManeuvers([firstManeuver])
        }
    }
    
    private func generateManeuvers(to destination: CLLocationCoordinate2D) -> [CPManeuver] {
        // 실제로는 MapKit Directions API 사용
        let maneuver = CPManeuver()
        maneuver.instructionVariants = ["500m 직진 후 우회전"]
        maneuver.initialTravelEstimates = CPTravelEstimates(distanceRemaining: Measurement(value: 500, unit: UnitLength.meters),
                                                           timeRemaining: 60)
        maneuver.symbolImage = UIImage(systemName: "arrow.turn.up.right")
        
        return [maneuver]
    }
}

// MARK: - CPSearchTemplateDelegate
extension NavigationCarPlayManager: CPSearchTemplateDelegate {
    func searchTemplate(_ searchTemplate: CPSearchTemplate, updatedSearchText searchText: String, completionHandler: @escaping ([CPListItem]) -> Void) {
        // 실시간 검색 결과
        searchPlaces(query: searchText) { results in
            let listItems = results.map { place in
                let item = CPListItem(text: place.name, detailText: place.address)
                item.setHandler { [weak self] _, completion in
                    self?.startNavigation(to: place.coordinate, name: place.name)
                    completion()
                }
                return item
            }
            completionHandler(listItems)
        }
    }
    
    func searchTemplate(_ searchTemplate: CPSearchTemplate, selectedResult item: CPListItem, completionHandler: @escaping () -> Void) {
        // 검색 결과 선택
        completionHandler()
    }
}
```

**3. 음악 앱 CarPlay 통합:**
```swift
import MediaPlayer

class MusicCarPlayManager {
    private var interfaceController: CPInterfaceController?
    private var nowPlayingTemplate: CPNowPlayingTemplate?
    
    func setupMusicTemplate() {
        let browseTemplate = createBrowseTemplate()
        interfaceController?.pushTemplate(browseTemplate, animated: true, completion: nil)
        
        // Now Playing 템플릿 설정
        setupNowPlayingTemplate()
    }
    
    private func createBrowseTemplate() -> CPListTemplate {
        // 플레이리스트 섹션
        let recentlyPlayed = CPListItem(text: "최근 재생", detailText: "마지막으로 들은 음악")
        let playlists = CPListItem(text: "플레이리스트", detailText: "12개 목록")
        let favorites = CPListItem(text: "좋아요 표시한 곡", detailText: "147곡")
        
        recentlyPlayed.setHandler { [weak self] _, completion in
            self?.showRecentlyPlayed()
            completion()
        }
        
        playlists.setHandler { [weak self] _, completion in
            self?.showPlaylists()
            completion()
        }
        
        let section = CPListSection(items: [recentlyPlayed, playlists, favorites])
        let template = CPListTemplate(title: "음악", sections: [section])
        template.tabTitle = "음악"
        template.tabImage = UIImage(systemName: "music.note")
        
        return template
    }
    
    private func setupNowPlayingTemplate() {
        nowPlayingTemplate = CPNowPlayingTemplate.shared
        
        // 커스텀 버튼 추가
        let likeButton = CPNowPlayingButton(handler: { [weak self] _ in
            self?.toggleLike()
        })
        likeButton.image = UIImage(systemName: "heart")
        
        let shuffleButton = CPNowPlayingButton(handler: { [weak self] _ in
            self?.toggleShuffle()
        })
        shuffleButton.image = UIImage(systemName: "shuffle")
        
        nowPlayingTemplate?.updateNowPlayingButtons([likeButton, shuffleButton])
    }
    
    private func showPlaylists() {
        fetchPlaylists { [weak self] playlists in
            let playlistItems = playlists.map { playlist in
                let item = CPListItem(text: playlist.name, 
                                    detailText: "\(playlist.songCount)곡",
                                    image: playlist.artwork)
                item.setHandler { _, completion in
                    self?.playPlaylist(playlist)
                    completion()
                }
                return item
            }
            
            let section = CPListSection(items: playlistItems)
            let template = CPListTemplate(title: "플레이리스트", sections: [section])
            
            self?.interfaceController?.pushTemplate(template, animated: true, completion: nil)
        }
    }
    
    private func playPlaylist(_ playlist: Playlist) {
        // Media Player 프레임워크를 통한 재생
        let player = MPMusicPlayerController.applicationMusicPlayer
        player.setQueue(with: playlist.songs.map { $0.id })
        player.play()
        
        // Now Playing 화면으로 전환
        interfaceController?.presentTemplate(nowPlayingTemplate!, animated: true, completion: nil)
    }
    
    private func toggleLike() {
        guard let currentSong = MPMusicPlayerController.applicationMusicPlayer.nowPlayingItem else { return }
        
        // 좋아요 상태 토글
        let isLiked = !currentSong.isLiked
        currentSong.isLiked = isLiked
        
        // 버튼 이미지 업데이트
        let heartImage = UIImage(systemName: isLiked ? "heart.fill" : "heart")
        // 버튼 업데이트 로직...
    }
}
```

**4. 커뮤니케이션 앱 (전화/메시지):**
```swift
import Contacts
import CallKit

class CommunicationCarPlayManager {
    private var interfaceController: CPInterfaceController?
    
    func setupCommunicationTemplate() {
        let contactsTemplate = createContactsTemplate()
        interfaceController?.pushTemplate(contactsTemplate, animated: true, completion: nil)
    }
    
    private func createContactsTemplate() -> CPContactTemplate {
        let contact = CNContact()
        // 연락처 정보 설정...
        
        let callButton = CPContactButton(handler: { [weak self] _ in
            self?.makeCall(to: contact.phoneNumbers.first?.value.stringValue ?? "")
        })
        callButton.image = UIImage(systemName: "phone.fill")
        
        let messageButton = CPContactButton(handler: { [weak self] _ in
            self?.sendMessage(to: contact.phoneNumbers.first?.value.stringValue ?? "")
        })
        messageButton.image = UIImage(systemName: "message.fill")
        
        let template = CPContactTemplate(contact: contact)
        template.contactButtons = [callButton, messageButton]
        
        return template
    }
    
    private func makeCall(to phoneNumber: String) {
        let callHandle = CXHandle(type: .phoneNumber, value: phoneNumber)
        let startCallAction = CXStartCallAction(call: UUID(), handle: callHandle)
        
        let transaction = CXTransaction(action: startCallAction)
        let callController = CXCallController()
        
        callController.request(transaction) { error in
            if let error = error {
                print("Failed to start call: \(error.localizedDescription)")
            }
        }
    }
    
    private func sendMessage(to phoneNumber: String) {
        // 음성 입력을 통한 메시지 전송
        startVoiceInput { [weak self] messageText in
            self?.sendSMS(to: phoneNumber, message: messageText)
        }
    }
    
    private func startVoiceInput(completion: @escaping (String) -> Void) {
        // 음성 인식 시작
        // CarPlay에서는 Siri를 통한 음성 입력 권장
        completion("음성으로 입력된 메시지")
    }
}
```

**5. 외부 디스플레이 일반 지원:**
```swift
import UIKit

class ExternalDisplayManager {
    private var externalWindow: UIWindow?
    private var externalScreen: UIScreen?
    
    init() {
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(screenDidConnect),
            name: UIScreen.didConnectNotification,
            object: nil
        )
        
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(screenDidDisconnect),
            name: UIScreen.didDisconnectNotification,
            object: nil
        )
    }
    
    @objc private func screenDidConnect(_ notification: Notification) {
        guard let screen = notification.object as? UIScreen else { return }
        
        setupExternalDisplay(screen: screen)
    }
    
    @objc private func screenDidDisconnect(_ notification: Notification) {
        guard let screen = notification.object as? UIScreen,
              screen == externalScreen else { return }
        
        teardownExternalDisplay()
    }
    
    private func setupExternalDisplay(screen: UIScreen) {
        externalScreen = screen
        
        // 외부 화면용 윈도우 생성
        externalWindow = UIWindow(frame: screen.bounds)
        externalWindow?.screen = screen
        
        // 외부 화면용 뷰 컨트롤러
        let externalViewController = ExternalDisplayViewController()
        externalWindow?.rootViewController = externalViewController
        externalWindow?.isHidden = false
        
        // 해상도별 최적화
        optimizeForResolution(screen.bounds.size)
    }
    
    private func optimizeForResolution(_ size: CGSize) {
        // 4K, 1080p 등 해상도별 최적화
        let scale = UIScreen.main.scale
        
        if size.width >= 3840 { // 4K
            // 4K 디스플레이 최적화
            print("Optimizing for 4K display")
        } else if size.width >= 1920 { // 1080p
            // 1080p 디스플레이 최적화
            print("Optimizing for 1080p display")
        }
    }
    
    private func teardownExternalDisplay() {
        externalWindow?.isHidden = true
        externalWindow = nil
        externalScreen = nil
    }
}

class ExternalDisplayViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.backgroundColor = .black
        setupExternalInterface()
    }
    
    private func setupExternalInterface() {
        // 외부 디스플레이용 인터페이스 구성
        let titleLabel = UILabel()
        titleLabel.text = "외부 디스플레이 모드"
        titleLabel.textColor = .white
        titleLabel.font = UIFont.systemFont(ofSize: 48)
        titleLabel.textAlignment = .center
        
        view.addSubview(titleLabel)
        titleLabel.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            titleLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            titleLabel.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
        
        // 프레젠테이션 모드용 컨텐츠 추가
        setupPresentationContent()
    }
    
    private func setupPresentationContent() {
        // 프레젠테이션, 비디오 재생 등을 위한 컨텐츠
    }
}
```

**6. CarPlay 시뮬레이터와 테스팅:**
```swift
class CarPlayTestManager {
    static func setupTestEnvironment() {
        #if targetEnvironment(simulator)
        // 시뮬레이터에서 CarPlay 테스트 설정
        print("CarPlay Simulator Mode")
        
        // 가상 CarPlay 연결 시뮬레이션
        simulateCarPlayConnection()
        #endif
    }
    
    #if targetEnvironment(simulator)
    private static func simulateCarPlayConnection() {
        // CarPlay 연결 시뮬레이션
        let notification = Notification(name: .carPlayDidConnect)
        NotificationCenter.default.post(notification)
    }
    #endif
    
    static func testVoiceCommands() {
        // 음성 명령 테스트
        let voiceCommands = [
            "음악 재생",
            "집으로 가는 길 안내",
            "엄마에게 전화걸어",
            "메시지 읽어줘"
        ]
        
        for command in voiceCommands {
            processVoiceCommand(command)
        }
    }
    
    private static func processVoiceCommand(_ command: String) {
        print("Processing voice command: \(command)")
        // 실제 음성 명령 처리 로직
    }
}

extension Notification.Name {
    static let carPlayDidConnect = Notification.Name("CarPlayDidConnect")
    static let carPlayDidDisconnect = Notification.Name("CarPlayDidDisconnect")
}
```

**7. CarPlay 접근성과 안전성:**
```swift
class CarPlayAccessibilityManager {
    static func setupAccessibility() {
        // 음성 안내 최적화
        UIAccessibility.post(notification: .screenChanged, argument: "CarPlay 모드 활성화")
        
        // 운전 중 안전을 위한 제스처 제한
        limitGestureComplexity()
    }
    
    private static func limitGestureComplexity() {
        // 복잡한 제스처 비활성화
        // 큰 터치 영역 확보
        // 단순한 탭 제스처만 허용
    }
    
    static func provideDrivingModeAlerts() {
        // 운전 모드 알림
        let alert = UIAlertController(
            title: "운전 중 안전 모드",
            message: "일부 기능이 제한됩니다.",
            preferredStyle: .alert
        )
        
        alert.addAction(UIAlertAction(title: "확인", style: .default))
        
        // 현재 표시 중인 뷰 컨트롤러에 표시
    }
}
```

CarPlay는 iOS 앱이 자동차라는 완전히 다른 환경에서 작동하도록 하는 플랫폼이다. 이는 단순한 포팅이 아니라, 안전성과 사용성을 위한 근본적인 재설계를 요구한다. 성공적인 CarPlay 앱은 운전자의 주의를 분산시키지 않으면서도 핵심 기능을 효과적으로 제공한다.

## Connections
→ [[305-internationalization-advanced]]
→ [[306-memory-management-edge-cases]]
← [[303-homekit-iot-integration]]
← [[220-push-notifications-ecosystem]]

---
Level: L3
Date: 2025-08-16
Tags: #carplay #external-display #automotive #safety #voice-control #navigation #music