# AVAudioSession

## Core Insight
AVAudioSession is iOS's audio traffic controller - it manages which app gets the microphone and speaker, and how they're configured.

AVAudioSession is a singleton that negotiates your app's audio needs with the system. You don't directly access audio hardware - you tell AVAudioSession what you want, and it configures the hardware appropriately. This indirection allows iOS to manage competing audio needs from multiple apps and system services.

For VoIP, the magic category is `.playAndRecord`. This enables both microphone input and speaker output simultaneously. You typically add `.allowBluetooth` and `.defaultToSpeaker` options. The mode should be `.voiceChat` or `.videoChat`, which optimizes for two-way communication with echo cancellation and noise suppression.

The session must be activated at the right time. Activate too early, and you interrupt other apps unnecessarily. Activate too late, and your call has no audio. CallKit helps by telling you exactly when to configure audio - in the `provider:didActivateAudioSession:` callback. This ensures your audio doesn't conflict with system sounds or other calls.

Sample rate and buffer size configuration affects latency and quality. VoIP apps typically want 48kHz sample rate (matching Opus) and small buffer sizes (256-512 samples) for low latency. But smaller buffers mean more CPU wake-ups and higher battery drain. The `preferredIOBufferDuration` property lets you request a specific buffer duration.

Route changes are critical to handle. When users plug in headphones, connect Bluetooth, or switch to speaker, AVAudioSession posts notifications. Your app must respond appropriately - perhaps pausing audio processing during the transition to avoid glitches.

## Connections
→ [[audio_route_handling]] - Managing output devices
→ [[callkit_framework]] - Coordinates activation
→ [[echo_cancellation]] - Enabled by mode selection
→ [[background_modes]] - Keeps session active
← [[opus_codec]] - Processes audio from session
← [[voip_architecture]] - Core component

---
Level: L1
Date: 2025-08-15
Tags: #ios #audio #avfoundation #system