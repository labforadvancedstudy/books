# SIP Protocol

## Core Insight
SIP is the telephone network's lingua franca translated to the internet - it establishes, modifies, and tears down calls using text-based messages similar to HTTP.

Session Initiation Protocol predates WebRTC and remains the backbone of many VoIP systems, especially enterprise and carrier networks. While WebRTC handles browser-to-browser calls, SIP connects to traditional phone systems, PBXs, and carrier networks. Understanding SIP is essential for building VoIP apps that interoperate with existing infrastructure.

SIP is beautifully simple in concept - it's like HTTP for calls. INVITE starts a call, ACK confirms, BYE ends it. Messages are human-readable text: "INVITE sip:alice@example.com" with headers describing the call. The body typically contains SDP describing media capabilities. This simplicity made SIP successful but also complex as extensions accumulated over decades.

The protocol separates signaling from media. SIP messages flow through proxy servers that route calls, handle authentication, and implement features like call forwarding. But once the call is established, media (RTP packets) flows directly between endpoints or through media servers. This separation allows flexible deployment architectures.

Registration is how clients become reachable. Your app sends REGISTER to a SIP server with your current IP address. When someone calls your SIP URI, the server knows where to forward the INVITE. This is similar to WebRTC's signaling but standardized across vendors.

For iOS apps, SIP integration typically uses libraries like PJSIP or LinPhone. These handle the complex protocol details, NAT traversal (using ICE/STUN/TURN like WebRTC), and media processing. But SIP's age shows - it predates NATs, assumes stable IPs, and lacks modern encryption by default. Most deployments now use SIP over TLS with SRTP for media.

## Connections
→ [[sdp_format]] - Carried in SIP messages
→ [[rtp_protocol]] - Media transport
→ [[sip_registration]] - Becoming reachable
→ [[pbx_integration]] - Enterprise connectivity
← [[webrtc_protocol]] - Modern alternative
← [[carrier_integration]] - PSTN connectivity

---
Level: L2
Date: 2025-08-15
Tags: #sip #protocol #voip #telephony