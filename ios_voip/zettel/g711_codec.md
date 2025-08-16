# G.711 Codec

## Core Insight
G.711 is the digital equivalent of analog telephony - it's the 1972 standard that digitized the world's phone system and remains universally supported.

G.711 is ancient by tech standards but refuses to die because of one killer feature: everything supports it. Every VoIP system, every phone network, every PBX speaks G.711. When modern codecs fail to negotiate, G.711 is the fallback. It's the English of VoIP codecs - not the best, but everyone understands it.

The codec is beautifully simple. It samples audio at 8kHz (capturing frequencies up to 4kHz - basic speech), quantizes each sample to 8 bits using logarithmic compression (µ-law in North America, A-law in Europe). That's 64 kbps - exactly the bandwidth of a traditional phone line. No complex psychoacoustic models, no frame dependencies, no lookahead.

This simplicity brings advantages. Nearly zero latency - each sample is independent. Trivial CPU usage - modern processors handle G.711 without breaking a sweat. Perfect for legacy integration - it passes through traditional phone networks unchanged. Packet loss resilience - losing a packet only affects 20ms of audio, no error propagation.

But the quality ceiling is low. 8kHz sampling means no high frequencies - 's' sounds are muffled, music is terrible. 64 kbps is wasteful for the quality delivered - Opus achieves better quality at 16 kbps. There's no built-in packet loss concealment or echo cancellation. It's a Model T in the age of Teslas.

For iOS VoIP apps, G.711 matters for PSTN connectivity. When calling traditional phone numbers, carriers often require G.711. Your beautiful Opus audio gets transcoded to G.711 at the gateway. Supporting G.711 directly avoids transcoding quality loss and latency.

## Connections
→ [[pcm_audio]] - Underlying format
→ [[pstn_gateway]] - Primary use case
→ [[codec_negotiation]] - Universal fallback
← [[opus_codec]] - Modern replacement
← [[sip_protocol]] - Commonly uses G.711
← [[legacy_integration]] - Compatibility requirement

---
Level: L4
Date: 2025-08-15
Tags: #g711 #codec #legacy #telephony