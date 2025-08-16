# L8: 프로덕션 배포 - 실전에서 살아남기

## Core Insight
프로덕션은 개발 환경과 다른 세계다. 성능 프로파일링, 메모리 최적화, 크로스 컴파일, CI/CD 파이프라인 - 이 모든 것이 실제 사용자를 만나기 위한 준비다.

---

## 성능 프로파일링

### CPU 프로파일링

```bash
# perf 사용 (Linux)
$ cargo build --release
$ perf record -g ./target/release/myapp
$ perf report

# Instruments 사용 (macOS)
$ cargo instruments -t "Time Profiler" --release

# flamegraph 생성
$ cargo install flamegraph
$ cargo flamegraph --release
```

### 실제 병목 찾기

```rust
use std::time::Instant;

#[derive(Default)]
struct Profiler {
    timings: HashMap<String, Duration>,
}

impl Profiler {
    fn measure<F, R>(&mut self, name: &str, f: F) -> R 
    where 
        F: FnOnce() -> R
    {
        let start = Instant::now();
        let result = f();
        let duration = start.elapsed();
        
        *self.timings.entry(name.to_string()).or_default() += duration;
        
        result
    }
    
    fn report(&self) {
        let total: Duration = self.timings.values().sum();
        
        for (name, duration) in &self.timings {
            let percentage = (duration.as_secs_f64() / total.as_secs_f64()) * 100.0;
            println!("{}: {:?} ({:.2}%)", name, duration, percentage);
        }
    }
}

// 사용
let mut profiler = Profiler::default();

let data = profiler.measure("parse", || parse_input(&input));
let result = profiler.measure("process", || process_data(&data));
profiler.measure("save", || save_result(&result));

profiler.report();
```

### 메모리 프로파일링

```rust
// jemalloc과 통계
#[global_allocator]
static ALLOC: jemallocator::Jemalloc = jemallocator::Jemalloc;

// 메모리 추적
use peak_alloc::PeakAlloc;

#[global_allocator]
static PEAK_ALLOC: PeakAlloc = PeakAlloc;

fn main() {
    // 작업 수행
    
    let peak_mem = PEAK_ALLOC.peak_usage_as_gb();
    println!("Peak memory: {} GB", peak_mem);
}
```

## 메모리 최적화

### String과 &str 최적화

```rust
// 나쁨: 불필요한 할당
fn process_bad(data: Vec<String>) {
    for s in data {
        if s.starts_with("prefix") {
            // String 소유권 이동
        }
    }
}

// 좋음: 참조 사용
fn process_good(data: &[&str]) {
    for &s in data {
        if s.starts_with("prefix") {
            // 참조만 사용
        }
    }
}

// Cow로 유연하게
use std::borrow::Cow;

fn process_cow(input: &str) -> Cow<str> {
    if input.contains("old") {
        Cow::Owned(input.replace("old", "new"))
    } else {
        Cow::Borrowed(input)  // 할당 없음!
    }
}
```

### SmallVec 활용

```rust
use smallvec::SmallVec;

// 스택에 최대 4개, 그 이상은 힙
type SmallBuffer = SmallVec<[u8; 4]>;

fn process() {
    let mut buffer: SmallBuffer = SmallVec::new();
    
    // 작은 경우 스택 사용 (할당 없음)
    buffer.push(1);
    buffer.push(2);
    
    // 큰 경우에만 힙 할당
    for i in 3..100 {
        buffer.push(i);
    }
}
```

### 아레나 할당자

```rust
use typed_arena::Arena;

struct Node<'a> {
    value: i32,
    children: Vec<&'a Node<'a>>,
}

fn build_tree() {
    let arena = Arena::new();
    
    // 모든 노드를 아레나에 할당
    let root = arena.alloc(Node {
        value: 1,
        children: vec![],
    });
    
    let child1 = arena.alloc(Node {
        value: 2,
        children: vec![],
    });
    
    // 아레나가 drop될 때 한 번에 해제
}
```

## 크로스 컴파일

### Docker 멀티스테이지 빌드

