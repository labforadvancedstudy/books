# App Store VoIP Rules

## Core Insight
Apple's VoIP rules are strict guardrails ensuring apps behave like proper phone citizens - violate them and face rejection or removal.

The fundamental rule since iOS 13: if you receive a VoIP push, you MUST display a call UI using CallKit. No exceptions. This killed many apps that used VoIP pushes for instant messaging or presence updates. Apple enforces this through runtime crashes - fail to report a call quickly enough after a VoIP push, and iOS terminates your app.

CallKit integration is mandatory for VoIP apps. You can't build a custom call UI that replaces the system one. CallKit must handle the incoming call screen, the in-call UI must be accessible from the system UI, and calls must appear in the Phone app's recents. This ensures consistent user experience across all calling apps.

Background execution is tightly controlled. The VoIP background mode only activates during active calls reported to CallKit. You can't use it for standby, presence updates, or message sync. Apps caught abusing background modes face rejection. Apple's reviewers test this by monitoring CPU and network usage in background.

User privacy requirements are strict. You must request microphone permission before the first call. The permission prompt must clearly explain why you need microphone access. Recording calls requires explicit consent and clear indication that recording is active. Some jurisdictions require two-party consent for recording.

Export compliance is required if your app uses encryption (which all VoIP apps do). You need to submit annual self-classification reports to the US Bureau of Industry and Security. Apps with E2E encryption might face additional restrictions in certain countries. The App Store Connect requires you to declare encryption usage and provide export compliance documentation.

## Connections
→ [[callkit_framework]] - Mandatory integration
→ [[pushkit_framework]] - Strict usage rules
→ [[privacy_permissions]] - Required disclosures
→ [[export_compliance]] - Encryption regulations
← [[app_review_process]] - Enforcement mechanism
← [[voip_architecture]] - Shapes design decisions

---
Level: L8
Date: 2025-08-15
Tags: #appstore #compliance #requirements #ios