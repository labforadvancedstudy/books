# Caching

## Core Insight
Caching is the internet's memory trick - storing answers to avoid asking the same question twice, trading space for speed.

Your browser caches images so the second visit loads instantly. Your ISP caches popular videos so neighbors stream smoothly. CDNs cache content globally so distance doesn't matter. Every level of the internet remembers to forget less.

Cache is provisional truth. "Here's what I told you last time, still good?" Most times yes, sometimes no. The hard problem: knowing when cached truth became lies. Too aggressive, serve stale content. Too conservative, waste bandwidth. Cache invalidation is famously computer science's hardest problem.

But caching enables scale. Netflix works because thousands watch the same episode cached nearby, not streamed from California. Google works because common searches return cached results. The internet handles billions of users by being clever about repetition.

The philosophical twist: caching makes the internet non-deterministic. The same request might return different results based on what's cached where. The internet you see depends partly on what your neighbors watched recently. We each browse slightly different webs.

## Connections
→ [[023_bandwidth]]
→ [[032_content_delivery_networks]]
→ [[025_tcp_ip_stack]]
← [[015_web_pages]]
← [[022_dns_system]]

---
Level: L2
Date: 2025-06-23
Tags: #caching #performance #memory #optimization