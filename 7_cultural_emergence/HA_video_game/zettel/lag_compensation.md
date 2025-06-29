# Lag Compensation

## Core Insight
Time travel as a service - where games bend causality to make distant players feel present.

When you shoot someone in an online game, you're shooting at the past. They've already moved. By the time your bullet arrives at the server, arrives at their client, they're somewhere else. Lag compensation is the beautiful lie that makes multiplayer possible.

Client-side prediction: your game guesses what will happen. Server reconciliation: reality corrects your guess. Interpolation: smooth over the differences. Rollback: when wrong, rewind time and replay with correct data. These techniques create the illusion of shared presence across continents.

It's applied philosophy: if two players experience different realities, which is true? Answer: both, neither, whatever feels fair.

## Connections
→ [[client_prediction]]
→ [[server_authoritative]]
→ [[rollback_netcode]]
→ [[interpolation]]
← [[network_latency]]
← [[tick_rate]]

---
Level: L5
Date: 2025-06-23
Tags: #networking #multiplayer #technical