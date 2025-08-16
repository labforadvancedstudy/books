# Jitter Buffer

## Core Insight
A jitter buffer is a time machine that reorders packets and smooths network chaos into steady audio playback - trading latency for reliability.

Networks deliver packets irregularly. One packet takes 20ms, the next takes 60ms, then 15ms. This jitter makes direct playback impossible - you'd hear choppy, robotic audio. The jitter buffer absorbs this variation by delaying playback, accumulating packets, then playing them at a steady rate.

The buffer is essentially a priority queue ordered by RTP timestamp. Packets arrive out of order? The buffer sorts them. Packet arrives late? If within the buffer window, it's inserted at the right position. Packet never arrives? After waiting (typically 2-3 packet times), the buffer declares it lost and triggers concealment.

The size is a critical tradeoff. Larger buffers handle more jitter but add latency. Every 20ms of buffer is 20ms of mouth-to-ear delay. Beyond 150ms total latency, conversation becomes difficult. Most jitter buffers start at 40-60ms and adapt based on network conditions.

Adaptive jitter buffers monitor packet arrival patterns. If jitter increases (variance in inter-arrival times), the buffer grows. If the network stabilizes, it shrinks. The adaptation must be smooth - sudden size changes cause audio artifacts. Some implementations use machine learning to predict optimal buffer size.

WebRTC's NetEQ (Network Equalizer) is a sophisticated jitter buffer that does more than reorder. It stretches or compresses audio to adjust playout speed, conceals lost packets using predictive algorithms, and maintains constant delay despite network variations. It's the unsung hero making VoIP calls sound smooth despite terrible networks.

## Connections
→ [[packet_loss_concealment]] - Handles missing packets
→ [[network_statistics]] - Monitors jitter metrics
→ [[audio_playout]] - Consumes buffered packets
← [[opus_codec]] - Provides packets to buffer
← [[rtp_protocol]] - Carries timestamps for ordering
← [[network_handover]] - Buffer maintains continuity

---
Level: L4
Date: 2025-08-15
Tags: #jitter #buffer #networking #audio