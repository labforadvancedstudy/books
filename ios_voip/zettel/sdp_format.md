# SDP Format

## Core Insight
SDP is a text-based contract describing everything about a media session - it's how endpoints agree on codecs, network addresses, and capabilities before establishing a connection.

Session Description Protocol looks deceptively simple - just key-value pairs describing media. But this simplicity hides enormous complexity. An SDP for a WebRTC call can be hundreds of lines, describing audio codecs, video codecs, network candidates, encryption keys, bandwidth limits, and dozens of extensions. It's the Swiss Army knife of media negotiation.

Each line follows a `<type>=<value>` format. `v=0` starts every SDP (version). `o=` identifies the session creator. `c=` specifies connection information. `m=` begins a media section. The order matters - SDP has strict grammar rules. One malformed line can break the entire negotiation.

The offer/answer model is how SDP enables negotiation. Alice sends an offer listing everything she can do: "I can send Opus at 48kHz, G.711 at 8kHz, use SRTP with these cipher suites." Bob's answer selects from Alice's options: "Let's use Opus at 48kHz with AES_CM_128_HMAC_SHA1_80." This negotiation ensures compatibility.

Modern SDP is full of extensions. ICE candidates appear as `a=candidate` lines. DTLS fingerprints for encryption as `a=fingerprint`. Simulcast instructions, bandwidth adaption, codec parameters - each adds more attributes. WebRTC's SDP is particularly complex, often requiring libraries to generate and parse correctly.

For iOS developers, you rarely write SDP manually. WebRTC libraries handle SDP generation/parsing. But understanding SDP is crucial for debugging. When calls fail to connect, when audio doesn't work, when video is black - the answer is often in the SDP. Learning to read SDP is like learning to read Matrix code.

## Connections
→ [[ice_candidates]] - Listed in SDP
→ [[codec_negotiation]] - Primary SDP purpose
→ [[dtls_fingerprint]] - Security parameters
← [[webrtc_protocol]] - Uses SDP for negotiation
← [[sip_protocol]] - Carries SDP in message body
← [[signaling_server]] - Forwards SDP between peers

---
Level: L2
Date: 2025-08-15
Tags: #sdp #protocol #negotiation #media