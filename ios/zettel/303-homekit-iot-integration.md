# HomeKit과 IoT 통합: 스마트홈의 중추

## Core Insight
HomeKit은 단순한 기기 제어를 넘어서, 집 전체를 하나의 지능적 시스템으로 만드는 프라이버시 중심의 IoT 플랫폼이다.

HomeKit의 진정한 혁신은 기술적 복잡성을 사용자로부터 완전히 숨기는 것이다. 사용자는 "침실 불 켜줘"라고 말하기만 하면 되고, 그 뒤의 암호화, 인증, 네트워크 통신은 모두 자동으로 처리된다. 이것이 Apple다운 접근법이다.

**HomeKit 아키텍처 이해:**

**1. 기본 HomeKit 설정:**
```swift
import HomeKit

class HomeManager: NSObject, ObservableObject {
    @Published var homes: [HMHome] = []
    @Published var accessories: [HMAccessory] = []
    @Published var isAvailable = false
    
    private let homeManager = HMHomeManager()
    
    override init() {
        super.init()
        homeManager.delegate = self
    }
    
    func requestAuthorization() {
        // iOS에서 HomeKit 권한은 자동으로 요청됨
        // 사용자가 첫 번째 액세서리를 추가하려 할 때 권한 다이얼로그 표시
    }
    
    func addAccessory() {
        guard let primaryHome = homeManager.primaryHome else {
            createHome { [weak self] success in
                if success {
                    self?.startAccessoryBrowser()
                }
            }
            return
        }
        
        startAccessoryBrowser()
    }
    
    private func createHome(completion: @escaping (Bool) -> Void) {
        homeManager.addHome(withName: "우리 집") { home, error in
            if let home = home {
                print("Home created: \(home.name)")
                completion(true)
            } else {
                print("Failed to create home: \(error?.localizedDescription ?? "Unknown error")")
                completion(false)
            }
        }
    }
    
    private func startAccessoryBrowser() {
        guard let home = homeManager.primaryHome else { return }
        
        let browser = HMAccessoryBrowser()
        browser.delegate = self
        browser.startSearchingForNewAccessories()
        
        // 30초 후 검색 중단
        DispatchQueue.main.asyncAfter(deadline: .now() + 30) {
            browser.stopSearchingForNewAccessories()
        }
    }
}

// MARK: - HMHomeManagerDelegate
extension HomeManager: HMHomeManagerDelegate {
    func homeManagerDidUpdateHomes(_ manager: HMHomeManager) {
        DispatchQueue.main.async {
            self.homes = manager.homes
            self.isAvailable = true
        }
    }
    
    func homeManagerDidUpdatePrimaryHome(_ manager: HMHomeManager) {
        guard let primaryHome = manager.primaryHome else { return }
        
        DispatchQueue.main.async {
            self.accessories = primaryHome.accessories
        }
    }
}

// MARK: - HMAccessoryBrowserDelegate
extension HomeManager: HMAccessoryBrowserDelegate {
    func accessoryBrowser(_ browser: HMAccessoryBrowser, didFindNewAccessory accessory: HMAccessory) {
        print("Found accessory: \(accessory.name)")
        
        // 자동으로 추가하거나 사용자에게 선택 옵션 제공
        addAccessoryToHome(accessory)
    }
    
    func accessoryBrowser(_ browser: HMAccessoryBrowser, didRemoveNewAccessory accessory: HMAccessory) {
        print("Accessory removed: \(accessory.name)")
    }
    
    private func addAccessoryToHome(_ accessory: HMAccessory) {
        guard let home = homeManager.primaryHome else { return }
        
        home.addAccessory(accessory) { error in
            if let error = error {
                print("Failed to add accessory: \(error.localizedDescription)")
            } else {
                print("Successfully added accessory: \(accessory.name)")
                
                DispatchQueue.main.async {
                    self.accessories = home.accessories
                }
            }
        }
    }
}
```

