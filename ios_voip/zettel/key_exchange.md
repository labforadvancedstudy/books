# Key Exchange

## Core Insight
Key exchange is cryptographic agreement at a distance - two parties calculating the same secret without ever transmitting it, even when enemies are listening.

Diffie-Hellman key exchange is the foundation. Alice and Bob each generate random private keys, compute public keys, exchange them, and derive a shared secret. Even if Eve intercepts all communications, she can't compute the secret without solving the discrete logarithm problem. This mathematical magic enables secure communication over insecure channels.

But basic DH is vulnerable to man-in-the-middle attacks. Without authentication, Alice might be exchanging keys with Eve pretending to be Bob. This is why key exchange must be combined with identity verification - certificates, pre-shared keys, or out-of-band verification like comparing fingerprints.

Perfect Forward Secrecy requires ephemeral keys. Generate new key pairs for each call, derive session keys, then destroy private keys after use. Even if long-term keys are compromised later, past calls remain secure. The cost is computational - generating key pairs for every call - but modern phones handle this easily.

For VoIP, key exchange timing is critical. Exchange during call setup adds latency. Pre-exchange before calls requires predicting who might call whom. Most systems use hybrid approaches: long-term keys for identity, ephemeral keys for sessions, with caching for frequently-called contacts.

The Signal Protocol's Double Ratchet advances key exchange further. Keys ratchet forward with each message, providing forward secrecy within calls. Even if current keys leak, past and future audio remains secure. This continuous key evolution is elegant but complex to implement correctly.

## Connections
→ [[diffie_hellman]] - Core algorithm
→ [[forward_secrecy]] - Security property
→ [[identity_verification]] - Prevents MITM
← [[e2e_encryption]] - Requires key agreement
← [[dtls_handshake]] - Includes key exchange
← [[signal_protocol]] - Advanced implementation

---
Level: L6
Date: 2025-08-15
Tags: #cryptography #key-exchange #security