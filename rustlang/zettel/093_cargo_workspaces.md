# Cargo Workspaces

## 핵심 개념
여러 패키지를 하나의 저장소에서 관리하는 모노레포 구조

## 워크스페이스 구성
```toml
# 루트 Cargo.toml
[workspace]
members = [
    "core",
    "api",
    "cli",
    "shared",
]

resolver = "2"  # Rust 2021 에디션 리졸버

[workspace.package]
version = "0.1.0"
authors = ["Team"]
edition = "2021"
license = "MIT"

[workspace.dependencies]
tokio = { version = "1.40", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
sqlx = "0.8"
```

## 멤버 패키지 설정
```toml
# api/Cargo.toml
[package]
name = "api"
version.workspace = true
edition.workspace = true

[dependencies]
core = { path = "../core" }
shared = { path = "../shared" }
tokio.workspace = true
serde.workspace = true
```

## 공유 의존성 관리
```toml
[workspace.dependencies]
# 버전 중앙 관리
axum = "0.7"
tower = "0.5"
tracing = "0.1"

# 기능 플래그 통일
tokio = { version = "1.40", features = ["full"] }

# Git 의존성
my-lib = { git = "https://github.com/org/lib", branch = "main" }
```

## 빌드 최적화
```toml
[profile.dev]
opt-level = 0

[profile.dev.package."*"]  
opt-level = 3  # 의존성은 최적화

[profile.release]
lto = true  # Link Time Optimization
codegen-units = 1
strip = true
```

## 워크스페이스 명령어
```bash
# 전체 빌드
cargo build --workspace

# 특정 패키지만
cargo build -p api

# 의존하는 패키지 포함
cargo build -p api --all-features

# 전체 테스트
cargo test --workspace

# 특정 패키지 실행
cargo run -p cli -- --help
```

## 내부 패키지 참조
```rust
// api/src/main.rs
use core::domain::User;
use shared::config::Config;

// 재수출
pub use core::{
    Error,
    Result,
};
```

## 기능 플래그 전파
```toml
# core/Cargo.toml
[features]
default = []
postgres = ["sqlx/postgres"]
redis = ["redis-rs"]

# api/Cargo.toml
[features]
default = ["core/postgres"]
full = ["core/postgres", "core/redis"]
```

## 개발 워크플로우
```bash
# 변경 감지 및 재컴파일
cargo watch -x "test --workspace"

# 특정 패키지만 게시
cargo publish -p shared --dry-run

# 의존성 그래프
cargo tree --workspace

# 중복 의존성 확인
cargo tree -d
```

## 버전 관리
```rust
// 워크스페이스 버전 동기화 스크립트
use toml_edit::{Document, value};

fn sync_versions(version: &str) {
    for member in workspace_members() {
        let mut doc = read_cargo_toml(member);
        doc["package"]["version"] = value(version);
        write_cargo_toml(member, doc);
    }
}
```

## CI/CD 통합
```yaml
# .github/workflows/ci.yml
- name: Test workspace
  run: |
    cargo test --workspace --all-features
    cargo clippy --workspace -- -D warnings
    cargo fmt --all -- --check
```

## Rust 특성 활용
- **경로 의존성**: 로컬 패키지 참조
- **기능 통합**: 워크스페이스 레벨 기능
- **빌드 캐싱**: 공유 target 디렉토리
- **원자적 버전 관리**: 동시 업데이트

## 장점
- 코드 재사용
- 일관된 의존성 버전
- 빠른 빌드 (캐싱)
- 단일 CI/CD 파이프라인

## 관련 개념
- [[094_private_registries]] - 비공개 크레이트 레지스트리
- [[095_build_scripts]] - 빌드 스크립트
- [[096_feature_flags]] - 기능 플래그