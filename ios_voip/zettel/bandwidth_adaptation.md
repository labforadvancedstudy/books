# Bandwidth Adaptation

## Core Insight
Bandwidth adaptation is continuous optimization - constantly measuring network capacity and adjusting quality to use every available bit without causing congestion.

Networks are dynamic. Available bandwidth fluctuates as users move, networks congest, and conditions change. Fixed-rate streaming causes problems: too high and packets drop, too low and quality suffers unnecessarily. Adaptive bitrate (ABR) continuously adjusts to match network capacity.

The feedback loop is key. Send media → monitor results → adjust accordingly. RTCP provides the feedback: packet loss rates, round-trip times, jitter statistics. High packet loss means you're sending too fast. Increasing RTT suggests queuing (congestion). The algorithm interprets these signals and adjusts the encoder bitrate.

WebRTC's congestion control is sophisticated. Google Congestion Control (GCC) uses a Kalman filter to estimate available bandwidth. It probe for more bandwidth by gradually increasing send rate, backs off when detecting congestion. Transport-Wide Congestion Control (TWCC) provides detailed feedback about every packet, enabling precise control.

Opus makes adaptation seamless. You can change bitrate every frame (20ms) from 6 to 510 kbps. Reduce bitrate when bandwidth drops, increase when it recovers. The codec handles the transition smoothly - no artifacts, no reinitialization. The receiver doesn't even need to know the bitrate changed.

But adaptation isn't just bitrate. You can adjust frame size (larger frames = more efficiency, higher latency), enable/disable FEC (forward error correction), change packet redundancy, even switch codecs. Video is more dramatic - resolution, framerate, and codec complexity all adapt.

The challenge is balancing reaction speed with stability. React too slowly and quality suffers during congestion. React too quickly and quality bounces unnecessarily. Most algorithms use some hysteresis - quick to reduce bitrate, slower to increase.

## Connections
→ [[rtcp_feedback]] - Provides network metrics
→ [[opus_codec]] - Enables smooth adaptation
→ [[congestion_control]] - Detects capacity
→ [[qos_metrics]] - Measures adaptation success
← [[network_monitoring]] - Triggers adaptation
← [[call_quality]] - Directly impacts experience

---
Level: L7
Date: 2025-08-15
Tags: #bandwidth #adaptation #congestion #quality