# WebRTC Protocol

## Core Insight
WebRTC is a complete real-time communication stack that handles everything from codec negotiation to NAT traversal - it's the modern standard that replaced proprietary VoIP solutions.

WebRTC isn't just one protocol - it's an orchestrated collection of standards working together. At its heart, it solves the fundamental problem of establishing peer-to-peer connections between browsers or apps, even when both peers are behind NATs or firewalls. This is surprisingly hard because most devices don't have public IP addresses.

The connection dance starts with signaling - exchanging SDPs (Session Description Protocol) that describe what each peer can do. "I can send Opus audio at 48kHz, I can receive VP8 video, here are my network candidates." This happens outside WebRTC itself, typically through WebSockets to your signaling server.

ICE (Interactive Connectivity Establishment) then tries every possible way to connect: direct connection (rare), STUN-assisted NAT traversal (common), or TURN relay (last resort). It gathers candidates - possible network paths - and tests them in parallel. The first successful path wins. This adaptability is why WebRTC "just works" across different networks.

Once connected, RTP (Real-time Transport Protocol) carries the actual media, while RTCP provides feedback about quality. SRTP encrypts everything by default. DataChannels can carry arbitrary data alongside media. The stack automatically handles packet loss, jitter, and bandwidth changes.

For iOS, you typically use a native WebRTC library (Google's libwebrtc or a fork) wrapped in Swift/Objective-C. The library handles the complex low-level details while you manage the signaling and UI layer.

## Connections
→ [[ice_protocol]] - Connection establishment
→ [[sdp_format]] - Capability negotiation
→ [[stun_server]] - NAT traversal
→ [[turn_server]] - Relay fallback
→ [[opus_codec]] - Default audio codec
← [[signaling_server]] - Exchanges SDPs between peers
← [[webrtc_implementation]] - iOS-specific usage

---
Level: L2
Date: 2025-08-15
Tags: #webrtc #protocol #real-time #p2p