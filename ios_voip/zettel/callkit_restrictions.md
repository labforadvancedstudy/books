# CallKit Restrictions

## Core Insight
CallKit restrictions are Apple's leash on VoIP apps - you get native integration but lose control over critical aspects of the calling experience.

The most painful restriction: you cannot customize the incoming call screen. No custom ringtones (only system sounds), no video preview, no custom buttons beyond audio/video toggle. Your beautifully designed call UI is replaced by Apple's standard interface. This ensures consistency but eliminates differentiation.

VoIP push requirements are strict and unforgiving. Receive a VoIP push? You MUST call reportNewIncomingCall within seconds. No exceptions for invalid pushes, decryption failures, or server errors. Report a call or crash. This prevents background abuse but means your error handling must be bulletproof.

Audio session activation is controlled by CallKit. You cannot activate AVAudioSession until CallKit tells you to (in the didActivateAudioSession callback). Activate early? Your app might be terminated. This coordination ensures system audio priority but complicates audio testing and configuration.

Call identification has privacy restrictions. You can't access the phone's contacts directly from the CallKit provider. The phone number or handle you report shows as-is unless the user has a matching contact. No enriching with profile pictures or additional info - that's iOS's job.

Multiple calls are limited. While CallKit supports multiple simultaneous calls, the UI becomes confusing. Users see multiple call screens, must manually switch between calls, and might not understand what's happening. Most VoIP apps artificially limit to one active call to avoid confusion.

Background execution is tightly coupled to CallKit state. End the CallKit call, and your background privileges end immediately. You can't maintain the network connection for post-call analytics or cleanup. Everything must happen while the call is active or be deferred to the next foreground launch.

## Connections
→ [[callkit_framework]] - Source of restrictions
→ [[voip_push_handling]] - Strict requirements
→ [[audio_session_timing]] - Activation limits
← [[app_store_voip_rules]] - Enforcement mechanism
← [[custom_ui_limitations]] - Design constraints
← [[privacy_boundaries]] - System protection

---
Level: L8
Date: 2025-08-15
Tags: #callkit #restrictions #ios #limitations