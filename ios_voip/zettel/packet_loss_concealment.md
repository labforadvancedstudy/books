# Packet Loss Concealment

## Core Insight
Packet Loss Concealment is audio hallucination - synthesizing plausible sounds to fill gaps where packets vanished, making network problems inaudible.

Networks lose packets. WiFi interference, congestion, routing issues - packets simply don't arrive. In VoIP, each lost packet is 20ms of missing audio. Without concealment, you'd hear clicks, pops, or silence. Good PLC makes moderate loss (1-3%) nearly imperceptible.

The simplest concealment replays the last received packet. Lost packet? Play the previous one again. This works surprisingly well for steady sounds like vowels. But for dynamic speech, repetition creates robotic artifacts. More sophisticated algorithms are needed.

Modern PLC uses signal prediction. Analyze recent audio to understand pitch, energy, and spectral characteristics. When a packet is lost, extrapolate these features forward. Generate synthetic audio matching the expected continuation. It's like autocomplete for sound - predicting what probably came next.

Opus includes sophisticated PLC. The decoder maintains internal state about recent audio. When you call decode with a null packet, it generates synthetic audio based on this state. The synthesis uses pitch prediction, spectral extrapolation, and energy matching. For single packet losses, it's nearly perfect.

Forward Error Correction (FEC) prevents needing PLC. Opus can embed a low-quality copy of packet N-1 inside packet N. If packet N-1 is lost but packet N arrives, you recover the audio at reduced quality. This redundancy increases bandwidth by 20-30% but dramatically improves perceived quality on lossy networks.

The challenge is burst losses. Losing one packet is manageable. Losing five consecutive packets (100ms) is catastrophic. No algorithm can hallucinate that much missing audio convincingly. This is why jitter buffers and FEC are critical - preventing burst losses is better than concealing them.

## Connections
→ [[fec_redundancy]] - Prevents need for PLC
→ [[jitter_buffer]] - Detects packet loss
→ [[audio_synthesis]] - Generation techniques
← [[opus_codec]] - Includes PLC algorithm
← [[network_statistics]] - Triggers concealment
← [[call_quality]] - Major factor in perception

---
Level: L4
Date: 2025-08-15
Tags: #plc #packet-loss #audio #concealment