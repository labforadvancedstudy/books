# Private Registries

## 핵심 개념
기업 내부용 크레이트를 위한 비공개 레지스트리 구축

## Cargo 설정
```toml
# .cargo/config.toml
[registries]
company = { index = "https://git.company.com/rust/crates-index" }

[source.crates-io]
replace-with = "company-mirror"

[source.company-mirror]
registry = "https://crates.company.com/api/v1/crates"
```

## 레지스트리 인증
```toml
# ~/.cargo/credentials.toml
[registries.company]
token = "api_token_here"

# 또는 환경 변수
# CARGO_REGISTRIES_COMPANY_TOKEN=api_token_here
```

## Alexandrie 서버 설정
```toml
# alexandrie.toml
[general]
bind = "0.0.0.0:3000"
max_upload_size = 536870912  # 512MB

[index]
type = "git"
url = "git@github.com:company/crates-index.git"

[storage]
type = "s3"
bucket = "company-crates"
region = "us-east-1"
```

## 패키지 게시
```toml
# Cargo.toml
[package]
name = "internal-lib"
version = "0.1.0"
publish = ["company"]  # 특정 레지스트리만

[dependencies]
public-crate = "1.0"  # crates.io에서
internal-dep = { version = "0.2", registry = "company" }
```

## 게시 자동화
```rust
// publish.rs
use std::process::Command;

fn publish_to_registry(package: &str, registry: &str) {
    Command::new("cargo")
        .args(&["publish", "--registry", registry, "-p", package])
        .status()
        .expect("Failed to publish");
}

// CI/CD 통합
fn validate_before_publish() {
    // 버전 확인
    // 라이선스 검증
    // 보안 스캔
}
```

## Docker 기반 레지스트리
```dockerfile
FROM rust:latest as builder
RUN cargo install cargo-registry

FROM debian:slim
COPY --from=builder /usr/local/cargo/bin/cargo-registry /usr/bin/
VOLUME ["/var/lib/cargo-registry"]
EXPOSE 8080
CMD ["cargo-registry", "serve"]
```

## 미러링 설정
```rust
// 크레이트 미러링 스크립트
async fn mirror_crate(name: &str, version: &str) {
    // crates.io에서 다운로드
    let crate_data = download_from_crates_io(name, version).await?;
    
    // 내부 레지스트리에 업로드
    upload_to_private_registry(crate_data).await?;
    
    // 인덱스 업데이트
    update_index(name, version).await?;
}
```

## 접근 제어
```rust
// 레지스트리 미들웨어
async fn auth_middleware(req: Request, next: Next) -> Response {
    let token = req.headers().get("Authorization");
    
    if !validate_token(token).await {
        return Response::builder()
            .status(401)
            .body("Unauthorized");
    }
    
    next.run(req).await
}

// 팀별 권한
enum Permission {
    Publish,
    Yank,
    Admin,
}
```

## 캐싱 전략
```toml
# 로컬 캐시 설정
[net]
offline = false
git-fetch-with-cli = true

[http]
timeout = 30
cainfo = "/path/to/ca-bundle.crt"
```

## 의존성 감사
```rust
// 보안 스캔 통합
use cargo_audit::audit;

fn pre_publish_audit(manifest: &Path) -> Result<()> {
    let db = audit::Database::fetch()?;
    let report = audit::audit_package(manifest, &db)?;
    
    if !report.vulnerabilities.is_empty() {
        return Err("Security vulnerabilities found");
    }
    
    Ok(())
}
```

## Rust 특성 활용
- **Cargo 통합**: 네이티브 레지스트리 지원
- **Git 인덱스**: 버전 관리
- **토큰 인증**: 보안 접근
- **캐싱**: 빌드 속도 향상

## 장점
- 내부 코드 보호
- 접근 제어
- 의존성 감사
- 빌드 안정성

## 관련 개념
- [[093_cargo_workspaces]] - 워크스페이스
- [[097_cross_compilation]] - 크로스 컴파일
- [[098_binary_distribution]] - 바이너리 배포