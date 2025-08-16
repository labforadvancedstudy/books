# Reconnection Strategy

## Core Insight
Reconnection strategy is graceful degradation and recovery - assuming everything will fail and having a plan to recover without users noticing.

Network connections fail constantly on mobile. WiFi drops, cellular towers handoff, tunnels block signal, apps get suspended. A naive VoIP app drops calls at every hiccup. A robust app transparently recovers, maintaining call continuity across failures. The difference is reconnection strategy.

Exponential backoff prevents thundering herds. When connection fails, don't immediately retry - that might worsen congestion. Wait 1 second, then 2, 4, 8, up to some maximum (typically 30-60 seconds). Add jitter (random variation) to prevent synchronized retries from multiple clients. This simple algorithm prevents most cascade failures.

State preservation enables seamless recovery. Before disconnection is detected, save call state: who's calling whom, call duration, mute status. When reconnecting, restore this state. The call continues from where it left off, not from scratch. Users might notice a brief freeze but not a dropped call.

ICE restart is WebRTC's reconnection mechanism. Keep the same call (same DTLS keys, same SRTP context) but gather new network candidates. Exchange updated SDPs through your signaling channel. Media resumes on the new path. This is faster than establishing a new call and preserves security context.

Parallel paths improve reliability. Maintain connections on both WiFi and cellular simultaneously. When one fails, instantly switch to the other. This uses more battery and bandwidth but eliminates reconnection delay. Some apps do this only for "important" users or poor network conditions.

User feedback manages expectations. Show "Reconnecting..." immediately when detecting issues. Update with progress: "Trying alternate network..." Users tolerate delays better when they understand what's happening. But don't over-communicate - constant network status updates are annoying.

The key insight: plan for failure from the start. Every network operation needs timeout and retry logic. Every state change needs to be recoverable. Every user action needs to handle the "currently reconnecting" case. Resilience isn't added later - it's designed in.

## Connections
→ [[exponential_backoff]] - Retry algorithm
→ [[ice_restart]] - WebRTC mechanism
→ [[state_preservation]] - Maintaining context
→ [[network_monitoring]] - Failure detection
← [[network_handover]] - Trigger for reconnection
← [[call_reliability]] - User expectation

---
Level: L9
Date: 2025-08-15
Tags: #reconnection #reliability #resilience #strategy