**2. 액세서리 제어 시스템:**
```swift
class AccessoryController {
    private let home: HMHome
    
    init(home: HMHome) {
        self.home = home
    }
    
    // 조명 제어
    func controlLight(_ accessory: HMAccessory, brightness: Int, color: UIColor? = nil) {
        guard let lightbulb = accessory.services.first(where: { $0.serviceType == HMServiceTypeLightbulb }) else {
            print("This accessory is not a lightbulb")
            return
        }
        
        // 밝기 조절
        if let brightnessCharacteristic = lightbulb.characteristics.first(where: { 
            $0.characteristicType == HMCharacteristicTypeBrightness 
        }) {
            brightnessCharacteristic.writeValue(brightness) { error in
                if let error = error {
                    print("Failed to set brightness: \(error.localizedDescription)")
                } else {
                    print("Brightness set to \(brightness)%")
                }
            }
        }
        
        // 색상 조절 (지원하는 경우)
        if let color = color {
            setLightColor(lightbulb, color: color)
        }
    }
    
    private func setLightColor(_ service: HMService, color: UIColor) {
        var hue: CGFloat = 0
        var saturation: CGFloat = 0
        color.getHue(&hue, saturation: &saturation, brightness: nil, alpha: nil)
        
        // Hue 설정
        if let hueCharacteristic = service.characteristics.first(where: { 
            $0.characteristicType == HMCharacteristicTypeHue 
        }) {
            let hueValue = Int(hue * 360)
            hueCharacteristic.writeValue(hueValue) { error in
                if let error = error {
                    print("Failed to set hue: \(error.localizedDescription)")
                }
            }
        }
        
        // Saturation 설정
        if let saturationCharacteristic = service.characteristics.first(where: { 
            $0.characteristicType == HMCharacteristicTypeSaturation 
        }) {
            let saturationValue = Int(saturation * 100)
            saturationCharacteristic.writeValue(saturationValue) { error in
                if let error = error {
                    print("Failed to set saturation: \(error.localizedDescription)")
                }
            }
        }
    }
    
    // 온도 조절기 제어
    func setThermostat(_ accessory: HMAccessory, targetTemperature: Double, mode: ThermostatMode) {
        guard let thermostat = accessory.services.first(where: { 
            $0.serviceType == HMServiceTypeThermostat 
        }) else { return }
        
        // 목표 온도 설정
        if let tempCharacteristic = thermostat.characteristics.first(where: { 
            $0.characteristicType == HMCharacteristicTypeTargetTemperature 
        }) {
            tempCharacteristic.writeValue(targetTemperature) { error in
                if let error = error {
                    print("Failed to set temperature: \(error.localizedDescription)")
                } else {
                    print("Target temperature set to \(targetTemperature)°C")
                }
            }
        }
        
        // 모드 설정 (난방/냉방/자동/끄기)
        if let modeCharacteristic = thermostat.characteristics.first(where: { 
            $0.characteristicType == HMCharacteristicTypeTargetHeatingCoolingState 
        }) {
            modeCharacteristic.writeValue(mode.rawValue) { error in
                if let error = error {
                    print("Failed to set thermostat mode: \(error.localizedDescription)")
                }
            }
        }
    }
    
    enum ThermostatMode: Int {
        case off = 0
        case heat = 1
        case cool = 2
        case auto = 3
    }
}
```

**3. 씬(Scene)과 자동화 시스템:**
```swift
class SceneManager {
    private let home: HMHome
    
    init(home: HMHome) {
        self.home = home
    }
    
    func createGoodMorningScene() {
        home.addActionSet(withName: "좋은 아침") { [weak self] actionSet, error in
            guard let actionSet = actionSet, error == nil else {
                print("Failed to create action set: \(error?.localizedDescription ?? "Unknown error")")
                return
            }
            
            self?.addActionsToMorningScene(actionSet)
        }
    }
    
    private func addActionsToMorningScene(_ actionSet: HMActionSet) {
        // 침실 조명 서서히 켜기
        if let bedroomLight = findAccessory(name: "침실 조명") {
            addLightAction(to: actionSet, accessory: bedroomLight, brightness: 70)
        }
        
        // 거실 온도 조절
        if let thermostat = findAccessory(name: "거실 온도조절기") {
            addThermostatAction(to: actionSet, accessory: thermostat, temperature: 22.0)
        }
        
        // 커피머신 켜기
        if let coffeeMachine = findAccessory(name: "커피머신") {
            addSwitchAction(to: actionSet, accessory: coffeeMachine, isOn: true)
        }
    }
    
    private func addLightAction(to actionSet: HMActionSet, accessory: HMAccessory, brightness: Int) {
        guard let lightService = accessory.services.first(where: { $0.serviceType == HMServiceTypeLightbulb }),
              let brightnessCharacteristic = lightService.characteristics.first(where: { 
                  $0.characteristicType == HMCharacteristicTypeBrightness 
              }) else { return }
        
        let action = HMCharacteristicWriteAction(characteristic: brightnessCharacteristic, targetValue: brightness)
        
        actionSet.addAction(action) { error in
            if let error = error {
                print("Failed to add light action: \(error.localizedDescription)")
            }
        }
    }
    
    func executeScene(named sceneName: String) {
        guard let scene = home.actionSets.first(where: { $0.name == sceneName }) else {
            print("Scene '\(sceneName)' not found")
            return
        }
        
        home.executeActionSet(scene) { error in
            if let error = error {
                print("Failed to execute scene: \(error.localizedDescription)")
            } else {
                print("Scene '\(sceneName)' executed successfully")
            }
        }
    }
    
    private func findAccessory(name: String) -> HMAccessory? {
        return home.accessories.first { $0.name.contains(name) }
    }
}
```

