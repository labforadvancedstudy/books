# Hexagonal Architecture in Rust

## 핵심 개념
포트와 어댑터 패턴으로 도메인 로직과 외부 시스템 분리

## 포트 정의 (트레이트)
```rust
// 도메인이 필요로 하는 포트
pub trait UserRepository {
    async fn find_by_id(&self, id: UserId) -> Result<User, Error>;
    async fn save(&self, user: &User) -> Result<(), Error>;
}

// 도메인이 제공하는 포트
pub trait UserService {
    async fn register(&self, cmd: RegisterCommand) -> Result<UserId, Error>;
    async fn update_profile(&self, cmd: UpdateCommand) -> Result<(), Error>;
}
```

## 어댑터 구현
```rust
// 인바운드 어댑터 (REST API)
pub struct RestAdapter<S: UserService> {
    service: S,
}

impl<S: UserService> RestAdapter<S> {
    pub async fn handle_registration(&self, req: HttpRequest) -> HttpResponse {
        let cmd = parse_command(req);
        match self.service.register(cmd).await {
            Ok(id) => HttpResponse::Created(id),
            Err(e) => HttpResponse::BadRequest(e),
        }
    }
}

// 아웃바운드 어댑터 (PostgreSQL)
pub struct PostgresUserRepository {
    pool: PgPool,
}

impl UserRepository for PostgresUserRepository {
    async fn find_by_id(&self, id: UserId) -> Result<User, Error> {
        sqlx::query_as!(User, "SELECT * FROM users WHERE id = $1", id)
            .fetch_one(&self.pool)
            .await
            .map_err(Into::into)
    }
}
```

## 도메인 서비스 구현
```rust
pub struct UserServiceImpl<R: UserRepository> {
    repository: R,
}

impl<R: UserRepository> UserService for UserServiceImpl<R> {
    async fn register(&self, cmd: RegisterCommand) -> Result<UserId, Error> {
        // 순수 도메인 로직
        let user = User::new(cmd.email, cmd.name)?;
        self.repository.save(&user).await?;
        Ok(user.id())
    }
}
```

## 의존성 주입
```rust
// 애플리케이션 조립
pub fn setup_app() -> impl UserService {
    let repo = PostgresUserRepository::new(config.database_url);
    UserServiceImpl::new(repo)
}
```

## 테스트 용이성
```rust
#[cfg(test)]
mod tests {
    struct MockRepository {
        users: HashMap<UserId, User>,
    }
    
    impl UserRepository for MockRepository {
        // 테스트용 구현
    }
    
    #[tokio::test]
    async fn test_user_registration() {
        let repo = MockRepository::new();
        let service = UserServiceImpl::new(repo);
        // 외부 의존성 없이 테스트
    }
}
```

## Rust 특성 활용
- **트레이트**: 포트 추상화
- **제네릭**: 어댑터 교체 가능
- **async 트레이트**: 비동기 경계
- **타입 시스템**: 의존성 방향 강제

## 장점
- 도메인 로직 격리
- 테스트 용이성
- 기술 스택 독립성
- 명확한 의존성 방향

## 관련 개념
- [[081_ddd_rust]] - 도메인 모델링
- [[085_dependency_injection]] - DI 패턴
- [[086_test_doubles]] - 테스트 더블