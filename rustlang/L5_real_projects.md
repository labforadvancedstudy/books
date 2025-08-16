# L5: 실전 프로젝트 구축 - 이론에서 실무로

## Core Insight
실제 프로젝트를 만들면서 Rust의 철학이 어떻게 실무에 적용되는지 체험한다. CLI 도구부터 웹 서버, 시스템 프로그래밍까지, 각 도메인에서 Rust가 빛나는 이유를 발견한다.

---

## CLI 도구 만들기 - grep 클론

### 프로젝트 구조

```bash
$ cargo new minigrep
$ cd minigrep
$ tree
.
├── Cargo.toml
└── src
    ├── main.rs
    └── lib.rs
```

### Cargo.toml - 의존성 관리

```toml
[package]
name = "minigrep"
version = "0.1.0"
edition = "2021"

[dependencies]
clap = { version = "4.0", features = ["derive"] }
colored = "2.0"
regex = "1.7"
anyhow = "1.0"

[dev-dependencies]
assert_cmd = "2.0"
predicates = "2.0"
tempfile = "3.0"
```

### 인자 파싱 - clap 사용

```rust
use clap::Parser;

#[derive(Parser, Debug)]
#[command(name = "minigrep")]
#[command(about = "Rust로 만든 grep", long_about = None)]
struct Args {
    /// 검색할 패턴
    pattern: String,
    
    /// 검색할 파일들
    #[arg(required = true)]
    files: Vec<String>,
    
    /// 대소문자 무시
    #[arg(short, long)]
    ignore_case: bool,
    
    /// 줄 번호 표시
    #[arg(short = 'n', long)]
    line_number: bool,
    
    /// 재귀적 검색
    #[arg(short, long)]
    recursive: bool,
}
```

### 핵심 로직

```rust
// lib.rs
use std::fs;
use std::path::Path;
use regex::Regex;
use colored::*;
use anyhow::{Result, Context};

pub struct Config {
    pub pattern: Regex,
    pub files: Vec<String>,
    pub line_number: bool,
    pub recursive: bool,
}

impl Config {
    pub fn new(args: Args) -> Result<Self> {
        let pattern = if args.ignore_case {
            Regex::new(&format!("(?i){}", args.pattern))
        } else {
            Regex::new(&args.pattern)
        }.context("잘못된 정규표현식")?;
        
        Ok(Config {
            pattern,
            files: args.files,
            line_number: args.line_number,
            recursive: args.recursive,
        })
    }
}

pub fn run(config: Config) -> Result<()> {
    for file in &config.files {
        if config.recursive && Path::new(file).is_dir() {
            search_directory(file, &config)?;
        } else {
            search_file(file, &config)?;
        }
    }
    Ok(())
}

fn search_file(filename: &str, config: &Config) -> Result<()> {
    let contents = fs::read_to_string(filename)
        .with_context(|| format!("파일 읽기 실패: {}", filename))?;
    
    let results = search(&contents, &config.pattern, config.line_number);
    
    if !results.is_empty() {
        println!("{}", filename.blue().bold());
        for result in results {
            println!("{}", result);
        }
        println!();
    }
    
    Ok(())
}

fn search(contents: &str, pattern: &Regex, show_line_number: bool) -> Vec<String> {
    contents
        .lines()
        .enumerate()
        .filter_map(|(line_num, line)| {
            if pattern.is_match(line) {
                let highlighted = pattern.replace_all(line, |caps: &regex::Captures| {
                    caps[0].red().bold().to_string()
                });
                
                Some(if show_line_number {
                    format!("{:4}: {}", line_num + 1, highlighted)
                } else {
                    highlighted.to_string()
                })
            } else {
                None
            }
        })
        .collect()
}

fn search_directory(dir: &str, config: &Config) -> Result<()> {
    use walkdir::WalkDir;
    
    for entry in WalkDir::new(dir)
        .into_iter()
        .filter_map(|e| e.ok())
        .filter(|e| e.file_type().is_file())
    {
        search_file(entry.path().to_str().unwrap(), config)?;
    }
    
    Ok(())
}
```

### main.rs - 진입점

```rust
use clap::Parser;
use minigrep::{Args, Config, run};

fn main() {
    let args = Args::parse();
    
    let config = Config::new(args).unwrap_or_else(|err| {
        eprintln!("인자 파싱 실패: {}", err);
        std::process::exit(1);
    });
    
    if let Err(e) = run(config) {
        eprintln!("애플리케이션 에러: {}", e);
        std::process::exit(1);
    }
}
```

