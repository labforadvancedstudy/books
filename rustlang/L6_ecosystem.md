# L6: 생태계와 도구들 - Rust 커뮤니티의 보물들

## Core Insight
Rust의 진정한 힘은 언어 자체뿐만 아니라 그 주변 생태계에 있다. Cargo부터 시작해서 수천 개의 고품질 크레이트까지, Rust 생태계는 개발자 경험(DX)의 새로운 기준을 제시한다.

---

## Cargo - 역사상 최고의 패키지 매니저

### Cargo가 특별한 이유

다른 언어들의 고통:
- C/C++: Makefile, CMake, Bazel, ... (표준 없음)
- Python: pip, poetry, pipenv, conda, ... (너무 많음)
- JavaScript: npm, yarn, pnpm, ... (계속 바뀜)

Rust의 해답:
```bash
$ cargo new my_project  # 프로젝트 생성
$ cargo build          # 빌드
$ cargo run           # 실행
$ cargo test          # 테스트
$ cargo doc           # 문서 생성
$ cargo publish       # crates.io에 배포
```

### Cargo.toml의 마법

```toml
[package]
name = "awesome-rust-app"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
edition = "2021"
license = "MIT OR Apache-2.0"
description = "An awesome Rust application"
repository = "https://github.com/you/awesome-rust-app"
keywords = ["awesome", "rust"]
categories = ["command-line-utilities"]

[dependencies]
# 버전 지정의 다양한 방법
serde = "1.0"                              # ^1.0.0 과 같음
tokio = { version = "1", features = ["full"] }
regex = "=1.7.0"                          # 정확히 1.7.0
clap = ">=4.0, <5.0"                      # 범위 지정

# Git 저장소에서 직접
my_lib = { git = "https://github.com/user/repo", branch = "main" }

# 로컬 경로
local_lib = { path = "../local_lib" }

# 선택적 의존성
[dependencies.uuid]
version = "1.0"
features = ["v4", "serde"]
optional = true

[features]
default = ["tokio/rt-multi-thread"]
experimental = ["uuid"]

[dev-dependencies]
criterion = "0.5"  # 벤치마킹용

[build-dependencies]
cc = "1.0"  # build.rs에서 사용

[profile.release]
opt-level = 3     # 최대 최적화
lto = true        # Link Time Optimization
codegen-units = 1 # 단일 코드젠 유닛
```

### Workspace 관리

```toml
# 루트 Cargo.toml
[workspace]
members = [
    "core",
    "cli",
    "server",
    "shared",
]

[workspace.package]
version = "0.1.0"
edition = "2021"
authors = ["Team Name"]

[workspace.dependencies]
serde = "1.0"
tokio = "1.0"
```

개별 프로젝트:
```toml
# cli/Cargo.toml
[package]
name = "my-cli"
version.workspace = true
edition.workspace = true

[dependencies]
serde.workspace = true
shared = { path = "../shared" }
```

## rustfmt - 코드 포맷터

### 설정 파일 (rustfmt.toml)

```toml
# 기본 설정
edition = "2021"
max_width = 100
tab_spaces = 4

# 임포트 정리
imports_granularity = "Crate"
group_imports = "StdExternalCrate"

# 기타 스타일
use_field_init_shorthand = true
use_try_shorthand = true
format_code_in_doc_comments = true
wrap_comments = true
format_strings = true
```

사용법:
```bash
$ rustfmt src/main.rs        # 단일 파일
$ cargo fmt                   # 전체 프로젝트
$ cargo fmt -- --check       # CI에서 체크만
```

## Clippy - 린터의 왕

### 수준별 경고

```rust
#![warn(clippy::all)]
#![warn(clippy::pedantic)]
#![warn(clippy::nursery)]
#![deny(clippy::correctness)]

// 특정 린트 비활성화
#[allow(clippy::too_many_arguments)]
fn complex_function(a: i32, b: i32, c: i32, d: i32, e: i32, f: i32) {
    // ...
}

// 더 나은 코드 제안
// Clippy: use `if let` instead of `match` with single arm
match option {
    Some(value) => println!("{}", value),
    _ => {}
}

// Clippy 제안:
if let Some(value) = option {
    println!("{}", value);
}
```

