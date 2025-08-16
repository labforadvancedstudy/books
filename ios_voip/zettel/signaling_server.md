# Signaling Server

## Core Insight
The signaling server is a matchmaker for peer-to-peer connections - it introduces peers and exchanges their connection details, then steps aside for direct communication.

Signaling is WebRTC's unsolved problem. The standard defines how peers connect and exchange media, but not how they find each other initially. Every WebRTC application needs custom signaling to exchange SDPs, ICE candidates, and control messages. This flexibility is both WebRTC's strength and its complexity.

The server typically uses WebSockets for real-time bidirectional communication. When Alice calls Bob, she sends her SDP offer through the signaling server. The server forwards it to Bob (if he's online), Bob sends his answer back through the server, and ICE candidates trickle through as they're discovered. Once connected peer-to-peer, the server's job is done.

But signaling servers do more than relay messages. They handle user presence (who's online), authentication (who can call whom), push notifications (wake Bob's app), and call routing (find Bob across multiple devices). They're the control plane for your VoIP system, managing everything except the actual media.

State management is crucial. The server tracks ongoing calls, handles disconnections, and ensures consistency. If Alice thinks she's calling Bob but Bob thinks the call ended, the server reconciles this. It handles edge cases: simultaneous calls, network failures, app crashes. Good signaling servers are state machines managing distributed system complexity.

For iOS, the signaling server also sends VoIP pushes. When Bob is offline, the server queues the call, sends a VoIP push to wake his app, waits for him to connect, then delivers the pending offer. This coordination between push notifications and WebSocket connections is critical for reliable call delivery.

## Connections
→ [[websocket_protocol]] - Common transport
→ [[sdp_exchange]] - Primary purpose
→ [[voip_push_delivery]] - Waking iOS apps
→ [[presence_management]] - User availability
← [[webrtc_protocol]] - Requires signaling
← [[ice_protocol]] - Candidates flow through server

---
Level: L3
Date: 2025-08-15
Tags: #signaling #server #infrastructure #websocket