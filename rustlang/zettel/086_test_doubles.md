# Test Doubles in Rust

## 핵심 개념
테스트를 위한 대역 객체 - Mock, Stub, Fake, Spy, Dummy

## Stub 구현
```rust
struct StubUserRepository {
    users: Vec<User>,
}

impl UserRepository for StubUserRepository {
    async fn find_by_id(&self, id: UserId) -> Result<Option<User>, Error> {
        Ok(self.users.iter().find(|u| u.id == id).cloned())
    }
    
    async fn save(&self, _user: &User) -> Result<(), Error> {
        Ok(()) // 아무것도 하지 않음
    }
}

#[tokio::test]
async fn test_with_stub() {
    let repo = StubUserRepository {
        users: vec![User::new("test@example.com")],
    };
    let service = UserService::new(repo);
    // 테스트 로직
}
```

## Mock 구현 (mockall 크레이트)
```rust
use mockall::{automock, predicate::*};

#[automock]
trait EmailService {
    async fn send_welcome(&self, email: &str) -> Result<(), Error>;
}

#[tokio::test]
async fn test_email_sent() {
    let mut mock = MockEmailService::new();
    mock.expect_send_welcome()
        .with(eq("user@example.com"))
        .times(1)
        .returning(|_| Ok(()));
    
    let service = UserService::new(mock);
    service.register("user@example.com").await.unwrap();
    // mock은 자동으로 검증됨
}
```

## Fake 구현
```rust
pub struct FakeDatabase {
    data: Arc<RwLock<HashMap<String, String>>>,
}

impl Database for FakeDatabase {
    async fn get(&self, key: &str) -> Option<String> {
        self.data.read().await.get(key).cloned()
    }
    
    async fn set(&self, key: String, value: String) {
        self.data.write().await.insert(key, value);
    }
}

#[tokio::test]
async fn test_with_fake() {
    let fake_db = FakeDatabase::new();
    fake_db.set("key", "value").await;
    assert_eq!(fake_db.get("key").await, Some("value".to_string()));
}
```

## Spy 구현
```rust
pub struct SpyLogger {
    messages: Arc<RwLock<Vec<String>>>,
}

impl Logger for SpyLogger {
    fn log(&self, message: &str) {
        self.messages.write().unwrap().push(message.to_string());
    }
}

impl SpyLogger {
    pub fn messages(&self) -> Vec<String> {
        self.messages.read().unwrap().clone()
    }
    
    pub fn assert_logged(&self, expected: &str) {
        assert!(self.messages().iter().any(|m| m.contains(expected)));
    }
}

#[test]
fn test_logging() {
    let spy = SpyLogger::new();
    let service = Service::new(spy.clone());
    
    service.do_something();
    
    spy.assert_logged("Something was done");
}
```

## Dummy 구현
```rust
struct DummyConfig;

impl Config for DummyConfig {
    fn get(&self, _key: &str) -> Option<String> {
        panic!("Dummy should not be called");
    }
}

#[test]
fn test_without_config() {
    // Config가 필요하지만 사용하지 않는 경우
    let service = Service::new(DummyConfig);
    // Config를 사용하지 않는 메서드만 테스트
    assert_eq!(service.simple_calculation(), 42);
}
```

## 커스텀 매처
```rust
use mockall::predicate::{Predicate, PredicateStrExt};

fn email_predicate() -> impl Predicate<str> {
    predicate::str::contains("@")
        .and(predicate::str::ends_with(".com"))
}

#[tokio::test]
async fn test_email_validation() {
    let mut mock = MockEmailService::new();
    mock.expect_send()
        .with(email_predicate())
        .returning(|_| Ok(()));
}
```

## 상태 기반 테스트
```rust
pub struct StatefulMock {
    call_count: Arc<AtomicUsize>,
    responses: Vec<String>,
}

impl Service for StatefulMock {
    fn call(&self) -> String {
        let count = self.call_count.fetch_add(1, Ordering::SeqCst);
        self.responses.get(count).cloned().unwrap_or_default()
    }
}
```

## 빌더 패턴 테스트 더블
```rust
pub struct MockBuilder {
    expected_calls: Vec<ExpectedCall>,
}

impl MockBuilder {
    pub fn expect_call(mut self, method: &str, args: Vec<Value>) -> Self {
        self.expected_calls.push(ExpectedCall { method, args });
        self
    }
    
    pub fn build(self) -> Mock {
        Mock::new(self.expected_calls)
    }
}
```

## Rust 특성 활용
- **트레이트**: 테스트 더블 쉽게 교체
- **Arc/RwLock**: 상태 공유
- **매크로**: Mock 자동 생성
- **클로저**: 동적 동작 정의

## 테스트 더블 선택 기준
- **Dummy**: 파라미터만 채우기
- **Stub**: 고정된 응답 반환
- **Fake**: 간단한 구현체
- **Spy**: 호출 기록 및 검증
- **Mock**: 기대값 설정 및 검증

## 관련 개념
- [[085_dependency_injection]] - DI로 테스트 더블 주입
- [[091_property_testing]] - 속성 기반 테스트
- [[092_test_fixtures]] - 테스트 픽스처