### 테스트 작성

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use tempfile::NamedTempFile;
    use std::io::Write;
    
    #[test]
    fn test_case_sensitive() {
        let pattern = Regex::new("duct").unwrap();
        let contents = "\
Rust:
safe, fast, productive.
Pick three.";
        
        let results = search(contents, &pattern, false);
        assert_eq!(results.len(), 1);
        assert!(results[0].contains("productive"));
    }
    
    #[test]
    fn test_with_line_numbers() {
        let pattern = Regex::new("Rust").unwrap();
        let contents = "Rust is great\nI love Rust";
        
        let results = search(contents, &pattern, true);
        assert_eq!(results.len(), 2);
        assert!(results[0].starts_with("   1:"));
        assert!(results[1].starts_with("   2:"));
    }
}
```

## 웹 서버 구축 - RESTful API

### 프로젝트 설정

```toml
[dependencies]
axum = "0.7"
tokio = { version = "1", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
sqlx = { version = "0.7", features = ["runtime-tokio-rustls", "postgres"] }
tower = "0.4"
tower-http = { version = "0.5", features = ["cors", "trace"] }
tracing = "0.1"
tracing-subscriber = "0.3"
```

### 데이터 모델

```rust
use serde::{Deserialize, Serialize};
use sqlx::FromRow;

#[derive(Debug, Serialize, Deserialize, FromRow)]
pub struct Todo {
    pub id: i32,
    pub title: String,
    pub completed: bool,
    pub created_at: chrono::DateTime<chrono::Utc>,
}

#[derive(Debug, Deserialize)]
pub struct CreateTodo {
    pub title: String,
}

#[derive(Debug, Deserialize)]
pub struct UpdateTodo {
    pub title: Option<String>,
    pub completed: Option<bool>,
}
```

### 데이터베이스 레이어

```rust
use sqlx::{PgPool, postgres::PgPoolOptions};
use std::sync::Arc;

#[derive(Clone)]
pub struct AppState {
    pub db: Arc<PgPool>,
}

impl AppState {
    pub async fn new(database_url: &str) -> Result<Self, sqlx::Error> {
        let pool = PgPoolOptions::new()
            .max_connections(5)
            .connect(database_url)
            .await?;
        
        // 마이그레이션 실행
        sqlx::migrate!("./migrations")
            .run(&pool)
            .await?;
        
        Ok(AppState {
            db: Arc::new(pool),
        })
    }
}
```

### API 핸들러

```rust
use axum::{
    extract::{Path, State},
    http::StatusCode,
    response::IntoResponse,
    Json,
};

pub async fn list_todos(
    State(state): State<AppState>,
) -> Result<Json<Vec<Todo>>, StatusCode> {
    let todos = sqlx::query_as::<_, Todo>("SELECT * FROM todos ORDER BY created_at DESC")
        .fetch_all(&*state.db)
        .await
        .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;
    
    Ok(Json(todos))
}

pub async fn create_todo(
    State(state): State<AppState>,
    Json(payload): Json<CreateTodo>,
) -> Result<impl IntoResponse, StatusCode> {
    let todo = sqlx::query_as::<_, Todo>(
        "INSERT INTO todos (title, completed) VALUES ($1, false) RETURNING *"
    )
    .bind(&payload.title)
    .fetch_one(&*state.db)
    .await
    .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;
    
    Ok((StatusCode::CREATED, Json(todo)))
}

pub async fn get_todo(
    State(state): State<AppState>,
    Path(id): Path<i32>,
) -> Result<Json<Todo>, StatusCode> {
    let todo = sqlx::query_as::<_, Todo>("SELECT * FROM todos WHERE id = $1")
        .bind(id)
        .fetch_optional(&*state.db)
        .await
        .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?
        .ok_or(StatusCode::NOT_FOUND)?;
    
    Ok(Json(todo))
}

pub async fn update_todo(
    State(state): State<AppState>,
    Path(id): Path<i32>,
    Json(payload): Json<UpdateTodo>,
) -> Result<Json<Todo>, StatusCode> {
    // 보안 강화: prepared statements 사용으로 SQL Injection 방지
    let todo = match (payload.title, payload.completed) {
        (Some(title), Some(completed)) => {
            sqlx::query_as::<_, Todo>(
                "UPDATE todos SET title = $1, completed = $2 
                 WHERE id = $3 RETURNING *"
            )
            .bind(title)
            .bind(completed)
            .bind(id)
            .fetch_optional(&*state.db)
            .await
        }
        (Some(title), None) => {
            sqlx::query_as::<_, Todo>(
                "UPDATE todos SET title = $1 
                 WHERE id = $2 RETURNING *"
            )
            .bind(title)
            .bind(id)
            .fetch_optional(&*state.db)
            .await
        }
        (None, Some(completed)) => {
            sqlx::query_as::<_, Todo>(
                "UPDATE todos SET completed = $1 
                 WHERE id = $2 RETURNING *"
            )
            .bind(completed)
            .bind(id)
            .fetch_optional(&*state.db)
            .await
        }
        (None, None) => {
            return Err(StatusCode::BAD_REQUEST);
        }
    }
    .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?
    .ok_or(StatusCode::NOT_FOUND)?;
    
    Ok(Json(todo))
}

pub async fn delete_todo(
    State(state): State<AppState>,
    Path(id): Path<i32>,
) -> StatusCode {
    let result = sqlx::query("DELETE FROM todos WHERE id = $1")
        .bind(id)
        .execute(&*state.db)
        .await;
    
    match result {
        Ok(result) if result.rows_affected() > 0 => StatusCode::NO_CONTENT,
        Ok(_) => StatusCode::NOT_FOUND,
        Err(_) => StatusCode::INTERNAL_SERVER_ERROR,
    }
}
```

### 서버 구동

```rust
use axum::{
    routing::{get, post, put, delete},
    Router,
};
use tower_http::cors::CorsLayer;
use tracing_subscriber;

#[tokio::main]
async fn main() {
    // 로깅 초기화
    tracing_subscriber::fmt::init();
    
    // 데이터베이스 연결
    let database_url = std::env::var("DATABASE_URL")
        .unwrap_or_else(|_| "postgres://user:pass@localhost/todos".to_string());
    
    let state = AppState::new(&database_url)
        .await
        .expect("데이터베이스 연결 실패");
    
    // 라우터 구성
    let app = Router::new()
        .route("/todos", get(list_todos).post(create_todo))
        .route("/todos/:id", get(get_todo).put(update_todo).delete(delete_todo))
        .layer(CorsLayer::permissive())
        .with_state(state);
    
    // 서버 시작
    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();
    
    tracing::info!("서버 시작: http://0.0.0.0:3000");
    
    axum::serve(listener, app)
        .await
        .unwrap();
}
```

## 시스템 프로그래밍 - 파일 시스템 모니터

```rust
use notify::{Watcher, RecursiveMode, watcher, DebouncedEvent};
use std::sync::mpsc::channel;
use std::time::Duration;
use std::path::Path;

fn main() {
    // 채널 생성
    let (tx, rx) = channel();
    
    // 감시자 생성 (2초 디바운스)
    let mut watcher = watcher(tx, Duration::from_secs(2)).unwrap();
    
    // 감시할 경로 추가
    watcher.watch("./src", RecursiveMode::Recursive).unwrap();
    
    println!("파일 시스템 감시 중...");
    
    loop {
        match rx.recv() {
            Ok(event) => handle_event(event),
            Err(e) => println!("감시 에러: {:?}", e),
        }
    }
}

fn handle_event(event: DebouncedEvent) {
    match event {
        DebouncedEvent::Create(path) => {
            println!("생성: {:?}", path);
            if path.extension() == Some("rs".as_ref()) {
                compile_check(&path);
            }
        }
        DebouncedEvent::Write(path) => {
            println!("수정: {:?}", path);
            if path.extension() == Some("rs".as_ref()) {
                compile_check(&path);
            }
        }
        DebouncedEvent::Remove(path) => {
            println!("삭제: {:?}", path);
        }
        DebouncedEvent::Rename(from, to) => {
            println!("이름 변경: {:?} -> {:?}", from, to);
        }
        _ => {}
    }
}

fn compile_check(path: &Path) {
    use std::process::Command;
    
    println!("컴파일 체크: {:?}", path);
    
    let output = Command::new("rustc")
        .args(&["--edition", "2021", "--crate-type", "lib", "--no-emit"])
        .arg(path)
        .output()
        .expect("rustc 실행 실패");
    
    if !output.status.success() {
        println!("컴파일 에러:");
        println!("{}", String::from_utf8_lossy(&output.stderr));
    } else {
        println!("✓ 컴파일 성공");
    }
}
```

## 에러 처리의 예술

### 커스텀 에러 타입

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("데이터베이스 에러: {0}")]
    Database(#[from] sqlx::Error),
    
    #[error("IO 에러: {0}")]
    Io(#[from] std::io::Error),
    
    #[error("파싱 에러: {0}")]
    Parse(String),
    
    #[error("찾을 수 없음: {0}")]
    NotFound(String),
    
    #[error("권한 없음")]
    Unauthorized,
    
    #[error("유효성 검사 실패: {0}")]
    Validation(String),
}

// axum과 통합
impl IntoResponse for AppError {
    fn into_response(self) -> Response {
        let (status, error_message) = match self {
            AppError::Database(ref e) => (StatusCode::INTERNAL_SERVER_ERROR, e.to_string()),
            AppError::NotFound(ref e) => (StatusCode::NOT_FOUND, e.to_string()),
            AppError::Unauthorized => (StatusCode::UNAUTHORIZED, "Unauthorized".to_string()),
            AppError::Validation(ref e) => (StatusCode::BAD_REQUEST, e.to_string()),
            _ => (StatusCode::INTERNAL_SERVER_ERROR, "Internal server error".to_string()),
        };
        
        let body = Json(json!({
            "error": error_message,
        }));
        
        (status, body).into_response()
    }
}
```

### Result 체이닝

```rust
fn process_data(input: &str) -> Result<String, AppError> {
    parse_input(input)?
        .validate()?
        .transform()?
        .format()
        .ok_or_else(|| AppError::Parse("포맷 실패".into()))
}

// 더 복잡한 에러 처리
async fn complex_operation(id: i32) -> Result<Data, AppError> {
    let user = fetch_user(id)
        .await
        .map_err(|e| AppError::Database(e))?;
    
    let permissions = check_permissions(&user)
        .map_err(|_| AppError::Unauthorized)?;
    
    let data = fetch_data(&user, permissions)
        .await
        .map_err(|e| match e {
            DataError::NotFound => AppError::NotFound(format!("Data for user {}", id)),
            DataError::Invalid(msg) => AppError::Validation(msg),
            _ => AppError::Database(e.into()),
        })?;
    
    Ok(data)
}
```

## 통합 테스트

```rust
#[cfg(test)]
mod integration_tests {
    use super::*;
    use axum::http::StatusCode;
    use axum_test_helper::TestClient;
    
    #[tokio::test]
    async fn test_todo_crud() {
        let app = create_app().await;
        let client = TestClient::new(app);
        
        // CREATE
        let response = client
            .post("/todos")
            .json(&json!({ "title": "Test todo" }))
            .send()
            .await;
        
        assert_eq!(response.status(), StatusCode::CREATED);
        let todo: Todo = response.json().await;
        assert_eq!(todo.title, "Test todo");
        
        // READ
        let response = client
            .get(&format!("/todos/{}", todo.id))
            .send()
            .await;
        
        assert_eq!(response.status(), StatusCode::OK);
        
        // UPDATE
        let response = client
            .put(&format!("/todos/{}", todo.id))
            .json(&json!({ "completed": true }))
            .send()
            .await;
        
        assert_eq!(response.status(), StatusCode::OK);
        let updated: Todo = response.json().await;
        assert!(updated.completed);
        
        // DELETE
        let response = client
            .delete(&format!("/todos/{}", todo.id))
            .send()
            .await;
        
        assert_eq!(response.status(), StatusCode::NO_CONTENT);
    }
}
```

## 보안 강화: SQLx query! 매크로

### 컴파일 타임 SQL 검증

```rust
// .env 파일에 DATABASE_URL 설정 필요
// DATABASE_URL=postgres://user:pass@localhost/todos

use sqlx::query;

// 컴파일 타임에 SQL 쿼리 검증
pub async fn create_todo_safe(
    State(state): State<AppState>,
    Json(payload): Json<CreateTodo>,
) -> Result<impl IntoResponse, StatusCode> {
    // query! 매크로는 컴파일 타임에 실제 DB와 연결해 쿼리 검증
    let todo = query!(
        r#"
        INSERT INTO todos (title, completed, created_at) 
        VALUES ($1, false, NOW()) 
        RETURNING id, title, completed, created_at
        "#,
        payload.title
    )
    .fetch_one(&*state.db)
    .await
    .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;
    
    Ok((StatusCode::CREATED, Json(Todo {
        id: todo.id,
        title: todo.title,
        completed: todo.completed,
        created_at: todo.created_at,
    })))
}

// 타입 안전한 쿼리 빌더
pub async fn search_todos(
    State(state): State<AppState>,
    Query(params): Query<SearchParams>,
) -> Result<Json<Vec<Todo>>, StatusCode> {
    let mut query_builder = sqlx::QueryBuilder::new(
        "SELECT * FROM todos WHERE 1=1"
    );
    
    // 안전한 동적 쿼리 구성
    if let Some(title) = params.title {
        query_builder
            .push(" AND title ILIKE ")
            .push_bind(format!("%{}%", title));
    }
    
    if let Some(completed) = params.completed {
        query_builder
            .push(" AND completed = ")
            .push_bind(completed);
    }
    
    query_builder.push(" ORDER BY created_at DESC");
    
    let todos = query_builder
        .build_query_as::<Todo>()
        .fetch_all(&*state.db)
        .await
        .map_err(|_| StatusCode::INTERNAL_SERVER_ERROR)?;
    
    Ok(Json(todos))
}
```

### 보안 감사 체크리스트

```rust
// security_audit.rs
use std::collections::HashSet;

pub struct SecurityAudit {
    checks: Vec<SecurityCheck>,
}

#[derive(Debug)]
pub struct SecurityCheck {
    name: String,
    severity: Severity,
    passed: bool,
    message: String,
}

#[derive(Debug)]
pub enum Severity {
    Critical,
    High,
    Medium,
    Low,
}

impl SecurityAudit {
    pub fn new() -> Self {
        Self { checks: Vec::new() }
    }
    
    pub fn check_sql_injection(&mut self, code: &str) -> &mut Self {
        // SQL 문자열 연결 패턴 검사
        let dangerous_patterns = [
            "format!(\"", 
            "push_str(&format!",
            "+ &format!",
            ".push_str(&format!",
        ];
        
        let has_vulnerability = dangerous_patterns
            .iter()
            .any(|pattern| code.contains(pattern) && code.contains("WHERE"));
        
        self.checks.push(SecurityCheck {
            name: "SQL Injection".to_string(),
            severity: Severity::Critical,
            passed: !has_vulnerability,
            message: if has_vulnerability {
                "발견됨: 문자열 연결로 SQL 쿼리 구성. prepared statements 사용 필요".to_string()
            } else {
                "통과: prepared statements 사용 확인".to_string()
            }
        });
        
        self
    }
    
    pub fn check_input_validation(&mut self, handlers: &[&str]) -> &mut Self {
        for handler in handlers {
            let has_validation = handler.contains("validate") 
                || handler.contains("sanitize")
                || handler.contains("check");
            
            if !has_validation {
                self.checks.push(SecurityCheck {
                    name: "Input Validation".to_string(),
                    severity: Severity::High,
                    passed: false,
                    message: "입력 검증 로직 없음".to_string(),
                });
            }
        }
        self
    }
    
    pub fn report(&self) {
        println!("=== 보안 감사 결과 ===\n");
        
        let critical_issues: Vec<_> = self.checks
            .iter()
            .filter(|c| matches!(c.severity, Severity::Critical) && !c.passed)
            .collect();
        
        if !critical_issues.is_empty() {
            println!("🚨 치명적 보안 문제 발견:");
            for issue in critical_issues {
                println!("  - {}: {}", issue.name, issue.message);
            }
            println!();
        }
        
        for check in &self.checks {
            let icon = if check.passed { "✅" } else { "❌" };
            println!("{} {} ({:?}): {}", 
                icon, check.name, check.severity, check.message);
        }
    }
}

// 사용 예제
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_security_audit() {
        let vulnerable_code = r#"
            let query = format!("SELECT * FROM users WHERE id = {}", user_id);
        "#;
        
        let mut audit = SecurityAudit::new();
        audit.check_sql_injection(vulnerable_code).report();
        
        // 테스트는 실패해야 함 (취약점 발견)
        assert!(audit.checks.iter().any(|c| !c.passed));
    }
}
```

## 마무리: 실전의 교훈

실제 프로젝트를 통해 배운 것:
1. **보안이 최우선**: SQL Injection 같은 취약점은 치명적
2. **컴파일 타임 검증**: query! 매크로로 SQL 쿼리 안전성 보장
3. **에러 처리가 코드의 절반**: Result와 ? 연산자의 위력
4. **타입이 문서**: 잘 설계된 타입이 최고의 문서
5. **컴파일러가 최고의 동료**: 컴파일되면 대부분 작동
6. **생태계의 풍부함**: 필요한 크레이트가 거의 다 있음

---

## Connections
→ [[L6_ecosystem]] - Rust 생태계 탐험
→ [[zettel/040_error_handling]] - 에러 처리 철학
→ [[zettel/110_sqlx_async]] - 비동기 데이터베이스
← [[L4_concurrency]] - 동시성 기초

---

Level: L5 (실전 적용)
Date: 2025-08-15
Tags: #rust #cli #web-server #systems-programming #real-world