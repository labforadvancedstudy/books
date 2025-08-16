# CallKit Framework

## Core Insight
CallKit is iOS's bridge between your VoIP app and the native phone experience - it makes third-party calls feel like first-party calls.

CallKit fundamentally transforms VoIP apps from isolated islands into first-class citizens of the iOS phone ecosystem. When you receive a VoIP call, it appears on the lock screen exactly like a cellular call. The same UI, the same ringtone options, the same integration with CarPlay and Bluetooth devices.

The framework operates through `CXProvider` and `CXCallController`. The provider handles incoming calls and system events, while the controller initiates outgoing calls and manages call state. This separation ensures your app respects system-wide call policies - if the user is already on a cellular call, CallKit coordinates who gets audio priority.

The magic happens through a dance of callbacks. When a VoIP push arrives, you report a new incoming call to CallKit. The system takes over the UI, and when the user answers, CallKit tells your app to configure audio and start the actual connection. Your app never directly controls the call UI - you just provide metadata and respond to actions.

This delegation model means CallKit can enforce system policies uniformly. Do Not Disturb works automatically. Emergency calls can interrupt your VoIP call. The recent calls list in the Phone app shows your app's calls. It's not just UI consistency - it's behavioral consistency across the entire system.

## Connections
→ [[pushkit_framework]] - VoIP pushes trigger CallKit
→ [[avaudiosession]] - Audio configuration after call answers
→ [[voip_push_handling]] - Wake app to report incoming call
→ [[call_state_machine]] - Maps CallKit events to app state
← [[app_store_voip_rules]] - CallKit required for VoIP apps
← [[callkit_restrictions]] - System-imposed limitations

---
Level: L1
Date: 2025-08-15
Tags: #ios #callkit #system-integration #framework