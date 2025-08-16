# Feature Flags in Rust

## 핵심 개념
컴파일 타임 기능 토글로 조건부 컴파일 및 의존성 관리

## 기본 정의
```toml
[features]
default = ["tokio", "logging"]
tokio = ["dep:tokio", "async"]
async = []
logging = ["tracing", "tracing-subscriber"]
full = ["tokio", "async", "logging", "experimental"]
experimental = []

[dependencies]
tokio = { version = "1.40", optional = true }
tracing = { version = "0.1", optional = true }
tracing-subscriber = { version = "0.3", optional = true }
```

## 코드에서 사용
```rust
#[cfg(feature = "async")]
pub async fn async_function() {
    // 비동기 코드
}

#[cfg(not(feature = "async"))]
pub fn sync_function() {
    // 동기 코드
}

#[cfg(all(feature = "tokio", feature = "logging"))]
mod advanced_features {
    use tracing::info;
    
    pub async fn logged_operation() {
        info!("Starting operation");
        // ...
    }
}
```

## 조건부 모듈
```rust
#[cfg(feature = "postgres")]
mod postgres_backend;

#[cfg(feature = "sqlite")]
mod sqlite_backend;

pub trait Backend {
    fn query(&self, sql: &str) -> Result<Vec<Row>>;
}

#[cfg(feature = "postgres")]
pub use postgres_backend::PostgresBackend as DefaultBackend;

#[cfg(all(not(feature = "postgres"), feature = "sqlite"))]
pub use sqlite_backend::SqliteBackend as DefaultBackend;
```

## 기능 게이트
```rust
/// This function requires the "experimental" feature
#[cfg_attr(
    not(feature = "experimental"),
    deprecated(note = "Enable 'experimental' feature")
)]
pub fn experimental_api() {
    #[cfg(not(feature = "experimental"))]
    panic!("experimental feature not enabled");
    
    #[cfg(feature = "experimental")]
    {
        // 실험적 구현
    }
}
```

## 의존성 기능 전파
```toml
# 라이브러리 Cargo.toml
[features]
default = []
json = ["serde", "serde_json"]
yaml = ["serde", "serde_yaml"]

[dependencies]
serde = { version = "1.0", optional = true, features = ["derive"] }
serde_json = { version = "1.0", optional = true }
serde_yaml = { version = "0.9", optional = true }

# 사용자 Cargo.toml
[dependencies]
my-lib = { version = "0.1", features = ["json"] }
```

## 상호 배타적 기능
```rust
#[cfg(all(feature = "backend-postgres", feature = "backend-sqlite"))]
compile_error!("Cannot enable both postgres and sqlite backends");

pub enum Backend {
    #[cfg(feature = "backend-postgres")]
    Postgres(PostgresBackend),
    
    #[cfg(feature = "backend-sqlite")]
    Sqlite(SqliteBackend),
}
```

## 테스트와 기능 플래그
```rust
#[cfg(test)]
mod tests {
    #[test]
    #[cfg(feature = "tokio")]
    async fn test_async_feature() {
        // async 기능 테스트
    }
    
    #[test]
    #[cfg(not(feature = "tokio"))]
    fn test_sync_fallback() {
        // sync 폴백 테스트
    }
}

// 테스트 실행
// cargo test --features tokio
// cargo test --all-features
```

## 런타임 기능 플래그
```rust
use once_cell::sync::Lazy;

static FEATURES: Lazy<Features> = Lazy::new(|| {
    Features {
        experimental: env::var("ENABLE_EXPERIMENTAL").is_ok(),
        debug_mode: cfg!(debug_assertions),
        ab_test_enabled: rand::random::<f32>() < 0.5,
    }
});

pub fn is_feature_enabled(name: &str) -> bool {
    match name {
        "experimental" => FEATURES.experimental,
        "debug" => FEATURES.debug_mode,
        _ => false,
    }
}
```

## 문서화
```rust
/// Main client struct
/// 
/// # Features
/// 
/// - `async`: Enables async methods
/// - `blocking`: Enables blocking methods
/// - `rustls`: Use rustls for TLS (default)
/// - `native-tls`: Use native TLS implementation
#[cfg_attr(docsrs, doc(cfg(feature = "async")))]
pub struct Client {
    #[cfg(feature = "async")]
    runtime: tokio::runtime::Runtime,
}
```

## CI/CD에서 기능 테스트
```yaml
# .github/workflows/test.yml
strategy:
  matrix:
    features:
      - ""  # default features
      - "--all-features"
      - "--no-default-features"
      - "--features async"
      - "--features tokio,logging"

steps:
  - run: cargo test ${{ matrix.features }}
```

## Rust 특성 활용
- **컴파일 타임 결정**: 런타임 오버헤드 없음
- **타입 안전**: 기능별 API 분리
- **의존성 최소화**: 필요한 것만 컴파일
- **문서 통합**: 기능별 문서 자동 생성

## 모범 사례
- 기본 기능은 최소화
- 상호 배타적 기능 명확히 표시
- 기능 조합 테스트
- 문서에 기능 요구사항 명시

## 관련 개념
- [[095_build_scripts]] - 빌드 스크립트
- [[097_cross_compilation]] - 크로스 컴파일
- [[098_conditional_compilation]] - 조건부 컴파일