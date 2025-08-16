# Network Handover

## Core Insight
Network handover is the art of seamlessly switching between WiFi and cellular mid-call - the user should never notice the transition.

Network transitions happen constantly on mobile devices. Users walk out of WiFi range, enter elevators, drive through tunnels. Each transition means new IP addresses, different latencies, changed packet routes. Without proper handling, calls drop or freeze during these transitions.

The problem is that UDP sockets (used by RTP for media) are bound to specific interfaces. When WiFi disconnects, those sockets become useless. You need new sockets on the cellular interface, but the remote peer is still sending to your old WiFi address. This is where ICE restart comes in.

ICE restart is WebRTC's solution. When the network changes, you gather new ICE candidates on the new interface, exchange updated SDPs through your signaling channel, and establish a new media path. The old path might still work briefly (cellular was already active alongside WiFi), providing time for smooth transition.

iOS's Network Extension framework notifies you of network changes through `NWPathMonitor`. When the path changes, you trigger an ICE restart. But timing matters - trigger too aggressively, and you cause unnecessary disruptions. Wait too long, and the call freezes. Most apps wait 1-2 seconds for the network to stabilize.

The handover should be transparent to users. Continue playing received audio from the jitter buffer during transition. Show a brief "reconnecting" indicator if needed. Pre-emptively gather candidates on both interfaces when possible. Some apps maintain dual paths (WiFi and cellular) simultaneously, switching instantly when one fails.

## Connections
→ [[ice_protocol]] - Restart mechanism
→ [[network_monitoring]] - Detecting changes
→ [[jitter_buffer]] - Smooths transition
→ [[connection_resilience]] - Recovery strategies
← [[webrtc_protocol]] - Handles path migration
← [[call_quality]] - Affected by transitions

---
Level: L5
Date: 2025-08-15
Tags: #networking #handover #mobility #ice