# Combine: 데이터 흐름의 선언적 설계

## Core Insight
Combine은 시간에 따른 값의 변화를 선언적으로 다루는 함수형 반응형 프로그래밍 패러다임이다. 상태 변화를 스트림으로 모델링한다.

```swift
// 복잡한 비동기 로직을 선언적으로 표현
searchTextField.textPublisher
    .debounce(for: .milliseconds(300), scheduler: DispatchQueue.main)
    .removeDuplicates()
    .flatMap { query in
        searchService.search(query: query)
            .catch { _ in Just([]) }
    }
    .assign(to: \.searchResults, on: self)
    .store(in: &cancellables)
```

Combine의 세 가지 핵심 개념:
1. **Publisher**: 값을 시간에 따라 방출
2. **Subscriber**: 값을 받아서 처리
3. **Operator**: 값을 변환하고 조합

가장 혁신적인 부분은 **backpressure handling**이다. 생산자가 소비자보다 빠를 때를 자동으로 처리한다.

SwiftUI와의 완벽한 통합:
```swift
class ViewModel: ObservableObject {
    @Published var isLoading = false
    @Published var items: [Item] = []
    
    func loadItems() {
        isLoading = true
        
        itemService.fetchItems()
            .receive(on: DispatchQueue.main)
            .sink(
                receiveCompletion: { _ in self.isLoading = false },
                receiveValue: { self.items = $0 }
            )
            .store(in: &cancellables)
    }
}
```

하지만 async/await의 등장으로 Combine의 위치가 애매해졌다:
- **Simple async tasks**: async/await가 더 간단
- **Stream processing**: Combine이 여전히 강력
- **UI binding**: @Published가 여전히 유용

현실: 많은 프로젝트가 Combine을 도입했다가 async/await로 마이그레이션 중이다.

## Connections
→ [[231-published-property-wrapper]]
→ [[232-custom-publishers]]
→ [[233-combine-vs-async-await]]
← [[030-swiftui-state-management]]

---
Level: L5
Date: 2025-08-16
Tags: #ios #combine #reactive #functional #swiftui #async