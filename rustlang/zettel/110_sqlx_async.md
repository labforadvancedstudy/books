# SQLx Async Database Access

## 핵심 개념
컴파일 타임 검증과 비동기 지원을 제공하는 SQL 툴킷

## 기본 설정
```toml
[dependencies]
sqlx = { version = "0.8", features = ["runtime-tokio", "postgres", "chrono", "uuid"] }
tokio = { version = "1", features = ["full"] }

# .env
DATABASE_URL=postgres://user:password@localhost/mydb
```

## 연결 풀
```rust
use sqlx::postgres::PgPoolOptions;
use sqlx::Pool;

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    let pool = PgPoolOptions::new()
        .max_connections(5)
        .min_connections(1)
        .acquire_timeout(Duration::from_secs(3))
        .idle_timeout(Duration::from_secs(10))
        .max_lifetime(Duration::from_secs(60))
        .connect(&env::var("DATABASE_URL")?).await?;
    
    // 연결 상태 확인
    pool.acquire().await?.ping().await?;
    
    Ok(())
}
```

## 컴파일 타임 쿼리 검증
```rust
use sqlx::query;

// 컴파일 시점에 SQL 검증
async fn get_user(pool: &PgPool, id: i32) -> Result<User, sqlx::Error> {
    let user = sqlx::query_as!(
        User,
        r#"
        SELECT id, email, name, created_at
        FROM users
        WHERE id = $1
        "#,
        id
    )
    .fetch_one(pool)
    .await?;
    
    Ok(user)
}
```

## 타입 안전 쿼리
```rust
#[derive(sqlx::FromRow)]
struct User {
    id: i32,
    email: String,
    name: Option<String>,
    created_at: chrono::DateTime<chrono::Utc>,
}

async fn find_users(pool: &PgPool) -> Result<Vec<User>, sqlx::Error> {
    let users = sqlx::query_as::<_, User>(
        "SELECT * FROM users WHERE active = true"
    )
    .fetch_all(pool)
    .await?;
    
    Ok(users)
}
```

## 트랜잭션
```rust
use sqlx::{Transaction, Postgres};

async fn transfer_funds(
    pool: &PgPool,
    from: i32,
    to: i32,
    amount: Decimal,
) -> Result<(), sqlx::Error> {
    let mut tx = pool.begin().await?;
    
    // 출금
    sqlx::query!(
        "UPDATE accounts SET balance = balance - $1 WHERE id = $2",
        amount,
        from
    )
    .execute(&mut *tx)
    .await?;
    
    // 입금
    sqlx::query!(
        "UPDATE accounts SET balance = balance + $1 WHERE id = $2",
        amount,
        to
    )
    .execute(&mut *tx)
    .await?;
    
    // 커밋
    tx.commit().await?;
    
    Ok(())
}
```

## 스트리밍 결과
```rust
use futures::TryStreamExt;

async fn process_large_dataset(pool: &PgPool) -> Result<(), sqlx::Error> {
    let mut stream = sqlx::query_as::<_, User>(
        "SELECT * FROM users ORDER BY id"
    )
    .fetch(pool);
    
    while let Some(user) = stream.try_next().await? {
        // 한 번에 하나씩 처리
        process_user(user).await;
    }
    
    Ok(())
}
```

## Prepared Statements
```rust
use sqlx::query::Query;

async fn batch_insert(pool: &PgPool, users: Vec<NewUser>) -> Result<(), sqlx::Error> {
    let mut tx = pool.begin().await?;
    
    // Prepared statement 재사용
    for user in users {
        sqlx::query!(
            "INSERT INTO users (email, name) VALUES ($1, $2)",
            user.email,
            user.name
        )
        .execute(&mut *tx)
        .await?;
    }
    
    tx.commit().await?;
    Ok(())
}
```

## 마이그레이션
```rust
// migrations/001_create_users.sql
-- migrate:up
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- migrate:down
DROP TABLE users;

// main.rs
sqlx::migrate!("./migrations")
    .run(&pool)
    .await?;
```

## 커스텀 타입
```rust
use sqlx::Type;

#[derive(Debug, Type)]
#[sqlx(type_name = "user_role")]
#[sqlx(rename_all = "lowercase")]
enum UserRole {
    Admin,
    User,
    Guest,
}

#[derive(FromRow)]
struct UserWithRole {
    id: i32,
    email: String,
    role: UserRole,
}
```

## 비동기 연결 관리
```rust
use sqlx::pool::PoolConnection;

async fn with_connection<F, T>(pool: &PgPool, f: F) -> Result<T, sqlx::Error>
where
    F: FnOnce(PoolConnection<Postgres>) -> T,
    T: Future<Output = Result<(), sqlx::Error>>,
{
    let conn = pool.acquire().await?;
    f(conn).await
}
```

## JSON 지원
```rust
use serde::{Deserialize, Serialize};
use sqlx::types::Json;

#[derive(Serialize, Deserialize)]
struct Metadata {
    tags: Vec<String>,
    version: i32,
}

async fn save_with_json(pool: &PgPool, metadata: Metadata) -> Result<(), sqlx::Error> {
    sqlx::query!(
        "INSERT INTO documents (metadata) VALUES ($1)",
        Json(metadata) as _
    )
    .execute(pool)
    .await?;
    
    Ok(())
}
```

## 에러 처리
```rust
use sqlx::error::ErrorKind;

async fn handle_db_error(pool: &PgPool) {
    match get_user(pool, 1).await {
        Ok(user) => println!("Found: {}", user.email),
        Err(sqlx::Error::RowNotFound) => {
            println!("User not found");
        }
        Err(e) if matches!(e.kind(), ErrorKind::UniqueViolation) => {
            println!("Duplicate entry");
        }
        Err(e) => eprintln!("Database error: {}", e),
    }
}
```

## Rust 특성 활용
- **컴파일 타임 검증**: SQL 쿼리 타입 체크
- **비동기**: 논블로킹 I/O
- **타입 안전**: 자동 타입 매핑
- **제로 비용**: 최소 런타임 오버헤드

## 성능 팁
- 연결 풀 크기 조정
- Prepared statements 활용
- 배치 처리
- 인덱스 최적화
- 스트리밍으로 메모리 절약

## 관련 개념
- [[083_repository_pattern]] - 리포지토리 패턴
- [[111_connection_pooling]] - 연결 풀링
- [[112_database_migrations]] - 마이그레이션