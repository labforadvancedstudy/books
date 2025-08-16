# Swift 6: 동시성의 혁명

## Core Insight
Swift 6의 동시성은 문법이 아니라 사고방식의 전환이다. async/await는 비동기를 동기처럼 생각하게 만드는 마법이다.

Data race를 컴파일 타임에 방지한다는 것은 무엇을 의미하는가? 더 이상 "조심해서 코딩하기"가 아니라 "잘못된 코드는 컴파일되지 않기"다. Sendable, Actor, @MainActor - 이들은 안전을 강제하는 타입 시스템이다.

Actor는 상태를 격리한다. 각 Actor는 자신만의 섬이고, async를 통해서만 소통한다. 이는 제약이 아니라 해방이다. 더 이상 lock과 semaphore를 고민할 필요 없다. 컴파일러가 동시성을 보장한다.

Task, TaskGroup, AsyncSequence는 동시성을 구조화한다. 병렬 처리가 for loop처럼 간단해진다. Structured Concurrency는 동시성을 예측 가능하고 취소 가능하게 만든다.

## Connections
→ [[027-actor-isolation-pattern]]
→ [[028-structured-concurrency]]
← [[029-swift-evolution]]

---
Level: L4
Date: 2025-08-15
Tags: #ios #swift6 #concurrency #async-await #actors