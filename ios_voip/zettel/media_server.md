# Media Server

## Core Insight
Media servers are optional middlemen in VoIP - they enable features impossible with pure P2P but add latency, cost, and complexity to your architecture.

Media servers sit in the media path, receiving streams from senders and forwarding to receivers. Unlike P2P where media flows directly, servers enable recording, transcription, mixing, and group calls beyond P2P limits. But each hop adds 5-50ms latency and requires bandwidth for every stream.

The Selective Forwarding Unit (SFU) is the lightest server type. It receives multiple streams and forwards them selectively. No decoding, no mixing - just intelligent packet routing. Client A sends video to the SFU, which forwards it to clients B, C, and D. Each client still receives multiple streams but only uploads one. This scales to hundreds of participants with minimal server CPU.

Multipoint Control Units (MCU) are heavyweight servers. They decode all incoming streams, mix them (combining audio, compositing video), re-encode, and send a single stream to each participant. Clients get perfectly mixed conference audio and composed video layouts. But the server CPU cost is enormous - decoding and encoding for every participant.

Recording requires media servers. P2P calls have no central point to capture media. Servers can record raw streams or mixed output. But recording raises legal questions (consent, storage, privacy) and technical challenges (synchronization, storage costs). Many apps record only on demand, not by default.

Transcoding enables compatibility. Client A sends VP9 video, but Client B only supports H.264. The media server transcodes between formats. Similarly for audio - converting between Opus and G.711 for PSTN connectivity. But transcoding is CPU-intensive and adds latency.

Geographic distribution of media servers minimizes latency. Place servers near user populations. Use the closest server for regional calls. For international calls, either route through multiple servers (adding latency) or pick a compromise location. This requires infrastructure in multiple regions and intelligent routing.

## Connections
→ [[sfu_architecture]] - Forwarding pattern
→ [[mcu_architecture]] - Mixing pattern
→ [[transcoding]] - Format conversion
→ [[recording_architecture]] - Capture mechanism
← [[scalability_patterns]] - When servers help
← [[group_calling]] - Primary use case

---
Level: L3
Date: 2025-08-15
Tags: #media-server #infrastructure #sfu #mcu