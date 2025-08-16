# CQRS (Command Query Responsibility Segregation) in Rust

## 핵심 개념
명령(Command)과 조회(Query)를 분리하여 각각 최적화

## 명령 정의
```rust
pub trait Command {
    type Result;
    type Error;
}

pub struct CreateUserCommand {
    pub email: String,
    pub name: String,
}

impl Command for CreateUserCommand {
    type Result = UserId;
    type Error = DomainError;
}
```

## 조회 정의
```rust
pub trait Query {
    type Result;
}

pub struct GetUserByIdQuery {
    pub id: UserId,
}

impl Query for GetUserByIdQuery {
    type Result = Option<UserDto>;
}
```

## 명령 핸들러
```rust
pub trait CommandHandler<C: Command> {
    async fn handle(&self, command: C) -> Result<C::Result, C::Error>;
}

pub struct CreateUserHandler<R: UserRepository> {
    repository: R,
    event_bus: EventBus,
}

impl<R: UserRepository> CommandHandler<CreateUserCommand> for CreateUserHandler<R> {
    async fn handle(&self, cmd: CreateUserCommand) -> Result<UserId, DomainError> {
        // 도메인 로직 실행
        let user = User::create(cmd.email, cmd.name)?;
        
        // 저장
        self.repository.save(&user).await?;
        
        // 이벤트 발행
        self.event_bus.publish(UserCreatedEvent {
            user_id: user.id(),
            timestamp: Utc::now(),
        }).await;
        
        Ok(user.id())
    }
}
```

## 조회 핸들러
```rust
pub trait QueryHandler<Q: Query> {
    async fn handle(&self, query: Q) -> Q::Result;
}

pub struct GetUserByIdHandler {
    read_model: ReadModelStore,
}

impl QueryHandler<GetUserByIdQuery> for GetUserByIdHandler {
    async fn handle(&self, query: GetUserByIdQuery) -> Option<UserDto> {
        // 읽기 최적화된 모델에서 조회
        self.read_model.get_user(query.id).await
    }
}
```

## 이벤트 소싱 통합
```rust
pub trait Event {
    fn aggregate_id(&self) -> AggregateId;
    fn event_type(&self) -> &str;
}

pub struct EventStore {
    storage: Box<dyn EventStorage>,
}

impl EventStore {
    pub async fn append(&self, events: Vec<Box<dyn Event>>) -> Result<(), Error> {
        self.storage.append(events).await
    }
    
    pub async fn load(&self, id: AggregateId) -> Result<Vec<Box<dyn Event>>, Error> {
        self.storage.load(id).await
    }
}

// 애그리거트 재구성
pub fn rebuild_aggregate(events: Vec<Box<dyn Event>>) -> User {
    let mut user = User::default();
    for event in events {
        user.apply(event);
    }
    user
}
```

## 읽기 모델 프로젝션
```rust
pub struct UserProjection {
    read_store: ReadModelStore,
}

impl EventHandler<UserCreatedEvent> for UserProjection {
    async fn handle(&self, event: UserCreatedEvent) {
        // 읽기 모델 업데이트
        let dto = UserDto {
            id: event.user_id,
            created_at: event.timestamp,
            // 읽기에 최적화된 형태
        };
        self.read_store.save_user(dto).await;
    }
}
```

## 명령 버스
```rust
pub struct CommandBus {
    handlers: HashMap<TypeId, Box<dyn Any>>,
}

impl CommandBus {
    pub async fn dispatch<C: Command + 'static>(&self, command: C) -> Result<C::Result, C::Error> 
    where
        C::Result: 'static,
        C::Error: 'static,
    {
        let handler = self.handlers
            .get(&TypeId::of::<C>())
            .and_then(|h| h.downcast_ref::<Box<dyn CommandHandler<C>>>())
            .ok_or(BusError::HandlerNotFound)?;
            
        handler.handle(command).await
    }
}
```

## Rust 특성 활용
- **타입 시스템**: 명령/조회 분리 강제
- **트레이트**: 핸들러 추상화
- **async/await**: 비동기 처리
- **enum**: 이벤트 타입 안전성

## 장점
- 읽기/쓰기 독립 최적화
- 확장성 향상
- 복잡한 도메인 로직 관리
- 이벤트 소싱과 자연스러운 통합

## 관련 개념
- [[081_ddd_rust]] - 도메인 주도 설계
- [[088_event_sourcing]] - 이벤트 소싱
- [[089_event_bus]] - 이벤트 버스 구현