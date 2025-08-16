# Dependency Injection in Rust

## 핵심 개념
컴파일 타임과 런타임 DI 패턴으로 의존성 관리

## 생성자 주입
```rust
pub struct UserService<R: UserRepository, E: EmailService> {
    repository: R,
    email_service: E,
}

impl<R: UserRepository, E: EmailService> UserService<R, E> {
    pub fn new(repository: R, email_service: E) -> Self {
        Self { repository, email_service }
    }
    
    pub async fn register(&self, email: String) -> Result<(), Error> {
        let user = User::new(email);
        self.repository.save(&user).await?;
        self.email_service.send_welcome(&user).await?;
        Ok(())
    }
}
```

## 트레이트 객체 DI
```rust
pub struct Application {
    user_service: Box<dyn UserServiceTrait>,
    auth_service: Box<dyn AuthServiceTrait>,
}

impl Application {
    pub fn new(
        user_service: Box<dyn UserServiceTrait>,
        auth_service: Box<dyn AuthServiceTrait>,
    ) -> Self {
        Self { user_service, auth_service }
    }
}
```

## Provider 패턴
```rust
pub trait Provider<T> {
    fn provide(&self) -> T;
}

pub struct DatabaseProvider {
    config: DatabaseConfig,
}

impl Provider<PgPool> for DatabaseProvider {
    fn provide(&self) -> PgPool {
        PgPoolOptions::new()
            .connect_lazy(&self.config.url)
            .expect("Failed to create pool")
    }
}
```

## DI 컨테이너 구현
```rust
pub struct Container {
    providers: HashMap<TypeId, Box<dyn Any>>,
}

impl Container {
    pub fn register<T: 'static>(&mut self, provider: impl Provider<T> + 'static) {
        self.providers.insert(
            TypeId::of::<T>(),
            Box::new(provider),
        );
    }
    
    pub fn resolve<T: 'static>(&self) -> Option<T> {
        self.providers
            .get(&TypeId::of::<T>())
            .and_then(|p| p.downcast_ref::<Box<dyn Provider<T>>>())
            .map(|p| p.provide())
    }
}
```

## 매크로 기반 DI
```rust
use shaku::{Component, Interface, module};

trait MyService: Interface {
    fn do_something(&self);
}

#[derive(Component)]
#[shaku(interface = MyService)]
struct MyServiceImpl {
    #[shaku(inject)]
    database: Arc<dyn Database>,
}

module! {
    MyModule {
        components = [MyServiceImpl],
        providers = []
    }
}
```

## 빌더 패턴과 DI
```rust
pub struct AppBuilder {
    database_url: Option<String>,
    redis_url: Option<String>,
}

impl AppBuilder {
    pub fn with_database(mut self, url: String) -> Self {
        self.database_url = Some(url);
        self
    }
    
    pub fn build(self) -> Result<Application, Error> {
        let db = PgPool::connect(&self.database_url.ok_or(Error::MissingDatabase)?).await?;
        let cache = RedisClient::connect(&self.redis_url.unwrap_or_default());
        
        let user_repo = PostgresUserRepository::new(db.clone());
        let user_service = UserService::new(user_repo);
        
        Ok(Application {
            user_service: Box::new(user_service),
            // ...
        })
    }
}
```

## 라이프타임과 DI
```rust
pub struct ServiceWithLifetime<'a> {
    config: &'a Config,
    logger: &'a Logger,
}

impl<'a> ServiceWithLifetime<'a> {
    pub fn new(config: &'a Config, logger: &'a Logger) -> Self {
        Self { config, logger }
    }
}
```

## 비동기 DI
```rust
#[async_trait]
pub trait AsyncProvider<T> {
    async fn provide(&self) -> Result<T, Error>;
}

pub struct AsyncContainer {
    providers: HashMap<TypeId, Box<dyn Any + Send + Sync>>,
}

impl AsyncContainer {
    pub async fn resolve<T: 'static>(&self) -> Result<T, Error> {
        // 비동기 프로바이더 해결
    }
}
```

## Rust 특성 활용
- **제네릭**: 컴파일 타임 DI
- **트레이트 객체**: 런타임 DI
- **라이프타임**: 참조 기반 DI
- **빌더 패턴**: 유연한 구성

## 장점
- 테스트 용이성
- 모듈화
- 유연한 구성
- 의존성 역전

## 관련 개념
- [[082_hexagonal_architecture]] - 포트와 어댑터
- [[086_test_doubles]] - 테스트 더블
- [[090_service_locator]] - 서비스 로케이터 안티패턴