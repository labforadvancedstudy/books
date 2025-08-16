# Docker Multi-stage Builds for Rust

## 핵심 개념
컴파일과 런타임을 분리하여 최소 크기의 프로덕션 이미지 생성

## 기본 멀티스테이지 빌드
```dockerfile
# 빌드 스테이지
FROM rust:1.79 AS builder

WORKDIR /app
COPY Cargo.toml Cargo.lock ./
COPY src ./src

RUN cargo build --release

# 런타임 스테이지
FROM debian:bookworm-slim

RUN apt-get update && apt-get install -y \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

COPY --from=builder /app/target/release/myapp /usr/local/bin/myapp

EXPOSE 8080
CMD ["myapp"]
```

## 의존성 캐싱 최적화
```dockerfile
FROM rust:1.79 AS builder

WORKDIR /app

# 의존성만 먼저 빌드 (캐싱 활용)
COPY Cargo.toml Cargo.lock ./
RUN mkdir src && \
    echo "fn main() {}" > src/main.rs && \
    cargo build --release && \
    rm -rf src

# 실제 소스 복사 및 빌드
COPY src ./src
RUN touch src/main.rs && \
    cargo build --release

FROM gcr.io/distroless/cc-debian12

COPY --from=builder /app/target/release/myapp /app/myapp

ENTRYPOINT ["/app/myapp"]
```

## Alpine Linux 최소 이미지
```dockerfile
# musl 타겟으로 빌드
FROM rust:1.79-alpine AS builder

RUN apk add --no-cache musl-dev

WORKDIR /app
COPY . .

RUN cargo build --release --target x86_64-unknown-linux-musl

# 최소 런타임
FROM alpine:3.19

RUN apk add --no-cache ca-certificates

COPY --from=builder /app/target/x86_64-unknown-linux-musl/release/myapp /usr/local/bin/

CMD ["myapp"]
```

## 정적 링킹 (scratch 이미지)
```dockerfile
FROM rust:1.79 AS builder

# 정적 링킹 설정
RUN rustup target add x86_64-unknown-linux-musl
RUN apt-get update && apt-get install -y musl-tools

WORKDIR /app
COPY . .

RUN cargo build --release --target x86_64-unknown-linux-musl

# 최소 이미지 (크기: ~5MB)
FROM scratch

COPY --from=builder /app/target/x86_64-unknown-linux-musl/release/myapp /myapp

ENTRYPOINT ["/myapp"]
```

## Chef를 이용한 고급 캐싱
```dockerfile
FROM lukemathwalker/cargo-chef:latest-rust-1.79 AS chef
WORKDIR /app

# 레시피 생성
FROM chef AS planner
COPY . .
RUN cargo chef prepare --recipe-path recipe.json

# 의존성 빌드
FROM chef AS builder
COPY --from=planner /app/recipe.json recipe.json
RUN cargo chef cook --release --recipe-path recipe.json

# 애플리케이션 빌드
COPY . .
RUN cargo build --release

# 런타임
FROM debian:bookworm-slim

COPY --from=builder /app/target/release/myapp /usr/local/bin/

CMD ["myapp"]
```

## 워크스페이스 지원
```dockerfile
FROM rust:1.79 AS builder

WORKDIR /app

# 워크스페이스 구조 복사
COPY Cargo.toml Cargo.lock ./
COPY api/Cargo.toml api/
COPY core/Cargo.toml core/
COPY shared/Cargo.toml shared/

# 더미 파일로 의존성 빌드
RUN mkdir -p api/src core/src shared/src && \
    echo "fn main() {}" > api/src/main.rs && \
    echo "fn main() {}" > core/src/lib.rs && \
    echo "fn main() {}" > shared/src/lib.rs && \
    cargo build --release -p api && \
    rm -rf api/src core/src shared/src

# 실제 소스 복사
COPY . .
RUN cargo build --release -p api

FROM debian:bookworm-slim

COPY --from=builder /app/target/release/api /usr/local/bin/

CMD ["api"]
```

## 크로스 컴파일
```dockerfile
FROM messense/rust-musl-cross:x86_64-musl AS builder

WORKDIR /app
COPY . .

RUN cargo build --release --target x86_64-unknown-linux-musl

# ARM64용
FROM messense/rust-musl-cross:aarch64-musl AS builder-arm

WORKDIR /app
COPY . .

RUN cargo build --release --target aarch64-unknown-linux-musl
```

## 개발/프로덕션 분리
```dockerfile
# 개발 스테이지
FROM rust:1.79 AS development

WORKDIR /app
RUN cargo install cargo-watch

COPY . .

CMD ["cargo", "watch", "-x", "run"]

# 프로덕션 빌드
FROM rust:1.79 AS builder

WORKDIR /app
COPY . .

RUN cargo build --release

# 프로덕션 런타임
FROM debian:bookworm-slim AS production

COPY --from=builder /app/target/release/myapp /usr/local/bin/

CMD ["myapp"]
```

## 보안 강화
```dockerfile
FROM rust:1.79 AS builder

# 비 root 사용자로 빌드
RUN useradd -m -u 1001 rust
USER rust

WORKDIR /home/rust/app
COPY --chown=rust:rust . .

RUN cargo build --release

# 런타임
FROM gcr.io/distroless/cc-debian12

COPY --from=builder /home/rust/app/target/release/myapp /app/
USER nonroot

ENTRYPOINT ["/app/myapp"]
```

## 빌드 인자 활용
```dockerfile
ARG RUST_VERSION=1.79
FROM rust:${RUST_VERSION} AS builder

ARG FEATURES=""
ARG PROFILE="release"

WORKDIR /app
COPY . .

RUN cargo build --profile ${PROFILE} ${FEATURES:+--features ${FEATURES}}

FROM debian:bookworm-slim

ARG PROFILE="release"
COPY --from=builder /app/target/${PROFILE}/myapp /usr/local/bin/

CMD ["myapp"]
```

## Rust 특성 활용
- **정적 링킹**: 최소 런타임 의존성
- **크로스 컴파일**: 다중 플랫폼 지원
- **캐싱**: 빌드 시간 최적화
- **보안**: distroless 이미지

## 최적화 팁
- 의존성 레이어 분리
- cargo-chef 사용
- .dockerignore 활용
- 멀티 플랫폼 빌드
- 캐시 마운트 활용

## 관련 개념
- [[114_static_linking]] - 정적 링킹
- [[115_binary_optimization]] - 바이너리 최적화
- [[116_container_security]] - 컨테이너 보안