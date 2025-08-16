# State Synchronization

## Core Insight
State synchronization in distributed VoIP is maintaining consensus about call reality - ensuring all parties agree on whether they're connected, muted, or hung up.

The fundamental problem is network partitions. Alice sends "hang up" but the message is lost. She thinks the call ended, Bob thinks it's ongoing. Without synchronization mechanisms, these split-brain scenarios accumulate, creating confused states where participants have different views of reality.

Event sourcing provides one solution. Instead of synchronizing state directly, synchronize events that cause state changes. "User pressed answer at timestamp T" rather than "call is now connected." Each client replays events to derive state. If events arrive out of order, timestamps allow correct reconstruction.

CRDTs (Conflict-free Replicated Data Types) offer eventual consistency. Call state becomes a CRDT where operations commute - order doesn't matter. "Add participant" and "mute myself" can happen in any order with the same result. This tolerates network delays and partitions but limits what operations are possible.

Heartbeats detect desynchronization. Clients periodically exchange state checksums. Mismatch triggers reconciliation. "I think we're in state X" / "I think we're in state Y" / "Let's agree on state Z." This catches silent failures where one party's state corrupted without obvious symptoms.

The source of truth matters. Is the server authoritative (it decides call state) or are clients (they negotiate directly)? Server authority simplifies consistency but adds latency and becomes a failure point. Client authority enables P2P but requires complex consensus protocols.

Timeouts provide automatic reconciliation. If no heartbeat for 30 seconds, assume disconnection. If no media for 10 seconds, show "connection problem." These timeouts prevent zombie states where one party left but others don't know. But timeouts must be tuned carefully - too short causes false disconnections, too long leaves users confused.

## Connections
→ [[event_sourcing]] - State derivation
→ [[crdt_patterns]] - Consistency approach
→ [[heartbeat_protocol]] - Liveness checking
← [[call_state_machine]] - What to synchronize
← [[distributed_systems]] - Core challenge
← [[reconnection_strategy]] - Recovery mechanism

---
Level: L9
Date: 2025-08-15
Tags: #synchronization #distributed #consistency #state