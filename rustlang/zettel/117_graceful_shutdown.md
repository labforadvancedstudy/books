# Graceful Shutdown in Rust

## 핵심 개념
시그널을 받았을 때 진행 중인 작업을 안전하게 완료하고 종료

## 기본 시그널 처리
```rust
use tokio::signal;

#[tokio::main]
async fn main() {
    // SIGTERM 대기
    signal::ctrl_c().await.expect("Failed to listen for ctrl+c");
    
    println!("Shutting down gracefully...");
    // 정리 작업
    cleanup().await;
}
```

## 복잡한 시그널 처리
```rust
use tokio::signal::unix::{signal, SignalKind};

async fn shutdown_signal() {
    let mut sigterm = signal(SignalKind::terminate())
        .expect("Failed to create SIGTERM handler");
    let mut sigint = signal(SignalKind::interrupt())
        .expect("Failed to create SIGINT handler");
    
    tokio::select! {
        _ = sigterm.recv() => {
            println!("Received SIGTERM");
        }
        _ = sigint.recv() => {
            println!("Received SIGINT (Ctrl+C)");
        }
    }
}
```

## 웹 서버 우아한 종료
```rust
use axum::{Router, Server};
use std::net::SocketAddr;
use tokio::sync::broadcast;

#[tokio::main]
async fn main() {
    let (shutdown_tx, mut shutdown_rx) = broadcast::channel(1);
    
    let app = Router::new()
        .route("/", get(handler));
    
    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    
    let server = Server::bind(&addr)
        .serve(app.into_make_service())
        .with_graceful_shutdown(async move {
            shutdown_rx.recv().await.ok();
        });
    
    tokio::spawn(async move {
        signal::ctrl_c().await.unwrap();
        println!("Shutdown signal received");
        shutdown_tx.send(()).unwrap();
    });
    
    server.await.unwrap();
    println!("Server stopped gracefully");
}
```

## 작업 추적 및 대기
```rust
use std::sync::Arc;
use tokio::sync::{RwLock, Semaphore};

struct GracefulShutdown {
    active_tasks: Arc<Semaphore>,
    is_shutting_down: Arc<RwLock<bool>>,
}

impl GracefulShutdown {
    fn new(max_tasks: usize) -> Self {
        Self {
            active_tasks: Arc::new(Semaphore::new(max_tasks)),
            is_shutting_down: Arc::new(RwLock::new(false)),
        }
    }
    
    async fn run_task<F, Fut>(&self, f: F) -> Result<(), &'static str>
    where
        F: FnOnce() -> Fut,
        Fut: Future<Output = ()>,
    {
        if *self.is_shutting_down.read().await {
            return Err("System is shutting down");
        }
        
        let permit = self.active_tasks.acquire().await.unwrap();
        f().await;
        drop(permit);
        
        Ok(())
    }
    
    async fn shutdown(&self) {
        *self.is_shutting_down.write().await = true;
        
        // 모든 작업이 완료될 때까지 대기
        for _ in 0..self.active_tasks.available_permits() {
            self.active_tasks.acquire().await.unwrap().forget();
        }
    }
}
```

## 타임아웃과 함께 종료
```rust
use tokio::time::{timeout, Duration};

async fn shutdown_with_timeout(duration: Duration) {
    println!("Starting graceful shutdown...");
    
    match timeout(duration, shutdown_tasks()).await {
        Ok(_) => println!("All tasks completed"),
        Err(_) => {
            eprintln!("Shutdown timeout exceeded, forcing termination");
            force_shutdown().await;
        }
    }
}

async fn shutdown_tasks() {
    // 진행 중인 작업 완료
}

async fn force_shutdown() {
    // 강제 종료
    std::process::exit(1);
}
```

## 데이터베이스 연결 종료
```rust
use sqlx::PgPool;

struct AppState {
    db: PgPool,
    shutdown: broadcast::Sender<()>,
}

impl AppState {
    async fn graceful_shutdown(&self) {
        println!("Closing database connections...");
        self.db.close().await;
        
        println!("Notifying all workers...");
        self.shutdown.send(()).ok();
    }
}
```

## 백그라운드 작업 관리
```rust
use tokio::task::JoinSet;
use tokio_util::sync::CancellationToken;

struct WorkerManager {
    workers: JoinSet<()>,
    cancel_token: CancellationToken,
}

impl WorkerManager {
    fn spawn_worker(&mut self, id: usize) {
        let token = self.cancel_token.clone();
        
        self.workers.spawn(async move {
            loop {
                tokio::select! {
                    _ = token.cancelled() => {
                        println!("Worker {} shutting down", id);
                        break;
                    }
                    _ = do_work() => {
                        // 작업 수행
                    }
                }
            }
        });
    }
    
    async fn shutdown(mut self) {
        self.cancel_token.cancel();
        
        while let Some(result) = self.workers.join_next().await {
            match result {
                Ok(_) => println!("Worker stopped"),
                Err(e) => eprintln!("Worker panicked: {}", e),
            }
        }
    }
}
```

## 상태 저장
```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize)]
struct AppState {
    last_processed_id: u64,
    pending_jobs: Vec<Job>,
}

async fn save_state_on_shutdown(state: AppState) {
    let json = serde_json::to_string(&state).unwrap();
    tokio::fs::write("state.json", json).await.unwrap();
    println!("State saved successfully");
}

async fn restore_state_on_startup() -> AppState {
    match tokio::fs::read_to_string("state.json").await {
        Ok(json) => serde_json::from_str(&json).unwrap(),
        Err(_) => AppState::default(),
    }
}
```

## 헬스체크 통합
```rust
use std::sync::atomic::{AtomicBool, Ordering};

static HEALTHY: AtomicBool = AtomicBool::new(true);

async fn health_check() -> impl IntoResponse {
    if HEALTHY.load(Ordering::Relaxed) {
        StatusCode::OK
    } else {
        StatusCode::SERVICE_UNAVAILABLE
    }
}

async fn begin_shutdown() {
    // 헬스체크 실패 시작
    HEALTHY.store(false, Ordering::Relaxed);
    
    // 로드밸런서가 트래픽 전환할 시간 제공
    tokio::time::sleep(Duration::from_secs(10)).await;
    
    // 실제 종료 시작
    shutdown().await;
}
```

## Kubernetes 통합
```rust
// Kubernetes preStop hook 처리
async fn k8s_graceful_shutdown() {
    // SIGTERM 수신 대기
    signal::ctrl_c().await.unwrap();
    
    // preStop 훅 실행 시간 고려
    let shutdown_deadline = Duration::from_secs(30);
    
    timeout(shutdown_deadline, async {
        // 새 요청 거부
        stop_accepting_requests().await;
        
        // 진행 중인 요청 완료
        drain_connections().await;
        
        // 리소스 정리
        cleanup_resources().await;
    }).await.ok();
}
```

## Rust 특성 활용
- **비동기 런타임**: 우아한 작업 취소
- **채널**: 종료 신호 전파
- **RAII**: 자동 리소스 정리
- **타입 시스템**: 상태 추적

## 체크리스트
1. 시그널 핸들러 등록
2. 새 요청 거부
3. 진행 중인 작업 완료
4. 데이터 영속화
5. 연결 종료
6. 로그 플러시
7. 메트릭 전송

## 관련 개념
- [[118_signal_handling]] - 시그널 처리
- [[119_health_checks]] - 헬스 체크
- [[120_state_persistence]] - 상태 영속성