# TURN Server

## Core Insight
TURN servers are relay stations that forward traffic between peers who can't connect directly - they're expensive but essential for ensuring 100% connectivity.

TURN (Traversal Using Relays around NAT) is the nuclear option of NAT traversal. When direct connection fails, when STUN fails, when symmetric NATs or strict firewalls block everything, TURN saves the day by relaying all traffic through a server both peers can reach. It always works, but at a cost.

Unlike STUN which just tells you your public address, TURN actively relays every packet. When you allocate a TURN relay, the server gives you a public address on itself. Your peer sends media to that address, TURN forwards it to you, and vice versa. It's basically a VPN for your media stream, adding latency and consuming server bandwidth.

The cost is significant. A single audio call might use 50kbps each direction. With TURN, your server handles 100kbps per call. A thousand concurrent calls means 100Mbps of server bandwidth, 24/7. Plus CPU for packet processing. This is why commercial TURN services charge per GB or per minute.

Authentication is critical. TURN uses temporary credentials (typically HMAC-based) that your signaling server generates. Without auth, your TURN server becomes an open relay that anyone could abuse. The credentials usually expire after a few minutes, just long enough to establish a call.

Despite the costs, you need TURN for reliability. Studies show 8-15% of calls require TURN relay, depending on the network environment. Mobile networks and enterprise firewalls are the worst offenders. Without TURN, these users simply can't make calls.

## Connections
→ [[turn_authentication]] - Credential generation
→ [[relay_allocation]] - Getting a TURN address
← [[ice_protocol]] - Uses TURN as last resort
← [[stun_server]] - Tried before falling back to TURN
← [[bandwidth_costs]] - Major expense in VoIP infrastructure

---
Level: L3
Date: 2025-08-15
Tags: #turn #relay #nat #infrastructure