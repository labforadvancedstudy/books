# IPv6 지원

## 개념
Apple App Store 제출 필수 요구사항. VoIP 앱은 IPv6-only 네트워크에서도 완벽하게 작동해야 함.

## 네트워크 연결 처리
```swift
import Network

class IPv6NetworkManager {
    private let connection: NWConnection
    
    init(host: String, port: UInt16) {
        // NWEndpoint로 IPv4/IPv6 자동 처리
        let endpoint = NWEndpoint.hostPort(
            host: NWEndpoint.Host(host),
            port: NWEndpoint.Port(rawValue: port)!
        )
        
        // IPv6 우선 설정
        let parameters = NWParameters.udp
        parameters.preferNoProxies = false
        parameters.includePeerToPeer = false
        
        // IP 버전 제약 설정
        let ipOptions = NWProtocolIP.Options()
        ipOptions.version = .v6  // IPv6 우선
        parameters.defaultProtocolStack.internetProtocol = ipOptions
        
        connection = NWConnection(to: endpoint, using: parameters)
    }
}
```

## SIP URI 처리
```swift
struct SIPAddress {
    let user: String
    let host: String
    let port: UInt16?
    
    // IPv6 주소 형식 처리
    var uri: String {
        if host.contains(":") {
            // IPv6 주소는 대괄호로 감싸기
            if let port = port {
                return "sip:\(user)@[\(host)]:\(port)"
            } else {
                return "sip:\(user)@[\(host)]"
            }
        } else {
            // IPv4 또는 도메인
            if let port = port {
                return "sip:\(user)@\(host):\(port)"
            } else {
                return "sip:\(user)@\(host)"
            }
        }
    }
}
```

## WebRTC ICE 후보 처리
```swift
class ICECandidateHandler {
    func processCandidate(_ candidate: RTCIceCandidate) {
        let sdp = candidate.sdp
        
        // IPv6 후보 파싱
        if sdp.contains("typ host") {
            // IPv6 주소 형식: xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx
            let ipv6Pattern = "([0-9a-fA-F]{0,4}:){7}[0-9a-fA-F]{0,4}"
            let ipv4Pattern = "([0-9]{1,3}\\.){3}[0-9]{1,3}"
            
            if sdp.range(of: ipv6Pattern, options: .regularExpression) != nil {
                print("IPv6 candidate detected")
                // IPv6 후보 우선 처리
            }
        }
    }
}
```

## DNS64/NAT64 호환성
```swift
class DNS64Support {
    // Apple의 getaddrinfo 사용 권장
    func resolveHost(_ hostname: String) -> [String] {
        var addresses: [String] = []
        
        var hints = addrinfo()
        hints.ai_family = AF_UNSPEC  // IPv4와 IPv6 모두 허용
        hints.ai_socktype = SOCK_STREAM
        
        var result: UnsafeMutablePointer<addrinfo>?
        
        if getaddrinfo(hostname, nil, &hints, &result) == 0 {
            var current = result
            while current != nil {
                if let addr = current?.pointee.ai_addr {
                    if current?.pointee.ai_family == AF_INET6 {
                        // IPv6 주소 처리
                        var hostname = [CChar](repeating: 0, count: Int(NI_MAXHOST))
                        getnameinfo(addr, socklen_t(current!.pointee.ai_addrlen),
                                  &hostname, socklen_t(hostname.count),
                                  nil, 0, NI_NUMERICHOST)
                        addresses.append(String(cString: hostname))
                    }
                }
                current = current?.pointee.ai_next
            }
            freeaddrinfo(result)
        }
        
        return addresses
    }
}
```

## 테스트 방법
```swift
class IPv6Testing {
    static func isIPv6Available() -> Bool {
        var zeroAddress = sockaddr_in6()
        zeroAddress.sin6_len = UInt8(MemoryLayout<sockaddr_in6>.size)
        zeroAddress.sin6_family = sa_family_t(AF_INET6)
        
        let reachability = withUnsafePointer(to: &zeroAddress) {
            $0.withMemoryRebound(to: sockaddr.self, capacity: 1) { zeroSockAddress in
                SCNetworkReachabilityCreateWithAddress(nil, zeroSockAddress)
            }
        }
        
        var flags: SCNetworkReachabilityFlags = []
        if let reachability = reachability,
           SCNetworkReachabilityGetFlags(reachability, &flags) {
            return flags.contains(.reachable)
        }
        
        return false
    }
}
```

## 연관 개념
- [[stun_server]]
- [[turn_server]]
- [[nat_traversal]]
- [[proxy_traversal]]

## 태그
#ipv6 #networking #appstore #dns64 #nat64