# Repository Pattern in Rust

## 핵심 개념
도메인 객체와 데이터 접근 로직을 분리하는 패턴

## 기본 트레이트 정의
```rust
pub trait Repository<T, ID> {
    type Error;
    
    async fn find(&self, id: ID) -> Result<Option<T>, Self::Error>;
    async fn find_all(&self) -> Result<Vec<T>, Self::Error>;
    async fn save(&self, entity: &T) -> Result<(), Self::Error>;
    async fn delete(&self, id: ID) -> Result<(), Self::Error>;
}
```

## 특화된 리포지토리
```rust
pub trait UserRepository: Repository<User, UserId> {
    async fn find_by_email(&self, email: &str) -> Result<Option<User>, Self::Error>;
    async fn find_active_users(&self) -> Result<Vec<User>, Self::Error>;
}
```

## 구현체 예시
```rust
pub struct SqlUserRepository {
    pool: SqlitePool,
}

impl Repository<User, UserId> for SqlUserRepository {
    type Error = sqlx::Error;
    
    async fn find(&self, id: UserId) -> Result<Option<User>, Self::Error> {
        sqlx::query_as!(
            User,
            "SELECT * FROM users WHERE id = ?",
            id.as_str()
        )
        .fetch_optional(&self.pool)
        .await
    }
    
    async fn save(&self, user: &User) -> Result<(), Self::Error> {
        sqlx::query!(
            "INSERT OR REPLACE INTO users (id, email, name) VALUES (?, ?, ?)",
            user.id.as_str(),
            user.email,
            user.name
        )
        .execute(&self.pool)
        .await?;
        Ok(())
    }
}
```

## 메모리 구현 (테스트용)
```rust
pub struct InMemoryRepository<T, ID> {
    data: Arc<RwLock<HashMap<ID, T>>>,
}

impl<T: Clone, ID: Hash + Eq + Clone> Repository<T, ID> for InMemoryRepository<T, ID> {
    type Error = Infallible;
    
    async fn find(&self, id: ID) -> Result<Option<T>, Self::Error> {
        Ok(self.data.read().await.get(&id).cloned())
    }
    
    async fn save(&self, entity: &T) -> Result<(), Self::Error> {
        self.data.write().await.insert(id, entity.clone());
        Ok(())
    }
}
```

## Unit of Work 패턴
```rust
pub struct UnitOfWork {
    connection: SqliteConnection,
    changes: Vec<Change>,
}

impl UnitOfWork {
    pub async fn begin() -> Result<Self, Error> {
        let mut conn = get_connection().await?;
        conn.begin().await?;
        Ok(Self { 
            connection: conn,
            changes: vec![]
        })
    }
    
    pub fn register_new<T>(&mut self, entity: T) {
        self.changes.push(Change::Insert(entity));
    }
    
    pub async fn commit(mut self) -> Result<(), Error> {
        for change in self.changes {
            // 변경사항 적용
        }
        self.connection.commit().await?;
        Ok(())
    }
}
```

## 스펙 패턴 결합
```rust
pub trait Specification<T> {
    fn is_satisfied_by(&self, entity: &T) -> bool;
}

impl<T> Repository<T> {
    async fn find_by_spec(&self, spec: impl Specification<T>) -> Vec<T> {
        self.find_all()
            .await?
            .into_iter()
            .filter(|e| spec.is_satisfied_by(e))
            .collect()
    }
}
```

## Rust 특성 활용
- **async/await**: 비동기 데이터 접근
- **제네릭**: 재사용 가능한 리포지토리
- **Associated Types**: 에러 타입 유연성
- **트레이트 객체**: 런타임 다형성

## 장점
- 데이터 접근 로직 캡슐화
- 테스트 용이성 (목 객체 쉽게 생성)
- 데이터베이스 독립성
- 쿼리 로직 중앙화

## 관련 개념
- [[081_ddd_rust]] - 도메인 주도 설계
- [[084_cqrs_rust]] - 명령과 조회 분리
- [[087_sqlx_async]] - 비동기 SQL