**4. 트리거 기반 자동화:**
```swift
class AutomationManager {
    private let home: HMHome
    
    init(home: HMHome) {
        self.home = home
    }
    
    func createLocationBasedAutomation() {
        // 집에 도착했을 때 트리거
        let predicate = HMEventTrigger.predicateForEvaluatingTrigger(withPresence: HMPresenceEvent(presenceEventType: .everyEntry, presenceUserType: .currentUser))
        
        guard let welcomeScene = home.actionSets.first(where: { $0.name == "집 도착" }) else {
            print("Welcome scene not found")
            return
        }
        
        home.addTrigger(HMEventTrigger(name: "집 도착 자동화", events: [], endEvents: [], recurrences: [], predicate: predicate)) { trigger, error in
            if let trigger = trigger as? HMEventTrigger, error == nil {
                trigger.addActionSet(welcomeScene) { error in
                    if let error = error {
                        print("Failed to add action set to trigger: \(error.localizedDescription)")
                    } else {
                        print("Location-based automation created successfully")
                    }
                }
            }
        }
    }
    
    func createTimeBasedAutomation() {
        // 매일 저녁 7시에 실행
        let dateComponents = DateComponents(hour: 19, minute: 0)
        let trigger = HMTimerTrigger(name: "저녁 조명", fireDate: Date(), timeZone: nil, recurrence: .daily, recurrenceCalendar: nil)
        
        home.addTrigger(trigger) { trigger, error in
            if let trigger = trigger, error == nil {
                // 저녁 씬 추가
                if let eveningScene = self.home.actionSets.first(where: { $0.name == "저녁 분위기" }) {
                    trigger.addActionSet(eveningScene) { error in
                        if let error = error {
                            print("Failed to add evening scene: \(error.localizedDescription)")
                        }
                    }
                }
            }
        }
    }
    
    func createSensorBasedAutomation() {
        // 움직임 감지 시 조명 켜기
        guard let motionSensor = home.accessories.first(where: { 
            $0.services.contains { $0.serviceType == HMServiceTypeMotionSensor }
        }) else { return }
        
        guard let motionService = motionSensor.services.first(where: { $0.serviceType == HMServiceTypeMotionSensor }),
              let motionCharacteristic = motionService.characteristics.first(where: { 
                  $0.characteristicType == HMCharacteristicTypeMotionDetected 
              }) else { return }
        
        let motionEvent = HMCharacteristicEvent(characteristic: motionCharacteristic, triggerValue: true)
        let trigger = HMEventTrigger(name: "움직임 감지 조명", events: [motionEvent], predicate: nil)
        
        home.addTrigger(trigger) { trigger, error in
            if let error = error {
                print("Failed to create motion trigger: \(error.localizedDescription)")
            } else {
                print("Motion-based automation created")
            }
        }
    }
}
```

**5. Siri와 음성 제어:**
```swift
import Intents

class SiriIntegration {
    func setupSiriShortcuts() {
        // "좋은 밤" 단축어 등록
        let intent = HMIntent()
        intent.suggestedInvocationPhrase = "좋은 밤"
        
        let interaction = INInteraction(intent: intent, response: nil)
        interaction.donate { error in
            if let error = error {
                print("Failed to donate shortcut: \(error.localizedDescription)")
            } else {
                print("Siri shortcut donated successfully")
            }
        }
    }
    
    func handleSiriRequest(_ intent: HMIntent) {
        // Siri 요청 처리
        switch intent.intentName {
        case "turn_on_lights":
            executeScene(named: "모든 조명 켜기")
        case "goodnight":
            executeScene(named: "잠자리")
        case "movie_time":
            executeScene(named: "영화 시청")
        default:
            print("Unknown Siri intent: \(intent.intentName)")
        }
    }
    
    private func executeScene(named sceneName: String) {
        // SceneManager를 통한 씬 실행
        print("Executing scene via Siri: \(sceneName)")
    }
}
```