```dockerfile
# Build stage
FROM rust:1.75 as builder

WORKDIR /app
COPY Cargo.toml Cargo.lock ./
COPY src ./src

# 의존성 캐싱
RUN cargo build --release --locked

# Runtime stage
FROM debian:bookworm-slim

RUN apt-get update && apt-get install -y \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

COPY --from=builder /app/target/release/myapp /usr/local/bin/

EXPOSE 8080
CMD ["myapp"]
```

### 정적 링킹 (musl)

```bash
# musl 타겟 추가
$ rustup target add x86_64-unknown-linux-musl

# 정적 바이너리 빌드
$ cargo build --release --target x86_64-unknown-linux-musl

# 확인
$ ldd target/x86_64-unknown-linux-musl/release/myapp
    not a dynamic executable  # 성공!
```

### 크기 최적화

```toml
# Cargo.toml
[profile.release]
opt-level = "z"     # 크기 최적화
lto = true          # Link Time Optimization
codegen-units = 1  # 단일 코드젠 유닛
strip = true        # 심볼 제거
panic = "abort"     # 패닉 핸들러 제거

# 추가 최적화
[profile.release.package."*"]
opt-level = "z"
```

바이너리 크기 분석:
```bash
$ cargo install cargo-bloat
$ cargo bloat --release --crates
```

## CI/CD 파이프라인

### GitHub Actions 전체 설정

```yaml
name: Production Pipeline

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:

env:
  CARGO_TERM_COLOR: always
  RUST_BACKTRACE: 1

jobs:
  test:
    name: Test Suite
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        rust: [stable, beta]
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Install Rust
      uses: dtolnay/rust-toolchain@master
      with:
        toolchain: ${{ matrix.rust }}
        components: rustfmt, clippy
    
    - name: Cache
      uses: Swatinem/rust-cache@v2
    
    - name: Format check
      run: cargo fmt --all -- --check
    
    - name: Clippy
      run: cargo clippy --all-targets --all-features -- -D warnings
    
    - name: Test
      run: cargo test --all-features
    
    - name: Doc tests
      run: cargo test --doc

  coverage:
    name: Code Coverage
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Install tarpaulin
      run: cargo install cargo-tarpaulin
    
    - name: Generate coverage
      run: cargo tarpaulin --out Xml
    
    - name: Upload to codecov
      uses: codecov/codecov-action@v3

  security:
    name: Security Audit
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: actions-rs/audit-check@v1
      with:
        token: ${{ secrets.GITHUB_TOKEN }}

  deploy:
    name: Deploy
    needs: [test, coverage, security]
    if: startsWith(github.ref, 'refs/tags/')
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Build release
      run: cargo build --release
    
    - name: Build Docker image
      run: |
        docker build -t myapp:${{ github.ref_name }} .
        docker tag myapp:${{ github.ref_name }} myapp:latest
    
    - name: Push to registry
      run: |
        echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
        docker push myapp:${{ github.ref_name }}
        docker push myapp:latest
```

## 로깅과 모니터링

### 구조화된 로깅

```rust
use tracing::{info, warn, error};
use tracing_subscriber;

fn setup_logging() {
    let subscriber = tracing_subscriber::fmt()
        .json()  // JSON 포맷
        .with_target(true)
        .with_thread_ids(true)
        .with_thread_names(true)
        .with_file(true)
        .with_line_number(true)
        .with_env_filter(
            EnvFilter::from_default_env()
                .add_directive("myapp=debug".parse().unwrap())
        )
        .init();
}

#[tracing::instrument(skip(sensitive_data))]
async fn process_request(
    request_id: &str,
    user_id: u64,
    sensitive_data: &str,
) -> Result<Response> {
    info!(
        request_id = %request_id,
        user_id = user_id,
        "Processing request"
    );
    
    // 작업 수행
    
    Ok(response)
}
```

### 메트릭 수집

```rust
use prometheus::{Encoder, TextEncoder, Counter, Histogram, register_counter, register_histogram};
use lazy_static::lazy_static;

lazy_static! {
    static ref REQUEST_COUNTER: Counter = register_counter!(
        "http_requests_total",
        "Total number of HTTP requests"
    ).unwrap();
    
    static ref REQUEST_DURATION: Histogram = register_histogram!(
        "http_request_duration_seconds",
        "HTTP request duration in seconds"
    ).unwrap();
}

async fn metrics_handler() -> String {
    let encoder = TextEncoder::new();
    let metric_families = prometheus::gather();
    let mut buffer = vec![];
    encoder.encode(&metric_families, &mut buffer).unwrap();
    String::from_utf8(buffer).unwrap()
}
```

