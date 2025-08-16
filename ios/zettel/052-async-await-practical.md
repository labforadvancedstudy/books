# async/await 실전 코드

## Core Insight
async/await는 콜백 지옥을 선형 코드로 바꾼다. 비동기가 동기처럼 읽힌다.

```swift
// Before: Callback Hell
func fetchUser(id: String, completion: @escaping (User?) -> Void) {
    api.getUser(id) { user in
        api.getPosts(user.id) { posts in
            api.getComments(posts) { comments in
                completion(User(posts: posts, comments: comments))
            }
        }
    }
}

// After: Linear Paradise
func fetchUser(id: String) async throws -> User {
    let user = try await api.getUser(id)
    let posts = try await api.getPosts(user.id)
    let comments = try await api.getComments(posts)
    return User(posts: posts, comments: comments)
}

// 병렬 처리
async let user = api.getUser(id)
async let settings = api.getSettings()
let (userData, userSettings) = try await (user, settings)

// TaskGroup으로 동적 병렬
let images = try await withThrowingTaskGroup(of: UIImage.self) { group in
    for url in imageURLs {
        group.addTask { try await downloadImage(from: url) }
    }
    return try await group.reduce(into: []) { $0.append($1) }
}
```

에러 처리가 자연스럽다. try/catch가 동기 코드와 동일하게 작동한다.

## Connections
→ [[053-actor-implementation]]
→ [[054-mainactor-usage]]
← [[026-swift6-concurrency-revolution]]

---
Level: L0
Date: 2025-08-15
Tags: #ios #swift #async-await #concurrency #code