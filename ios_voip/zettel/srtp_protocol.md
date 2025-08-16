# SRTP Protocol

## Core Insight
SRTP wraps every media packet in cryptographic armor - providing confidentiality, authenticity, and replay protection without adding significant latency.

Secure Real-time Transport Protocol extends RTP with encryption and authentication. Every RTP packet gets encrypted (payload only, headers remain visible for routing) and authenticated (ensuring no tampering). This happens thousands of times per second with minimal overhead - crucial for real-time media.

The encryption uses AES in Counter Mode (AES-CTR) or newer algorithms like ChaCha20. Stream ciphers are essential - they allow encrypting variable-length packets without padding. Each packet uses a unique keystream derived from the master key and packet index. Even if packets arrive out of order, each decrypts independently.

Authentication prevents tampering and replay attacks. Each packet includes a Message Authentication Code (MAC) - typically HMAC-SHA1 or Poly1305. The MAC covers both the encrypted payload and unencrypted header. Any modification invalidates the MAC, causing packet rejection. The replay window prevents resending old packets.

Key management is SRTP's weakness. The protocol doesn't define how to exchange keys - that's left to the application. WebRTC uses DTLS-SRTP: a DTLS handshake establishes keys, then SRTP uses them for media. SIP might use SDES (keys in SDP) or ZRTP (Diffie-Hellman key agreement). Each has security tradeoffs.

Master keys derive session keys through a key derivation function (KDF). One master key generates separate keys for encryption, authentication, and salting. Keys can be ratcheted - deriving new keys periodically for forward secrecy. But ratcheting must be synchronized or packets become undecryptable.

Performance is critical. SRTP adds 10-14 bytes per packet for authentication tags. Processing must be fast - any delay affects call quality. Modern CPUs handle AES acceleration in hardware, making SRTP nearly free. But on older devices, the CPU cost is measurable.

## Connections
→ [[dtls_handshake]] - Key establishment
→ [[key_derivation]] - Session key generation
→ [[replay_protection]] - Security feature
← [[webrtc_protocol]] - Mandates SRTP
← [[e2e_encryption]] - Can layer on SRTP
← [[rtp_protocol]] - Underlying protocol

---
Level: L6
Date: 2025-08-15
Tags: #srtp #security #encryption #protocol