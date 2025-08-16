# Call State Machine

## Core Insight
A call state machine is the single source of truth that prevents chaos - it ensures both parties agree on whether they're calling, connected, or hung up.

Calls have complex lifecycles. Idle → Calling → Ringing → Connected → Ending → Ended. But reality is messier. What if both parties call simultaneously? What if one party's "hang up" message gets lost? What if the network dies mid-call? Without a formal state machine, these edge cases create confused states where Alice thinks she's talking to Bob, but Bob thinks the call ended.

The state machine enforces valid transitions. From Idle, you can only go to Calling (outgoing) or Ringing (incoming). From Connected, you can only go to Ending or Reconnecting. Invalid transitions are rejected - you can't answer a call that's already connected, can't hang up a call that never started.

Each state has associated behaviors. In Calling state, play ringback tone and show "Calling..." UI. In Connected state, enable audio and show call duration. In Reconnecting state, mute audio but maintain the call. States drive both UI and system behavior, ensuring consistency.

Events trigger transitions. User actions (answer, reject, hang up), network events (connected, disconnected), and timers (no answer timeout) all generate events. The state machine processes events based on current state - the same "hang up" event means different things in Calling versus Connected states.

For iOS, the state machine must sync with CallKit. When CallKit reports the user answered, transition to Connecting. When WebRTC connection establishes, transition to Connected. When either CallKit or WebRTC fails, handle appropriately. This dual synchronization - with system UI and network layer - is critical for reliable behavior.

## Connections
→ [[state_transitions]] - Valid state changes
→ [[call_events]] - Triggers for transitions
→ [[ui_synchronization]] - States drive interface
← [[callkit_framework]] - System state coordination
← [[webrtc_protocol]] - Network state events
← [[error_recovery]] - Handles invalid states

---
Level: L5
Date: 2025-08-15
Tags: #state-machine #call-lifecycle #architecture