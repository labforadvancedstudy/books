# Export Compliance

## Core Insight
Export compliance is the hidden bureaucracy of encryption - using standard cryptography in your VoIP app triggers Cold War-era regulations that persist today.

Encryption is classified as dual-use technology - useful for both civilian and military purposes. VoIP apps use encryption (SRTP, TLS, potentially E2E), automatically triggering export regulations. In the US, this means compliance with EAR (Export Administration Regulations). Ignore this, and face penalties ranging from app removal to criminal prosecution.

The self-classification process starts in App Store Connect. Apple asks: "Does your app use encryption?" For VoIP, the answer is always yes. Next: "Does your app qualify for exemption?" If you only use standard encryption (HTTPS, SRTP) for authentication and confidentiality, you typically qualify for exemption "5D992.c" - no further action needed.

But custom or non-standard encryption requires more. Using Signal Protocol for E2E encryption? Implementing your own crypto? You need to submit annual self-classification reports to BIS (Bureau of Industry and Security) by February 1st. The report lists your app, encryption uses, and classification. It's mostly bureaucratic box-checking, but mandatory.

Country-specific restrictions complicate deployment. Some countries restrict or prohibit encrypted communications. China requires special licenses. Russia has notification requirements. France used to require declaration (no longer). Your app might need different versions or features disabled for certain markets.

The encryption disclosure in App Store Connect affects availability. Mark your app as using non-exempt encryption, and Apple automatically prevents distribution to certain countries. This might be desirable (avoiding legal issues) or problematic (reducing market reach). Consider your target markets during architecture decisions.

Open-source complications exist. If your app is open-source or includes encrypted communication in source code repositories, you might need to notify BIS. The regulations are murky here, with different interpretations of what constitutes "export" of cryptographic source code.

## Connections
→ [[encryption_classification]] - Determining requirements
→ [[bis_reporting]] - Annual compliance
→ [[country_restrictions]] - Market limitations
← [[e2e_encryption]] - Triggers requirements
← [[app_store_voip_rules]] - Apple enforcement
← [[international_deployment]] - Geographic constraints

---
Level: L8
Date: 2025-08-15
Tags: #compliance #encryption #export #legal