커스텀 린트 레벨:
```toml
# clippy.toml
msrv = "1.70.0"
warn-on-all-wildcard-imports = true
allowed-duplicate-crates = ["hashbrown", "regex"]
```

## 필수 크레이트들

### 직렬화 - Serde

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
#[serde(rename_all = "camelCase")]
struct User {
    #[serde(rename = "userId")]
    id: u64,
    
    user_name: String,
    
    #[serde(skip_serializing_if = "Option::is_none")]
    email: Option<String>,
    
    #[serde(default)]
    active: bool,
    
    #[serde(with = "chrono::serde::ts_seconds")]
    created_at: DateTime<Utc>,
}

// JSON 변환
let user = User { /* ... */ };
let json = serde_json::to_string(&user)?;
let user2: User = serde_json::from_str(&json)?;

// YAML, TOML, MessagePack 등도 지원
let yaml = serde_yaml::to_string(&user)?;
let toml = toml::to_string(&user)?;
```

### 비동기 런타임 - Tokio

```rust
use tokio::time::{sleep, Duration};
use tokio::task;
use tokio::sync::mpsc;

#[tokio::main]
async fn main() {
    // 비동기 태스크 스폰
    let handle = task::spawn(async {
        sleep(Duration::from_secs(1)).await;
        "작업 완료"
    });
    
    // 채널
    let (tx, mut rx) = mpsc::channel(32);
    
    task::spawn(async move {
        tx.send("메시지").await.unwrap();
    });
    
    // 동시 실행
    let (result, msg) = tokio::join!(
        handle,
        rx.recv()
    );
    
    println!("{:?}, {:?}", result, msg);
}
```

### 웹 프레임워크 - Axum

```rust
use axum::{
    routing::{get, post},
    extract::{Query, Json, Path, State},
    middleware,
    Router,
};

async fn handler(
    Path(id): Path<u32>,
    Query(params): Query<HashMap<String, String>>,
    State(state): State<AppState>,
    Json(payload): Json<Value>,
) -> impl IntoResponse {
    // ...
}

let app = Router::new()
    .route("/users/:id", get(handler))
    .layer(middleware::from_fn(auth_middleware))
    .with_state(app_state);
```

### 로깅 - tracing

```rust
use tracing::{info, warn, error, debug, instrument};

#[instrument]
async fn process_request(id: u64, data: &str) -> Result<()> {
    info!(?id, "요청 처리 시작");
    
    debug!("데이터 파싱");
    let parsed = parse_data(data)?;
    
    info!(
        parsed_size = parsed.len(),
        "파싱 완료"
    );
    
    if parsed.is_empty() {
        warn!("빈 데이터");
    }
    
    Ok(())
}

// 구조화된 로깅
#[derive(Debug)]
struct SpanData {
    request_id: String,
    user_id: u64,
}

let span = tracing::info_span!(
    "http_request",
    request_id = %data.request_id,
    user_id = data.user_id
);

let _guard = span.enter();
```

## 테스팅 도구들

### 단위 테스트

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use pretty_assertions::assert_eq;  // 더 나은 diff
    
    #[test]
    fn test_addition() {
        assert_eq!(2 + 2, 4);
    }
    
    #[test]
    #[should_panic(expected = "divide by zero")]
    fn test_panic() {
        divide(10, 0);
    }
    
    #[test]
    #[ignore]  // cargo test -- --ignored 로 실행
    fn expensive_test() {
        // ...
    }
}
```

### 속성 기반 테스팅 - proptest

```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn test_reversing_twice_is_identity(s: String) {
        let reversed = reverse(&s);
        let double_reversed = reverse(&reversed);
        prop_assert_eq!(s, double_reversed);
    }
    
    #[test]
    fn test_parsing_printed_value(n: i32) {
        let printed = n.to_string();
        let parsed: i32 = printed.parse().unwrap();
        prop_assert_eq!(n, parsed);
    }
}
```

### 벤치마킹 - Criterion

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn fibonacci(n: u64) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        n => fibonacci(n-1) + fibonacci(n-2),
    }
}

