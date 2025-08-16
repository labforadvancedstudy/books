# Opus Codec

## Core Insight
Opus is the Swiss Army knife of audio codecs - it seamlessly morphs between speech and music modes, adapting to any bitrate from 6 to 510 kbps.

Opus revolutionized VoIP audio by combining the best of SILK (Skype's speech codec) and CELT (low-latency music codec) into one adaptive codec. It switches between modes in real-time: pure speech at low bitrates, hybrid for voice with background music, pure CELT for music. This adaptability means one codec handles everything from narrow-band speech to full-band stereo music.

The codec's genius is its scalability. At 6-10 kbps, it rivals the best narrow-band speech codecs. At 16-24 kbps (typical for VoIP), it delivers wideband speech that sounds natural. At 64-128 kbps, it matches AAC for music. The bitrate can change every packet (20ms) based on network conditions.

Opus includes built-in resilience features. Forward Error Correction (FEC) embeds a low-quality copy of the previous packet in the current one. If a packet is lost, the decoder can use the FEC data to conceal the loss. Packet Loss Concealment (PLC) synthesizes missing packets from surrounding audio. In-band FEC adds 20-30% overhead but dramatically improves quality on lossy networks.

The codec operates on 20ms frames by default but supports 2.5 to 60ms. Shorter frames mean lower latency but more overhead. VoIP typically uses 20ms for the sweet spot of latency versus efficiency. The algorithmic delay is just 26.5ms total - critical for natural conversation.

For iOS, Opus is included in WebRTC or available as a standalone library. Hardware acceleration isn't necessary - even old iPhones can encode/decode Opus with minimal CPU usage.

## Connections
→ [[audio_quality]] - Determines user experience
→ [[bandwidth_adaptation]] - Changes bitrate dynamically
→ [[packet_loss_concealment]] - Built-in FEC/PLC
← [[webrtc_protocol]] - Default audio codec
← [[g711_codec]] - Opus replaces legacy codecs
← [[echo_cancellation]] - Processes Opus output

---
Level: L4
Date: 2025-08-15
Tags: #opus #codec #audio #compression