# MVVM vs TCA: 아키텍처의 진화

## Core Insight
아키텍처는 복잡성을 관리하는 방식이며, MVVM에서 TCA로의 진화는 함수형 패러다임과 상태 예측 가능성의 승리다.

MVVM(Model-View-ViewModel)은 iOS 개발의 오랜 친구였다. UIKit 시대부터 우리를 괴롭혔던 Massive View Controller 문제의 해결책으로 등장했다. 뷰는 순수하게 표시만 하고, 뷰모델이 비즈니스 로직을 담당하고, 모델은 데이터를 관리한다. 깔끔하고 테스트하기 좋고, 관심사의 분리가 명확하다.

그런데 SwiftUI와 함께 새로운 패러다임이 등장했다. The Composable Architecture(TCA). Redux에서 영감을 받은 단방향 데이터 플로우 아키텍처다. 상태(State), 액션(Action), 리듀서(Reducer), 이펙트(Effect)로 구성된다.

**TCA의 혁명적 아이디어:**
- 모든 상태 변화는 액션을 통해서만 발생
- 리듀서는 순수 함수로 새로운 상태를 반환
- 사이드 이펙트는 명시적으로 분리
- 시간 여행 디버깅 가능

```swift
// MVVM 방식
class UserViewModel: ObservableObject {
    @Published var users: [User] = []
    @Published var isLoading = false
    
    func loadUsers() {
        isLoading = true
        // 네트워킹 코드...
    }
}

// TCA 방식
struct UserFeature: Reducer {
    struct State {
        var users: [User] = []
        var isLoading = false
    }
    
    enum Action {
        case loadUsers
        case usersLoaded([User])
    }
    
    func reduce(into state: inout State, action: Action) -> Effect<Action> {
        switch action {
        case .loadUsers:
            state.isLoading = true
            return .run { send in
                let users = await loadUsers()
                await send(.usersLoaded(users))
            }
        case .usersLoaded(let users):
            state.users = users
            state.isLoading = false
            return .none
        }
    }
}
```

TCA는 더 많은 보일러플레이트를 요구하지만, 대신 완전한 예측 가능성을 제공한다. 복잡한 앱에서는 이 트레이드오프가 충분히 가치 있다. 특히 여러 개발자가 협업하는 대규모 프로젝트에서 TCA의 엄격함은 구원이 된다.

**언제 MVVM을 쓸까:**
- 작은 프로젝트
- 빠른 프로토타이핑
- 팀이 아직 함수형 패러다임에 익숙하지 않을 때

**언제 TCA를 쓸까:**
- 복잡한 상태 관리가 필요한 앱
- 테스트 커버리지가 중요한 프로젝트
- 디버깅과 상태 추적이 핵심인 경우

아키텍처 선택은 트레이드오프다. 복잡성을 어디에 둘 것인가의 문제다. MVVM은 복잡성을 분산시키고, TCA는 복잡성을 중앙화한다. 어느 쪽이 더 관리하기 쉬운가는 팀과 프로젝트에 달려 있다.

## Connections
→ [[291-dependency-injection-strategies]]
→ [[292-modular-architecture-spm]]
← [[003-user-intention-architecture]]
← [[030-swiftui-state-management]]

---
Level: L6
Date: 2025-08-16
Tags: #architecture #mvvm #tca #state-management #composable-architecture