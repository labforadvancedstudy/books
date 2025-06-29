# Load Balancing

## Core Insight
Load balancing is digital crowd control - distributing incoming requests across multiple servers so no single machine drowns in traffic.

Imagine a thousand people trying to enter through one door. Chaos. Now imagine fifty doors with someone directing people to the shortest line. That's load balancing. Simple concept, complex execution, essential for any service that wants to stay alive under pressure.

The load balancer is the bouncer of the internet. It looks at incoming requests and decides: you go to Server A, you to Server B, you to Server C. But unlike a bouncer, it makes these decisions thousands of times per second, adjusting for server health, response times, and capacity.

Round-robin is the simplest - everyone takes turns. But real load balancing is smarter. It considers server CPU, memory, active connections, geographic location, even the type of request. Video streaming goes to the beefy servers. API calls to the quick ones.

When Black Friday hits or a site goes viral, load balancers are the heroes nobody sees. They shuffle traffic like card dealers, keeping the game running even when the stakes explode. One server dies? Traffic flows around it like water around a stone.

## Connections
→ [[036_distributed_systems]]
→ [[037_cloud_computing]]
→ [[040_isps]]
← [[030_client_server]]
← [[045_data_centers]]

---
Level: L3
Date: 2025-06-23
Tags: #infrastructure #scalability #reliability #performance