## 그레이스풀 셧다운

```rust
use tokio::signal;
use tokio::sync::broadcast;

async fn graceful_shutdown(
    shutdown_tx: broadcast::Sender<()>,
) {
    let ctrl_c = async {
        signal::ctrl_c()
            .await
            .expect("failed to install Ctrl+C handler");
    };
    
    #[cfg(unix)]
    let terminate = async {
        signal::unix::signal(signal::unix::SignalKind::terminate())
            .expect("failed to install signal handler")
            .recv()
            .await;
    };
    
    #[cfg(not(unix))]
    let terminate = std::future::pending::<()>();
    
    tokio::select! {
        _ = ctrl_c => {},
        _ = terminate => {},
    }
    
    info!("Shutdown signal received, starting graceful shutdown");
    let _ = shutdown_tx.send(());
}

async fn server_task(mut shutdown_rx: broadcast::Receiver<()>) {
    loop {
        tokio::select! {
            _ = do_work() => {
                // 작업 계속
            }
            _ = shutdown_rx.recv() => {
                info!("Shutting down server task");
                break;
            }
        }
    }
    
    // 정리 작업
    cleanup().await;
}
```

## 환경 설정 관리

```rust
use config::{Config, ConfigError, Environment, File};
use serde::Deserialize;

#[derive(Debug, Deserialize)]
struct Settings {
    database: DatabaseConfig,
    server: ServerConfig,
    logging: LoggingConfig,
}

#[derive(Debug, Deserialize)]
struct DatabaseConfig {
    url: String,
    max_connections: u32,
    timeout: u64,
}

impl Settings {
    fn new() -> Result<Self, ConfigError> {
        let env = std::env::var("RUN_ENV").unwrap_or_else(|_| "development".into());
        
        Config::builder()
            // 기본 설정
            .add_source(File::with_name("config/default"))
            // 환경별 설정
            .add_source(File::with_name(&format!("config/{}", env)).required(false))
            // 환경 변수 (APP_ 접두사)
            .add_source(Environment::with_prefix("APP").separator("_"))
            .build()?
            .try_deserialize()
    }
}
```

## 헬스 체크

```rust
#[derive(Serialize)]
struct Health {
    status: String,
    version: String,
    uptime: Duration,
    checks: HashMap<String, CheckResult>,
}

async fn health_check(State(state): State<AppState>) -> Json<Health> {
    let mut checks = HashMap::new();
    
    // 데이터베이스 체크
    checks.insert("database", check_database(&state.db).await);
    
    // Redis 체크
    checks.insert("redis", check_redis(&state.redis).await);
    
    // 디스크 공간 체크
    checks.insert("disk", check_disk_space());
    
    let status = if checks.values().all(|c| c.healthy) {
        "healthy"
    } else {
        "unhealthy"
    };
    
    Json(Health {
        status: status.to_string(),
        version: env!("CARGO_PKG_VERSION").to_string(),
        uptime: state.start_time.elapsed(),
        checks,
    })
}
```

## 배포 전략

### Blue-Green 배포

```yaml
# kubernetes deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: myapp
        image: myapp:v2.0.0
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
```

## 마무리: 프로덕션의 교훈

프로덕션 준비의 핵심:
1. **측정할 수 없으면 개선할 수 없다**: 프로파일링과 메트릭
2. **실패를 가정하라**: 그레이스풀 셧다운과 에러 처리
3. **자동화하라**: CI/CD로 인간 실수 제거
4. **관찰 가능하게**: 로깅과 모니터링으로 투명성 확보

프로덕션은 코드가 진짜 가치를 만드는 곳이다.

---

## Connections
→ [[L9_philosophy]] - Rust의 철학과 미래
→ [[zettel/099_profiling_tools]] - 프로파일링 도구 상세
→ [[zettel/117_graceful_shutdown]] - 그레이스풀 셧다운 패턴
← [[L7_advanced_patterns]] - 고급 패턴

---

Level: L8 (프로덕션 운영)
Date: 2025-08-15
Tags: #rust #production #deployment #monitoring #optimization