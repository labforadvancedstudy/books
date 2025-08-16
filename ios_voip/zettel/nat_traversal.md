# NAT 타입 감지 및 통과

## 개념
NAT(Network Address Translation) 환경에서 P2P VoIP 연결을 위한 타입 감지와 통과 전략.

## NAT 타입 분류
```swift
enum NATType {
    case fullCone       // 가장 개방적
    case restrictedCone // IP 제한
    case portRestricted // 포트 제한
    case symmetric      // 가장 엄격
    case unknown
    
    var traversalDifficulty: Int {
        switch self {
        case .fullCone: return 1
        case .restrictedCone: return 2
        case .portRestricted: return 3
        case .symmetric: return 4  // TURN 필수
        case .unknown: return 5
        }
    }
}
```

## NAT 타입 감지
```swift
class NATDetector {
    private let stunServer1 = "stun1.example.com:3478"
    private let stunServer2 = "stun2.example.com:3478"
    
    func detectNATType() async -> NATType {
        // Test 1: STUN 서버에 연결 가능?
        guard let response1 = await sendSTUNRequest(to: stunServer1) else {
            return .unknown
        }
        
        // Test 2: 다른 IP에서 동일한 매핑?
        let response2 = await sendSTUNRequest(
            to: stunServer2,
            changeIP: true,
            changePort: false
        )
        
        if response2 != nil {
            // Test 3: 다른 포트에서 동일한 매핑?
            let response3 = await sendSTUNRequest(
                to: stunServer1,
                changeIP: false,
                changePort: true
            )
            
            if response3 != nil {
                return .fullCone
            } else {
                return .restrictedCone
            }
        } else {
            // Test 4: 동일한 매핑 주소?
            let response4 = await sendSTUNRequest(to: stunServer1)
            if response1.mappedAddress == response4?.mappedAddress {
                return .portRestricted
            } else {
                return .symmetric
            }
        }
    }
}
```

## STUN Binding Request
```swift
struct STUNMessage {
    let type: UInt16 = 0x0001  // Binding Request
    let length: UInt16 = 0
    let magicCookie: UInt32 = 0x2112A442
    let transactionID: Data = Data(count: 12)  // Random
    
    func encode() -> Data {
        var data = Data()
        data.append(contentsOf: withUnsafeBytes(of: type.bigEndian) { Array($0) })
        data.append(contentsOf: withUnsafeBytes(of: length.bigEndian) { Array($0) })
        data.append(contentsOf: withUnsafeBytes(of: magicCookie.bigEndian) { Array($0) })
        data.append(transactionID)
        return data
    }
}
```

## ICE Gathering
```swift
class ICEGatherer: NSObject, RTCPeerConnectionDelegate {
    private var candidates: [RTCIceCandidate] = []
    
    func peerConnection(_ peerConnection: RTCPeerConnection, 
                       didGenerate candidate: RTCIceCandidate?) {
        guard let candidate = candidate else {
            // Gathering complete
            analyzeConnectivity()
            return
        }
        
        candidates.append(candidate)
        
        // 후보 타입 분석
        if candidate.sdp.contains("typ host") {
            print("Local candidate: \(candidate.sdp)")
        } else if candidate.sdp.contains("typ srflx") {
            print("Server reflexive (STUN): \(candidate.sdp)")
        } else if candidate.sdp.contains("typ relay") {
            print("Relay (TURN): \(candidate.sdp)")
        }
    }
    
    private func analyzeConnectivity() {
        let hasRelay = candidates.contains { $0.sdp.contains("typ relay") }
        let hasSrflx = candidates.contains { $0.sdp.contains("typ srflx") }
        
        if !hasSrflx && !hasRelay {
            // Symmetric NAT 또는 방화벽
            print("WARNING: P2P connection unlikely, TURN required")
        }
    }
}
```

## 멀티홈 네트워크 처리
```swift
class MultihomeNetworkHandler {
    func gatherAllInterfaces() -> [NetworkInterface] {
        var interfaces: [NetworkInterface] = []
        var ifaddr: UnsafeMutablePointer<ifaddrs>?
        
        guard getifaddrs(&ifaddr) == 0 else { return [] }
        defer { freeifaddrs(ifaddr) }
        
        var ptr = ifaddr
        while ptr != nil {
            let interface = ptr!.pointee
            let name = String(cString: interface.ifa_name)
            
            // IPv4 및 IPv6 주소 수집
            if let address = interface.ifa_addr {
                let family = address.pointee.sa_family
                if family == AF_INET || family == AF_INET6 {
                    interfaces.append(
                        NetworkInterface(
                            name: name,
                            address: getIPAddress(from: address),
                            family: family == AF_INET ? .ipv4 : .ipv6
                        )
                    )
                }
            }
            
            ptr = interface.ifa_next
        }
        
        return interfaces
    }
}
```

## 연관 개념
- [[stun_server]]
- [[turn_server]]
- [[ice_protocol]]
- [[firewall_bypass]]

## 태그
#nat #traversal #stun #ice #p2p