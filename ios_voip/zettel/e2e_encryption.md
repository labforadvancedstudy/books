# End-to-End Encryption

## Core Insight
E2E encryption ensures only the calling parties can decrypt the media - not even your servers can eavesdrop on the conversation.

End-to-end encryption for VoIP is more complex than messaging. With messages, you encrypt once and store. With real-time media, you're encrypting thousands of packets per second, each needing authentication and replay protection. The encryption must add minimal latency while remaining secure against sophisticated attacks.

WebRTC provides SRTP (Secure RTP) by default, but this is hop-by-hop encryption - secure to your server, but the server can decrypt. True E2E requires key exchange directly between clients. Signal Protocol pioneered this with their Double Ratchet algorithm, combining Diffie-Hellman key agreement with symmetric-key ratcheting for forward secrecy.

The key exchange happens during call setup. Clients generate ephemeral key pairs, exchange public keys through the signaling channel (potentially through multiple servers), and derive shared secrets. But how do you know you're talking to the right person? This requires identity verification - showing key fingerprints, QR codes, or using trusted public key infrastructure.

SRTP with E2E keys encrypts each RTP packet's payload while leaving headers visible for routing. Each packet includes an authentication tag preventing tampering. Sequence numbers prevent replay attacks. The encryption is typically AES-GCM or ChaCha20-Poly1305, chosen for speed and security.

Key rotation is crucial for forward secrecy. Many implementations rotate keys every few minutes or thousand packets. If keys are compromised later, past conversations remain secure. But rotation must be synchronized - both parties switch keys simultaneously without disrupting the media stream.

## Connections
→ [[srtp_protocol]] - Encryption mechanism
→ [[key_exchange]] - Establishing shared secrets
→ [[dtls_handshake]] - Alternative key agreement
→ [[signal_protocol]] - E2E implementation
← [[privacy_requirements]] - Motivation for E2E
← [[webrtc_protocol]] - Provides SRTP framework

---
Level: L6
Date: 2025-08-15
Tags: #encryption #security #e2e #privacy