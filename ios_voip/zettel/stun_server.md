# STUN Server

## Core Insight
STUN servers are mirrors that show devices their public face on the internet - without them, peers behind NATs would never know how others can reach them.

STUN (Session Traversal Utilities for NAT) solves a fundamental problem: when you're behind a NAT router, you only know your private IP (like 192.168.1.100), but peers on the internet need your public IP and port. The STUN server acts as an external observer that tells you, "From my perspective, you're at 203.0.113.45:54321."

The protocol is beautifully simple. Your client sends a binding request to the STUN server (typically on port 3478). The server responds with your public address as it sees it. This address:port combination becomes an ICE candidate that you share with peers. If both peers can reach each other's public addresses, you have a direct connection.

But STUN only works for certain NAT types. With "full cone" or "restricted cone" NATs, once a port is mapped, external peers can reach you there. But with "symmetric NATs" (common in corporate networks and cellular), each destination gets a different public port. STUN can't help here - you need TURN.

Google runs public STUN servers (stun.l.google.com:19302) that anyone can use. For production, you want geographically distributed STUN servers to minimize latency during the connection setup phase. The STUN query itself is just a few packets, but those milliseconds matter for call setup time.

## Connections
→ [[ice_protocol]] - Uses STUN to gather candidates
→ [[nat_traversal]] - Core problem STUN solves
← [[webrtc_protocol]] - Includes STUN in connection process
← [[turn_server]] - Fallback when STUN fails
← [[network_candidates]] - STUN provides server-reflexive candidates

---
Level: L3
Date: 2025-08-15
Tags: #stun #nat #networking #infrastructure