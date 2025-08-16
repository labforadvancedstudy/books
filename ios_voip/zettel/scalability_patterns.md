# Scalability Patterns

## Core Insight
VoIP scalability is about choosing what to centralize - every architectural decision trades server costs against client complexity and call quality.

The full-mesh pattern is simplest but doesn't scale. In group calls, each participant sends media to every other participant. With N users, each sends N-1 streams. This works for 3-4 participants but fails beyond that - upload bandwidth becomes the bottleneck. Most home connections can't upload more than 3-4 audio streams simultaneously.

Selective Forwarding Units (SFU) are the modern standard. Each participant sends one stream to the SFU, which forwards it to others. This requires N upstream and N downstream connections per client - manageable even for large calls. The SFU doesn't decode media, just routes packets, keeping server CPU low. But bandwidth costs scale linearly with participants.

Multipoint Control Units (MCU) mix media server-side. Participants send to the MCU, which decodes all streams, mixes them into one, and sends a single stream back. This minimizes client bandwidth and CPU but requires powerful servers. Each server can handle fewer simultaneous calls. Legacy video conferencing uses this pattern.

Cascading topologies handle massive scale. Regional SFUs connect to super-nodes, which interconnect globally. Media flows through the hierarchy, adding latency but enabling thousands of participants. Zoom uses this for webinars - presenters go through different infrastructure than viewers.

Adaptive topologies switch patterns based on call size. Start with P2P for 1-on-1 calls (lowest latency). Switch to SFU when a third person joins. Migrate to MCU for large calls where clients can't handle multiple streams. This complexity is hidden from users but requires careful orchestration.

Geographic distribution is essential at scale. Place media servers near users to minimize latency. Use anycast or geo-DNS for automatic routing. But this requires presence in multiple regions, increasing operational complexity. The tradeoff: global infrastructure costs versus call quality.

## Connections
→ [[sfu_architecture]] - Routing pattern
→ [[mcu_architecture]] - Mixing pattern
→ [[geographic_distribution]] - Regional deployment
→ [[bandwidth_costs]] - Economic constraints
← [[voip_architecture]] - System design
← [[group_calling]] - Scaling challenge

---
Level: L9
Date: 2025-08-15
Tags: #scalability #architecture #patterns #infrastructure