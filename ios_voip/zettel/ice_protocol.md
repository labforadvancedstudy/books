# ICE Protocol

## Core Insight
ICE is a systematic trial-and-error process that tests every possible network path between peers, racing to find the fastest connection that actually works.

Interactive Connectivity Establishment solves the hardest problem in P2P communication: how do two devices behind NATs find each other? The brilliant insight is to try everything in parallel. Gather all possible addresses (candidates), test all combinations, use the first working path. It's brute force, but it works.

Candidate gathering is the first phase. ICE collects host candidates (local IP addresses), server-reflexive candidates (public IP from STUN), and relay candidates (TURN allocations). A typical mobile device might have 10+ candidates: IPv4 and IPv6, WiFi and cellular, multiple interfaces. Each represents a potential path for media.

The connectivity check phase tests every candidate pair. If Alice has 5 candidates and Bob has 5, that's 25 possible paths. ICE sends STUN binding requests across all paths simultaneously. Successful responses prove connectivity. But not all paths are equal - direct connections are preferred over relayed ones.

Priority ordering ensures optimal path selection. Each candidate has a priority based on type (host > server-reflexive > relay) and network (WiFi > cellular). When multiple paths work, ICE picks the highest priority. This usually means direct connection when possible, falling back to TURN only when necessary.

ICE coordinates through the controlling/controlled role mechanism. One peer (controlling) decides the final path, while the other (controlled) follows. This prevents race conditions where peers pick different paths. The role is determined during offer/answer exchange.

Continuous monitoring keeps connections alive. ICE sends periodic STUN keepalives ensuring NAT mappings don't timeout. If the selected path fails, ICE can switch to another working path without restarting the call.

## Connections
→ [[stun_server]] - Provides server-reflexive candidates
→ [[turn_server]] - Provides relay candidates
→ [[network_candidates]] - What ICE gathers
→ [[connectivity_checks]] - How ICE tests paths
← [[webrtc_protocol]] - Uses ICE for connection
← [[network_handover]] - ICE restart for new paths

---
Level: L2
Date: 2025-08-15
Tags: #ice #nat #connectivity #protocol