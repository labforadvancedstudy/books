# Background Modes

## Core Insight
Background modes are iOS's battery-saving compromise - apps can't run freely in the background, but certain activities like VoIP calls get special exemptions.

iOS aggressively suspends apps to save battery. When users switch apps or lock the screen, your app typically gets 30 seconds to wrap up, then suspends. But VoIP apps need to maintain calls for hours. The "Voice over IP" background mode grants this privilege, keeping your app running during active calls.

The background mode is configured in Info.plist but managed dynamically by CallKit. When a call is active (reported to CallKit and not ended), iOS keeps your app running. Your network connections stay alive, timers keep firing, audio keeps flowing. End the call, and you return to normal background restrictions.

But "running" doesn't mean full performance. iOS may throttle your CPU, limit memory, or defer non-essential work. Background apps get lower scheduling priority - your audio processing might get preempted by foreground apps. This is why efficient code matters more in background than foreground.

The audio session plays a crucial role. An active AVAudioSession with `.playAndRecord` category signals iOS that you're doing real-time audio. This maintains higher CPU priority and prevents audio glitches. But activate the session without an active call, and iOS might terminate your app for abuse.

Background VoIP also enables special behaviors. Your app can receive VoIP pushes while terminated. Network connections can use "backgroundSessionConfiguration" for reliability. But with great power comes scrutiny - Apple reviews background mode usage carefully. Abuse it for non-VoIP purposes, and your app gets rejected.

## Connections
→ [[voip_push_handling]] - Launches app in background
→ [[avaudiosession]] - Maintains background priority
→ [[callkit_framework]] - Manages background lifecycle
→ [[battery_optimization]] - Constrained resources
← [[app_lifecycle]] - Special background states
← [[network_reliability]] - Background network behavior

---
Level: L5
Date: 2025-08-15
Tags: #ios #background #lifecycle #voip