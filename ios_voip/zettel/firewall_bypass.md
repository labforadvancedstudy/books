# 방화벽 우회 기법

## 개념
엄격한 방화벽 환경에서 VoIP 통화를 가능하게 하는 다양한 기법. 포트 443 터널링, WebSocket, TURN 등 활용.

## HTTPS 터널링 (Port 443)
```swift
class HTTPSTunneling {
    private var websocket: URLSessionWebSocketTask?
    
    func establishTunnel(to server: String) {
        // HTTPS(443) 포트로 WebSocket 연결
        let url = URL(string: "wss://\(server):443/voip")!
        let session = URLSession(configuration: .default)
        websocket = session.webSocketTask(with: url)
        
        // 통화 데이터를 WebSocket으로 전송
        websocket?.resume()
        sendVoIPData()
    }
    
    private func sendVoIPData() {
        // RTP 패킷을 WebSocket 메시지로 캡슐화
        let rtpData = getRTPPacket()
        let message = URLSessionWebSocketTask.Message.data(rtpData)
        
        websocket?.send(message) { error in
            if error == nil {
                self.receiveVoIPData()
            }
        }
    }
}
```

## TURN-TCP 사용
```swift
class TURNTCPStrategy {
    func configureTURNForFirewall() -> RTCIceServer {
        // TCP 기반 TURN 서버 사용
        return RTCIceServer(
            urlStrings: [
                "turn:turn.example.com:443?transport=tcp",
                "turns:turn.example.com:443?transport=tcp"  // TLS
            ],
            username: "user",
            credential: "pass"
        )
    }
    
    func createPeerConnection() -> RTCPeerConnection {
        let config = RTCConfiguration()
        config.iceServers = [configureTURNForFirewall()]
        config.iceTransportPolicy = .relay  // TURN 전용 모드
        config.tcpCandidatePolicy = .enabled
        config.candidateNetworkPolicy = .all
        
        return factory.peerConnection(
            with: config,
            constraints: RTCMediaConstraints(),
            delegate: self
        )
    }
}
```

## DNS over HTTPS (DoH)
```swift
class DNSoverHTTPS {
    func resolveSIPServer(domain: String) async throws -> String {
        // Cloudflare DoH 사용
        let dohURL = "https://cloudflare-dns.com/dns-query"
        let query = createDNSQuery(for: domain, type: "A")
        
        var request = URLRequest(url: URL(string: dohURL)!)
        request.httpMethod = "POST"
        request.setValue("application/dns-message", forHTTPHeaderField: "Content-Type")
        request.httpBody = query
        
        let (data, _) = try await URLSession.shared.data(for: request)
        return parseDNSResponse(data)
    }
}
```

## 포트 다중화 전략
```swift
class PortMultiplexing {
    private let primaryPorts = [443, 8443, 8080]
    private let fallbackPorts = [80, 3478, 5349]
    
    func findAvailablePort() async -> (String, Int)? {
        // 우선 포트 시도
        for port in primaryPorts {
            if await testConnection(port: port) {
                return ("primary", port)
            }
        }
        
        // 폴백 포트 시도
        for port in fallbackPorts {
            if await testConnection(port: port) {
                return ("fallback", port)
            }
        }
        
        return nil
    }
    
    private func testConnection(port: Int) async -> Bool {
        // TCP 연결 테스트
        let host = NWEndpoint.Host("sip.example.com")
        let port = NWEndpoint.Port(rawValue: UInt16(port))!
        let connection = NWConnection(
            host: host,
            port: port,
            using: .tcp
        )
        
        return await withCheckedContinuation { continuation in
            connection.stateUpdateHandler = { state in
                switch state {
                case .ready:
                    continuation.resume(returning: true)
                case .failed:
                    continuation.resume(returning: false)
                default:
                    break
                }
            }
            connection.start(queue: .global())
        }
    }
}
```

## HTTP/2 멀티플렉싱
```swift
class HTTP2Multiplexing {
    private var session: URLSession!
    
    init() {
        let config = URLSessionConfiguration.default
        config.httpAdditionalHeaders = ["Content-Type": "application/octet-stream"]
        config.multipathServiceType = .handover
        config.allowsCellularAccess = true
        
        // HTTP/2 활성화
        config.httpShouldUsePipelining = true
        config.httpMaximumConnectionsPerHost = 1
        
        session = URLSession(configuration: config)
    }
    
    func sendVoIPStream(data: Data) async throws {
        // HTTP/2 POST로 오디오 스트림 전송
        let url = URL(string: "https://voip.example.com:443/stream")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.httpBody = data
        request.setValue("\(data.count)", forHTTPHeaderField: "Content-Length")
        
        let (_, response) = try await session.data(for: request)
        
        if let httpResponse = response as? HTTPURLResponse,
           httpResponse.statusCode != 200 {
            throw FirewallError.blocked
        }
    }
}
```

## 적응형 프로토콜 선택
```swift
class AdaptiveProtocol {
    enum ConnectionMethod {
        case directUDP
        case stunTurn
        case websocket
        case http2Tunnel
        case tcpFallback
    }
    
    func selectBestMethod() async -> ConnectionMethod {
        // 병렬로 모든 방법 테스트
        async let udpTest = testDirectUDP()
        async let turnTest = testTURN()
        async let wsTest = testWebSocket()
        async let httpTest = testHTTP2()
        
        let results = await [
            udpTest,
            turnTest,
            wsTest,
            httpTest
        ]
        
        // 가장 빠른 방법 선택
        if results[0] { return .directUDP }
        if results[1] { return .stunTurn }
        if results[2] { return .websocket }
        if results[3] { return .http2Tunnel }
        
        return .tcpFallback
    }
}
```

## 연관 개념
- [[proxy_traversal]]
- [[turn_server]]
- [[nat_traversal]]
- [[websocket_transport]]

## 태그
#firewall #bypass #tunnel #turn #websocket