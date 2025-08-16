# VoIP Architecture

## Core Insight
VoIP architecture is a careful balance between centralization for control and decentralization for scalability - signaling flows through servers while media flows peer-to-peer when possible.

The fundamental architectural decision in VoIP is media path: peer-to-peer or server-relayed? P2P minimizes latency and server costs but complicates NAT traversal and recording. Server relay simplifies architecture and enables features (recording, transcription, group calls) but increases costs and latency. Most systems use hybrid approaches.

The classic three-tier architecture separates concerns. Signaling servers handle control plane: authentication, presence, call routing. Media servers (when needed) handle data plane: mixing for group calls, recording, transcoding. STUN/TURN infrastructure handles connectivity. This separation allows independent scaling of each tier.

Client architecture follows a layered model. The network layer handles sockets and protocols. The media engine processes audio/video. The signaling layer manages call state. The UI layer presents to users. Clean interfaces between layers enable swapping implementations - replacing WebRTC with proprietary protocols, for example.

State synchronization is architecturally critical. In distributed systems, clients and servers have different views of call state. Is the call ringing or connected? Did both parties hang up? Eventual consistency is acceptable for presence, but call state needs stronger guarantees. Many architectures use event sourcing or CRDTs for consistency.

Scalability patterns depend on use case. Consumer apps (WhatsApp, Telegram) optimize for millions of concurrent 1-on-1 calls - mostly P2P with minimal server involvement. Enterprise systems (Zoom, Teams) optimize for large group calls - using cascading media servers or selective forwarding units (SFUs). The architecture must match the primary use case.

Mobile constraints shape everything. Battery life demands efficient protocols. Unreliable networks require aggressive reconnection. Background restrictions need push notifications. The architecture that works for desktop browsers might fail completely on iOS.

## Connections
→ [[signaling_server]] - Control plane
→ [[media_server]] - Data plane
→ [[client_architecture]] - App structure
→ [[scalability_patterns]] - Growth strategies
← [[system_design]] - Overall approach
← [[deployment_architecture]] - Infrastructure layout

---
Level: L9
Date: 2025-08-15
Tags: #architecture #system-design #scalability