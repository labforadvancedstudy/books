# Network Socket

## Core Insight
Network sockets are the OS's abstraction for network communication - file descriptors that represent communication endpoints, hiding the complexity of packets and protocols.

For VoIP, UDP sockets dominate. Unlike TCP's reliable, ordered delivery, UDP sends packets immediately without guarantees. This seems worse, but for real-time media, late packets are useless. Better to lose a packet than delay all subsequent packets waiting for retransmission. UDP's lack of congestion control also means consistent latency.

Socket binding determines which interface and port your app uses. Bind to 0.0.0.0 (INADDR_ANY) to listen on all interfaces, or specific IPs for particular networks. For mobile VoIP, binding becomes complex - WiFi and cellular have different interfaces. When networks change, old sockets become useless, requiring new ones on the new interface.

The socket buffer is crucial for VoIP performance. Too small, and packets drop when bursts arrive. Too large, and latency increases. The kernel maintains separate send and receive buffers. For VoIP, typical receive buffers are 256KB-1MB to handle jitter. Send buffers can be smaller since you control transmission rate.

Non-blocking mode is essential. Blocking sockets halt your thread waiting for data - death for real-time audio. Non-blocking sockets return immediately, letting you poll or use select/epoll/kqueue for efficient event handling. This enables processing multiple sockets (RTP, RTCP, signaling) from one thread.

Socket options tune behavior. SO_REUSEADDR allows quick restart after crashes. IP_TOS sets packet priority for QoS. SO_RCVTIMEO implements timeouts. For iOS, the most important is SO_NOSIGPIPE - without it, writing to a closed socket crashes your app with SIGPIPE.

File descriptor limits constrain scalability. iOS apps have soft limits on open sockets. Each call might use 4+ sockets (audio RTP, audio RTCP, video RTP, video RTCP). Group calls multiply this. Hit the limit and socket creation fails mysteriously.

## Connections
→ [[udp_protocol]] - Transport layer
→ [[socket_buffer]] - Performance tuning
→ [[network_interface]] - Binding target
← [[rtp_protocol]] - Uses UDP sockets
← [[network_handover]] - Requires new sockets
← [[ios_networking]] - Platform specifics

---
Level: L1
Date: 2025-08-15
Tags: #socket #networking #udp #ios