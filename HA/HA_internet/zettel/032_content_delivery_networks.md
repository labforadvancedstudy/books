# Content Delivery Networks

## Core Insight
CDNs are the internet's solution to physics - bringing content closer to users because light speed, while fast, isn't instant.

Geography still matters online. A server in Tokyo takes 100+ milliseconds to reach New York. Doesn't sound like much until you're loading hundreds of resources. Distance equals latency equals frustration. CDNs solve this by cloning content globally.

Akamai pioneered it: put servers everywhere, copy popular content to all of them, serve from the nearest. Now your New York request gets answered from New York, not Tokyo. The internet appears faster by becoming more redundant.

But CDNs became more than caches. They're shields against attacks (absorbing DDoS), optimizers (compressing images), computers (running code at edge). Cloudflare Workers lets you run programs in 200+ cities simultaneously. The edge is becoming intelligent.

The centralization paradox: CDNs decentralize content while centralizing control. A few companies (Cloudflare, Akamai, Fastly) serve most of the internet. When Fastly had an outage, half the web disappeared. Distribution created new single points of failure.

CDNs reveal a truth: the global internet is an illusion. We browse local copies of global content. The internet you experience depends on where you stand.

## Connections
→ [[029_caching]]
→ [[023_bandwidth]]
→ [[045_data_centers]]
← [[007_streaming_videos]]
← [[038_edge_computing]]

---
Level: L3
Date: 2025-06-23
Tags: #cdn #performance #geography #infrastructure