fn benchmark_fibonacci(c: &mut Criterion) {
    c.bench_function("fibonacci 20", |b| {
        b.iter(|| fibonacci(black_box(20)))
    });
}

criterion_group!(benches, benchmark_fibonacci);
criterion_main!(benches);
```

## 문서화

### rustdoc

```rust
/// 두 수를 더합니다.
/// 
/// # Examples
/// 
/// ```
/// let result = my_crate::add(2, 3);
/// assert_eq!(result, 5);
/// ```
/// 
/// # Panics
/// 
/// 오버플로우 시 패닉
/// 
/// # Errors
/// 
/// 이 함수는 에러를 반환하지 않습니다.
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

/// 복잡한 구조체
/// 
/// ## 필드
/// 
/// * `name` - 사용자 이름
/// * `age` - 나이 (0-150)
#[derive(Debug)]
pub struct User {
    /// 사용자의 고유 식별자
    pub id: u64,
    
    #[doc(hidden)]  // 문서에서 숨김
    pub internal_field: String,
}
```

문서 생성:
```bash
$ cargo doc --open
$ cargo doc --no-deps  # 의존성 제외
```

## 크로스 컴파일

```bash
# 타겟 추가
$ rustup target add wasm32-unknown-unknown
$ rustup target add x86_64-pc-windows-gnu
$ rustup target add aarch64-apple-darwin

# 크로스 컴파일
$ cargo build --target wasm32-unknown-unknown
$ cargo build --target x86_64-pc-windows-gnu
```

Cross 도구 사용:
```bash
$ cargo install cross
$ cross build --target aarch64-unknown-linux-gnu
```

## 개발 도구

### rust-analyzer

VSCode settings.json:
```json
{
    "rust-analyzer.cargo.features": ["all"],
    "rust-analyzer.checkOnSave.command": "clippy",
    "rust-analyzer.inlayHints.enabled": true,
    "rust-analyzer.proc-macro.enabled": true,
    "rust-analyzer.lens.implementations": true
}
```

### cargo-watch

```bash
$ cargo install cargo-watch

# 파일 변경 시 자동 실행
$ cargo watch -x run
$ cargo watch -x test
$ cargo watch -x "check --all-features"
```

### cargo-expand

```bash
$ cargo install cargo-expand

# 매크로 확장 결과 보기
$ cargo expand
```

## 보안 도구

### cargo-audit

```bash
$ cargo install cargo-audit
$ cargo audit  # 보안 취약점 검사
```

### cargo-deny

```toml
# deny.toml
[bans]
multiple-versions = "warn"
wildcards = "deny"

[licenses]
unlicensed = "deny"
allow = ["MIT", "Apache-2.0"]
```

## 프로파일링

### flamegraph

```bash
$ cargo install flamegraph
$ cargo flamegraph
```

### cargo-bloat

```bash
$ cargo install cargo-bloat
$ cargo bloat --release
```

## CI/CD 설정

### GitHub Actions

```yaml
name: Rust CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - uses: actions-rs/toolchain@v1
      with:
        toolchain: stable
        override: true
        components: rustfmt, clippy
    
    - name: Format check
      run: cargo fmt -- --check
    
    - name: Clippy
      run: cargo clippy -- -D warnings
    
    - name: Test
      run: cargo test --all-features
    
    - name: Build
      run: cargo build --release
```

## 마무리: 생태계의 철학

Rust 생태계의 특징:
1. **일관성**: 모든 도구가 Cargo를 중심으로
2. **품질**: 크레이트들의 높은 완성도
3. **문서화**: 거의 모든 크레이트가 잘 문서화됨
4. **커뮤니티**: 친절하고 포용적인 문화

이 모든 것이 "개발자 경험"을 극대화한다.

---

## Connections
→ [[L7_advanced_patterns]] - 고급 패턴으로
→ [[zettel/093_cargo_workspaces]] - Workspace 심화
→ [[zettel/096_feature_flags]] - Feature 플래그 활용
← [[L5_real_projects]] - 실전 프로젝트

---

Level: L6 (도구와 생태계)
Date: 2025-08-15
Tags: #rust #cargo #ecosystem #tools #crates