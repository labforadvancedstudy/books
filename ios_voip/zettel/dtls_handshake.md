# DTLS Handshake

## Core Insight
DTLS brings TLS security to UDP - performing cryptographic handshakes over unreliable networks to establish keys for SRTP media encryption.

Datagram Transport Layer Security solves a unique problem: how do you perform a TLS-style handshake when packets might arrive out of order, duplicated, or not at all? TCP guarantees ordering, UDP doesn't. DTLS adds sequence numbers, retransmission timers, and replay detection to make TLS work over UDP.

The handshake follows TLS 1.2 or 1.3 patterns. Client sends ClientHello with supported cipher suites. Server responds with ServerHello choosing the cipher, plus its certificate. They exchange keys using Diffie-Hellman (ECDHE typically). Both sides derive master secrets and confirm with Finished messages. But unlike TLS, each message includes epoch and sequence numbers for ordering.

For WebRTC, DTLS-SRTP is the standard. The DTLS handshake happens after ICE establishes connectivity but before media flows. Both peers act as DTLS client and server simultaneously (different roles for different components). The handshake typically completes in 2-3 round trips - adding 50-200ms to call setup.

Certificate verification is WebRTC's clever innovation. Instead of traditional PKI with certificate authorities, WebRTC uses fingerprints. Each peer generates a self-signed certificate, computes its SHA-256 fingerprint, and includes it in the SDP. After DTLS handshake, peers verify the certificate matches the fingerprint. This prevents man-in-the-middle attacks without requiring certificate infrastructure.

The handshake produces keying material for SRTP. DTLS exports keys using the TLS "exporter" mechanism. These become SRTP master keys for encrypting media. The beauty is separation of concerns: DTLS handles key agreement, SRTP handles media encryption. Each protocol does what it's best at.

## Connections
→ [[srtp_protocol]] - Uses keys from DTLS
→ [[certificate_fingerprint]] - Verification mechanism
→ [[ice_protocol]] - Must complete first
← [[webrtc_protocol]] - Mandates DTLS-SRTP
← [[sdp_format]] - Carries fingerprints
← [[security_handshake]] - Key establishment

---
Level: L6
Date: 2025-08-15
Tags: #dtls #security #handshake #encryption