# Connection Lag

## Core Insight
Connection lag is the dead time between intention and action - the gap where users exist in uncertainty, wondering if technology has failed them.

The moment after pressing "call" is psychological limbo. Did it work? Is it connecting? Why isn't it ringing yet? This 1-3 second gap needs careful design. Too much feedback feels nervous ("Connecting..." → "Finding peer..." → "Establishing secure connection..."). Too little feels broken (frozen UI with no indication).

Ring latency shapes first impressions. Press call, wait 5 seconds for ringing - users assume failure and hang up. But making it ring immediately when connection isn't established creates false confidence. The solution is staged feedback: immediate local confirmation ("Calling..."), then real ringing when the remote actually receives the call.

Answer lag is particularly frustrating. You swipe to answer, but audio doesn't start immediately. Are they there? Should you say hello? This gap happens while CallKit hands off to your app, your app configures audio, and media starts flowing. Every millisecond matters - users start talking at 500ms, creating awkward "Hello? Hello?" moments.

Network establishment adds invisible delays. ICE candidate gathering (1-2 seconds), STUN requests (RTT to server), DTLS handshake (2-3 round trips). These happen in parallel but still sum to 2-5 seconds before media flows. Users don't understand why "calling over the internet" takes longer than traditional phones.

Perceived lag differs from actual lag. Showing a spinner makes time feel longer. Showing progress ("Securing connection...") makes it feel purposeful. Playing local ringback immediately makes calling feel instant even if connection takes seconds. The UI can manipulate time perception.

Mobile-specific lags compound frustration. App launch from VoIP push (1-2 seconds), audio session activation (100-500ms), network radio wake-up (100-1000ms). Each small delay adds to perceived sluggishness. Optimization requires attacking each component.

## Connections
→ [[ice_establishment_time]] - Connection setup
→ [[ui_feedback]] - Managing perception
→ [[cold_start_time]] - App launch delay
← [[voip_call_experience]] - Part of feeling
← [[user_expectations]] - Shapes satisfaction
← [[performance_optimization]] - Reduction target

---
Level: L0
Date: 2025-08-15
Tags: #lag #latency #perception #ux