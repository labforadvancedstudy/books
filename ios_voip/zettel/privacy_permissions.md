# Privacy Permissions

## Core Insight
Privacy permissions are trust negotiations with users - each request is a micro-transaction where you trade functionality for access to sensitive data.

Microphone permission is non-negotiable for VoIP. Without it, no calls are possible. But timing matters. Request too early (on first launch), and users deny because they don't understand why. Request during the first call attempt, and the call fails while waiting for permission. Most apps pre-prompt: explain why you need microphone, then request permission.

The permission prompt text is your only chance to explain. Apple provides a standard prefix ("AppName would like to access your microphone") but you control the description. "To make voice calls" is too vague. "To let you talk during voice and video calls with your contacts" provides context. This text significantly affects acceptance rates.

Contacts permission is optional but valuable. With it, you can match phone numbers to names, show contact photos, and integrate with the system contacts. Without it, users must manually add contacts within your app. But contacts are sensitive - users often deny this permission. Design your app to work well without it.

Notifications permission enables call alerts when your app is backgrounded but not terminated. While VoIP pushes work without notification permission, regular pushes (for messages, missed calls) require it. The prompt appears on first push registration, so time it after users understand your app's value.

Camera permission is required for video calls. Unlike microphone, you can defer this until the first video call. But handle the async permission flow carefully - the call might already be connected when permission is granted. Queue video frames while waiting, then start sending once approved.

iOS 14+ shows indicators when microphone or camera are active. The orange dot (microphone) or green dot (camera) appears in the status bar. Users can tap to see which app is recording. This transparency means you must stop audio/video capture immediately when calls end.

## Connections
→ [[permission_prompts]] - User communication
→ [[microphone_access]] - Core requirement
→ [[contacts_integration]] - Optional enhancement
← [[app_store_voip_rules]] - Privacy requirements
← [[user_trust]] - Building confidence
← [[privacy_indicators]] - System transparency

---
Level: L8
Date: 2025-08-15
Tags: #privacy #permissions #ios #trust