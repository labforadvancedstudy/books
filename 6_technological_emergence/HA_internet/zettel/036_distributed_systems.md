# Distributed Systems

## Core Insight
Distributed systems are the art of making many computers act as one - coordinating independent machines to work together despite distance, failures, and the speed of light.

It's a magic trick at scale. Users see one system. Behind the curtain: hundreds of servers, different countries, various states, all pretending to be a single coherent service. Like an orchestra where musicians can't see each other, instruments randomly disappear, and the speed of sound varies.

The CAP theorem rules this world: Consistency, Availability, Partition tolerance - pick two. You can't have all three. Physics won't let you. Every distributed system is a compromise, choosing which promise to break when reality bites.

Consensus is the holy grail. How do distant computers agree on anything? Paxos, Raft, Byzantine Generals - algorithms that sound like fantasy novels but determine whether your bank transaction succeeds. Getting computers to agree is harder than getting humans to agree.

Failure isn't a bug in distributed systems - it's a feature. Servers will crash. Networks will partition. Disks will die. The art is building systems that expect failure, embrace it, route around it. Resilience through pessimism.

## Connections
→ [[037_cloud_computing]]
→ [[038_edge_computing]]
→ [[039_microservices]]
← [[031_load_balancing]]
← [[035_databases]]

---
Level: L3
Date: 2025-06-23
Tags: #architecture #reliability #scalability #consensus