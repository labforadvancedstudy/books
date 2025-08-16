# VoIP Push Handling

## Core Insight
VoIP push handling is a race against time - you have seconds to wake up, process the push, and show the call UI before iOS kills your app.

When a VoIP push arrives, iOS launches your app in the background and calls `pushRegistry:didReceiveIncomingPushWith:`. You now have approximately 5 seconds to call `reportNewIncomingCall` on CallKit. Miss this deadline, and iOS crashes your app with a termination reason. This strict requirement prevents apps from abusing VoIP pushes for background processing.

The push payload should contain everything needed to display the call: caller name, caller ID, maybe an avatar URL. You can't make network requests here - there's no time, and the network might not be ready. Parse the push, create a `CXCallUpdate` with the caller info, and report to CallKit immediately. Network operations come later, after the user answers.

The execution environment is constrained. Your app launches in the background with limited memory and CPU. Core Data might not be ready. Network might be initializing. You need a fast path that bypasses normal app initialization. Many apps maintain a separate "push handler" that can operate with minimal dependencies.

State management is tricky. The app might be launching fresh, resuming from background, or already active. The same push might arrive multiple times (APNs can duplicate). You need idempotent handling - use a unique call UUID in the push, check if you've already reported this call, and ignore duplicates.

Error handling is critical. If the push is malformed, if decryption fails, if the caller ID is missing - you still must report something to CallKit. Report a call with generic info ("Unknown Caller") rather than crash. You can update the call information later when your app fully initializes.

## Connections
→ [[pushkit_framework]] - Delivers the push
→ [[callkit_framework]] - Must report immediately
→ [[background_modes]] - Execution environment
→ [[push_payload_format]] - What to include
← [[signaling_server]] - Sends the push
← [[app_lifecycle]] - Special launch mode

---
Level: L5
Date: 2025-08-15
Tags: #voip #push #background #ios