# Routing

## Core Insight
Routing is the internet's wayfinding system - packets navigating a constantly changing maze of billions of paths to reach their destination.

Every packet is a lost tourist with only an address. No map. No GPS. Just the kindness of routers pointing the way. "Looking for 192.168.1.1? Try that direction." Hop by hop, router by router, each packet finds its way through the digital wilderness.

The magic isn't that routing works perfectly - it's that it works at all. Millions of routers, owned by different companies, configured by different people, somehow agree on which way to send your data. It's coordinated anarchy, consensus without central control.

Routing tables are living documents, constantly updated as the network topology shifts. A cable cut in Egypt reroutes traffic through India. A new connection in Frankfurt creates shortcuts for European data. The internet's map redraws itself every second.

BGP (Border Gateway Protocol) is how the big networks talk. "I can reach Google in 3 hops." "Well I can do it in 2." Networks announce their routes like merchants hawking wares. The best path wins, until a better one appears.

But routing isn't neutral. Some paths cost more. Some countries block certain routes. Some ISPs prioritize their own traffic. The shortest path isn't always the chosen path. Politics, economics, and physics all shape where packets flow.

## Connections
→ [[029_caching]]
→ [[021_ip_addresses]]
→ [[025_tcp_ip_stack]]
← [[024_http_protocol]]
← [[022_dns_system]]

---
Level: L2
Date: 2025-06-23
Tags: #infrastructure #networking #protocols #routing