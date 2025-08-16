# PushKit Framework

## Core Insight
PushKit is a high-priority push system that can wake your terminated app instantly - it's the only way iOS allows VoIP apps to receive calls when not running.

Unlike regular push notifications that might be delayed or throttled, PushKit pushes are delivered immediately and can launch your app in the background. This is critical for VoIP because users expect calls to ring instantly, just like phone calls. You can't wait for the user to tap a notification.

The framework operates on a special entitlement - `voip` type pushes. When your server sends a VoIP push to Apple's Push Notification service with the right priority flag, iOS treats it as urgent. Even if your app was force-quit, iOS will launch it in the background and give you execution time.

But with great power comes great responsibility. Since iOS 13, you MUST report an incoming call to CallKit when you receive a VoIP push. No exceptions. If you don't call `reportNewIncomingCall` fast enough, iOS will crash your app. This prevents abuse where apps used VoIP pushes for background processing instead of actual calls.

The registration flow uses `PKPushRegistry`. You set yourself as the delegate, request `voip` type pushes, and receive a device token. This token is different from your regular APNs token - it's specifically for VoIP pushes. Your server needs to use this token with the right push type, or the pushes won't have VoIP priority.

## Connections
→ [[callkit_framework]] - Must report call immediately
→ [[voip_push_handling]] - Processing incoming push payload
→ [[background_modes]] - App launches in background
← [[signaling_server]] - Sends VoIP pushes for incoming calls
← [[app_store_voip_rules]] - Required for VoIP functionality

---
Level: L1
Date: 2025-08-15
Tags: #ios #pushkit #notifications #voip