# Battery Optimization

## Core Insight
Battery optimization in VoIP is death by a thousand cuts - every millisecond of CPU, every byte transmitted, every wake from sleep compounds into hours of battery life.

VoIP is inherently battery-intensive. The CPU encodes/decodes audio 50 times per second. The radio transmits constantly. The screen might stay on. Background processing prevents deep sleep. A poorly optimized VoIP app can drain a battery in 2-3 hours. A well-optimized one enables all-day calling.

CPU optimization starts with codec selection. Hardware-accelerated codecs save power, but Opus (software) often wins overall by using lower bitrates. The key is minimizing total CPU cycles: efficient DSP algorithms, SIMD instructions when available, and avoiding unnecessary processing. Process audio in larger chunks when possible - waking the CPU less frequently saves significant power.

Network optimization focuses on the radio. Cellular radios have multiple power states: active transmission (highest), tail time (medium), idle (lowest). Sending data in bursts, then allowing the radio to sleep, saves more power than continuous small transmissions. Bundle signaling messages, use larger packet intervals when quality permits.

The most effective optimization is doing nothing. Voice Activity Detection (VAD) stops sending during silence. Discontinuous Transmission (DTX) can reduce bandwidth by 40-50% in typical conversation. But be careful - aggressive VAD clips speech beginnings, making conversation feel unnatural.

Background optimization is crucial. Use iOS's background modes properly - don't keep the app active unnecessarily. Release resources when calls end. Defer non-essential work. Monitor thermal state and reduce quality if the device overheats.

## Connections
→ [[voice_activity_detection]] - Reduces transmission
→ [[codec_selection]] - Power vs quality tradeoff
→ [[background_modes]] - System-level optimization
→ [[cpu_optimization]] - Processing efficiency
← [[app_store_requirements]] - Battery life expectations
← [[user_experience]] - Battery anxiety affects usage

---
Level: L7
Date: 2025-08-15
Tags: #battery #optimization #performance #mobile