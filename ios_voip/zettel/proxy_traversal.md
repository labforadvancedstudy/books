# 프록시 서버 통과

## 개념
기업 환경에서 HTTP/HTTPS/SOCKS 프록시를 통해 VoIP 통화 연결. 방화벽 뒤에서도 통화 가능하게 함.

## HTTP CONNECT 터널링
```swift
class HTTPProxyTunnel {
    private var proxyHost: String
    private var proxyPort: Int
    private var targetHost: String
    private var targetPort: Int
    
    func establishTunnel(completion: @escaping (Bool) -> Void) {
        // HTTP CONNECT 메소드로 터널 생성
        let connectRequest = """
            CONNECT \(targetHost):\(targetPort) HTTP/1.1\r\n
            Host: \(targetHost):\(targetPort)\r\n
            Proxy-Connection: Keep-Alive\r\n
            \r\n
            """
        
        // 프록시 연결
        let session = URLSession(configuration: .default)
        let url = URL(string: "http://\(proxyHost):\(proxyPort)")!
        
        var request = URLRequest(url: url)
        request.httpMethod = "CONNECT"
        request.setValue("\(targetHost):\(targetPort)", forHTTPHeaderField: "Host")
        
        // 프록시 인증
        if let credentials = getProxyCredentials() {
            let auth = "\(credentials.username):\(credentials.password)"
                .data(using: .utf8)!
                .base64EncodedString()
            request.setValue("Basic \(auth)", forHTTPHeaderField: "Proxy-Authorization")
        }
        
        // 터널 연결 확인
        let task = session.dataTask(with: request) { data, response, error in
            if let httpResponse = response as? HTTPURLResponse,
               httpResponse.statusCode == 200 {
                completion(true)
            } else {
                completion(false)
            }
        }
        task.resume()
    }
}
```

## SOCKS5 프록시
```swift
class SOCKS5Proxy {
    private var socket: GCDAsyncSocket?
    
    func connectThroughSOCKS5(
        proxyHost: String,
        proxyPort: UInt16,
        targetHost: String,
        targetPort: UInt16
    ) {
        socket = GCDAsyncSocket(delegate: self, delegateQueue: .main)
        
        do {
            // 1. 프록시 연결
            try socket?.connect(toHost: proxyHost, onPort: proxyPort)
        } catch {
            print("Failed to connect to SOCKS5 proxy: \(error)")
        }
    }
    
    private func sendSOCKS5Handshake() {
        // SOCKS5 핸드셰이크
        var handshake: [UInt8] = [
            0x05,  // SOCKS version
            0x01,  // Number of auth methods
            0x00   // No authentication
        ]
        
        let data = Data(handshake)
        socket?.write(data, withTimeout: -1, tag: 0)
    }
    
    private func sendSOCKS5ConnectRequest(host: String, port: UInt16) {
        var request: [UInt8] = [
            0x05,  // SOCKS version
            0x01,  // CONNECT command
            0x00,  // Reserved
            0x03   // Domain name
        ]
        
        // 도메인 길이와 도메인
        request.append(UInt8(host.count))
        request.append(contentsOf: host.utf8)
        
        // 포트 (big endian)
        request.append(UInt8(port >> 8))
        request.append(UInt8(port & 0xFF))
        
        socket?.write(Data(request), withTimeout: -1, tag: 1)
    }
}
```

## WebRTC over Proxy
```swift
class WebRTCProxyConfiguration {
    static func configureForProxy(
        factory: RTCPeerConnectionFactory,
        proxySettings: ProxySettings
    ) -> RTCConfiguration {
        let config = RTCConfiguration()
        
        // TURN over TCP for proxy compatibility
        let turnServer = RTCIceServer(
            urlStrings: ["turn:\(proxySettings.turnServer):443?transport=tcp"],
            username: proxySettings.turnUsername,
            credential: proxySettings.turnPassword
        )
        
        config.iceServers = [turnServer]
        config.iceTransportPolicy = .relay  // TURN 전용
        config.bundlePolicy = .maxBundle
        config.rtcpMuxPolicy = .require
        config.tcpCandidatePolicy = .enabled  // TCP 활성화
        
        return config
    }
}
```

## 자동 프록시 감지
```swift
class ProxyDetector {
    static func getSystemProxySettings() -> ProxySettings? {
        guard let proxySettings = CFNetworkCopySystemProxySettings()?.takeRetainedValue() as? [String: Any] else {
            return nil
        }
        
        // HTTP 프록시 확인
        if let httpProxy = proxySettings[kCFNetworkProxiesHTTPProxy as String] as? String,
           let httpPort = proxySettings[kCFNetworkProxiesHTTPPort as String] as? Int {
            return ProxySettings(
                type: .http,
                host: httpProxy,
                port: httpPort
            )
        }
        
        // SOCKS 프록시 확인
        if let socksProxy = proxySettings[kCFNetworkProxiesSOCKSProxy as String] as? String,
           let socksPort = proxySettings[kCFNetworkProxiesSOCKSPort as String] as? Int {
            return ProxySettings(
                type: .socks5,
                host: socksProxy,
                port: socksPort
            )
        }
        
        return nil
    }
}
```

## 프록시 우회 전략
```swift
enum ProxyBypassStrategy {
    case directFirst     // 직접 연결 시도 후 프록시
    case proxyOnly      // 프록시만 사용
    case autoDetect     // 자동 감지
    
    func shouldUseProxy(for host: String) -> Bool {
        // 로컬 네트워크는 프록시 사용 안함
        if host == "localhost" || host.hasPrefix("192.168.") {
            return false
        }
        
        switch self {
        case .directFirst:
            // 직접 연결 실패 후 프록시 사용
            return !isDirectConnectionAvailable(host)
        case .proxyOnly:
            return true
        case .autoDetect:
            return ProxyDetector.getSystemProxySettings() != nil
        }
    }
}
```

## 연관 개념
- [[firewall_bypass]]
- [[turn_server]]
- [[ipv6_support]]

## 태그
#proxy #firewall #enterprise #tunnel #socks