**6. Apple Watch 통합:**
```swift
import WatchConnectivity

class WatchKitIntegration: NSObject, WCSessionDelegate {
    private let session = WCSession.default
    
    override init() {
        super.init()
        if WCSession.isSupported() {
            session.delegate = self
            session.activate()
        }
    }
    
    func sendAccessoryListToWatch() {
        guard session.isReachable else { return }
        
        let accessories = HomeManager.shared.accessories.map { accessory in
            [
                "name": accessory.name,
                "identifier": accessory.uniqueIdentifier.uuidString,
                "category": accessory.category.categoryType
            ]
        }
        
        session.sendMessage(["accessories": accessories], replyHandler: nil) { error in
            print("Failed to send to watch: \(error.localizedDescription)")
        }
    }
    
    // MARK: - WCSessionDelegate
    func session(_ session: WCSession, activationDidCompleteWith activationState: WCSessionActivationState, error: Error?) {
        if let error = error {
            print("Watch session activation failed: \(error.localizedDescription)")
        } else {
            print("Watch session activated")
        }
    }
    
    func sessionDidBecomeInactive(_ session: WCSession) {}
    func sessionDidDeactivate(_ session: WCSession) {}
    
    func session(_ session: WCSession, didReceiveMessage message: [String : Any]) {
        // Apple Watch에서 온 제어 명령 처리
        if let action = message["action"] as? String,
           let accessoryID = message["accessoryID"] as? String {
            handleWatchCommand(action: action, accessoryID: accessoryID)
        }
    }
    
    private func handleWatchCommand(action: String, accessoryID: String) {
        // 워치에서 온 명령 실행
        switch action {
        case "toggle_light":
            toggleLight(accessoryID: accessoryID)
        case "lock_door":
            lockDoor(accessoryID: accessoryID)
        default:
            print("Unknown watch command: \(action)")
        }
    }
}
```

**7. 보안과 프라이버시:**
```swift
class HomeKitSecurity {
    static func checkSecurityBestPractices(home: HMHome) -> [SecurityRecommendation] {
        var recommendations: [SecurityRecommendation] = []
        
        // 사용자 권한 검토
        for user in home.users {
            if user.privilege == .admin && user != home.currentUser {
                recommendations.append(.reviewAdminPrivileges(user.name))
            }
        }
        
        // 오래된 액세서리 확인
        for accessory in home.accessories {
            if let firmwareVersion = accessory.firmwareVersion {
                if isOutdatedFirmware(firmwareVersion) {
                    recommendations.append(.updateFirmware(accessory.name))
                }
            }
        }
        
        // 2단계 인증 확인
        if !home.homeHubState.contains(.connected) {
            recommendations.append(.setupHomeHub)
        }
        
        return recommendations
    }
    
    enum SecurityRecommendation {
        case reviewAdminPrivileges(String)
        case updateFirmware(String)
        case setupHomeHub
        case enableNotifications
    }
    
    private static func isOutdatedFirmware(_ version: String) -> Bool {
        // 펌웨어 버전 검사 로직
        return false // placeholder
    }
}
```

HomeKit은 IoT의 복잡성을 추상화하여 일반 사용자도 쉽게 스마트홈을 구축할 수 있게 한다. 가장 중요한 것은 프라이버시다. HomeKit 기기들은 Apple 서버를 거치지 않고 직접 통신하며, 모든 데이터는 end-to-end 암호화된다. 이는 편의성과 프라이버시를 동시에 달성한 Apple의 철학이 잘 드러나는 영역이다.

## Connections
→ [[304-carplay-external-display]]
→ [[305-internationalization-advanced]]
← [[302-arkit-lidar-capabilities]]
← [[210-keychain-security-model]]

---
Level: L3
Date: 2025-08-16
Tags: #homekit #iot #smart-home #automation #siri #privacy #security