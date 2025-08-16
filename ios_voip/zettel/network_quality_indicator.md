# 네트워크 상태 표시 UI

## 개념
VoIP 통화 중 실시간 네트워크 품질을 시각적으로 표시하여 사용자가 통화 상태를 즉각적으로 인지.

## UI 컴포넌트 구현
```swift
class NetworkQualityView: UIView {
    private let signalBars = [UIView]()
    private var currentQuality: NetworkQuality = .unknown
    
    enum NetworkQuality: Int {
        case excellent = 5  // 녹색 5칸
        case good = 4       // 녹색 4칸
        case fair = 3       // 노란색 3칸
        case poor = 2       // 주황색 2칸
        case bad = 1        // 빨간색 1칸
        case unknown = 0    // 회색
    }
    
    func updateQuality(from stats: NetworkStats) {
        // RTT, 패킷 손실, Jitter 기반 판단
        if stats.rtt < 150 && stats.packetLoss < 1 {
            currentQuality = .excellent
        } else if stats.rtt < 300 && stats.packetLoss < 3 {
            currentQuality = .good
        } else if stats.rtt < 500 && stats.packetLoss < 5 {
            currentQuality = .fair
        } else if stats.packetLoss < 10 {
            currentQuality = .poor
        } else {
            currentQuality = .bad
        }
        
        animateQualityChange()
    }
}
```

## 네트워크 타입 표시
```swift
class ConnectionTypeIndicator: UIView {
    @IBOutlet weak var typeLabel: UILabel!
    @IBOutlet weak var speedLabel: UILabel!
    
    func update(with monitor: NWPathMonitor) {
        monitor.pathUpdateHandler = { [weak self] path in
            DispatchQueue.main.async {
                if path.usesInterfaceType(.wifi) {
                    self?.typeLabel.text = "WiFi"
                    self?.typeLabel.textColor = .systemGreen
                } else if path.usesInterfaceType(.cellular) {
                    self?.typeLabel.text = self?.getCellularType()
                    self?.typeLabel.textColor = .systemOrange
                } else {
                    self?.typeLabel.text = "Unknown"
                    self?.typeLabel.textColor = .systemGray
                }
                
                // 대역폭 표시
                self?.speedLabel.text = "\(path.estimatedBandwidth / 1000) Kbps"
            }
        }
    }
    
    private func getCellularType() -> String {
        let info = CTTelephonyNetworkInfo()
        guard let type = info.serviceCurrentRadioAccessTechnology?.values.first else {
            return "Cellular"
        }
        
        switch type {
        case CTRadioAccessTechnologyLTE: return "LTE"
        case CTRadioAccessTechnologyNRNSA,
             CTRadioAccessTechnologyNR: return "5G"
        default: return "3G"
        }
    }
}
```

## 애니메이션 효과
```swift
extension NetworkQualityView {
    func animateQualityChange() {
        UIView.animate(withDuration: 0.3) {
            self.signalBars.enumerated().forEach { index, bar in
                if index < self.currentQuality.rawValue {
                    bar.alpha = 1.0
                    bar.backgroundColor = self.colorForQuality()
                } else {
                    bar.alpha = 0.3
                    bar.backgroundColor = .systemGray
                }
            }
        }
        
        // 품질 저하 시 경고
        if currentQuality == .poor || currentQuality == .bad {
            showQualityWarning()
        }
    }
}
```

## 연관 개념
- [[mos_score]]
- [[network_handover]]
- [[bandwidth_adaptation]]
- [[haptic_feedback]]

## 태그
#ui #network #quality #indicator #ux