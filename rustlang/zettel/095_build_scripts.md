# Build Scripts (build.rs)

## 핵심 개념
컴파일 시점에 실행되는 Rust 스크립트로 빌드 커스터마이징

## 기본 구조
```rust
// build.rs
fn main() {
    println!("cargo:rerun-if-changed=build.rs");
    println!("cargo:rerun-if-changed=src/schema.sql");
    
    // 빌드 로직
    generate_code();
    compile_protos();
    link_native_lib();
}
```

## 환경 변수 설정
```rust
use std::env;
use std::path::PathBuf;

fn main() {
    // Cargo가 제공하는 환경 변수
    let out_dir = env::var("OUT_DIR").unwrap();
    let target = env::var("TARGET").unwrap();
    let host = env::var("HOST").unwrap();
    
    // 커스텀 환경 변수 설정
    println!("cargo:rustc-env=BUILD_TIME={}", chrono::Utc::now());
    println!("cargo:rustc-cfg=feature=\"custom\"");
}
```

## 코드 생성
```rust
use std::fs::File;
use std::io::Write;
use std::path::Path;

fn main() {
    let out_dir = env::var("OUT_DIR").unwrap();
    let dest_path = Path::new(&out_dir).join("generated.rs");
    let mut f = File::create(&dest_path).unwrap();
    
    // SQL 스키마에서 구조체 생성
    writeln!(f, "pub struct User {{").unwrap();
    writeln!(f, "    pub id: i32,").unwrap();
    writeln!(f, "    pub name: String,").unwrap();
    writeln!(f, "}}").unwrap();
}

// main.rs에서 사용
include!(concat!(env!("OUT_DIR"), "/generated.rs"));
```

## Protocol Buffers 컴파일
```rust
fn main() {
    prost_build::Config::new()
        .out_dir("src/generated")
        .compile_protos(&["proto/service.proto"], &["proto/"])
        .unwrap();
        
    // gRPC 서비스 생성
    tonic_build::configure()
        .build_server(true)
        .build_client(true)
        .compile(&["proto/service.proto"], &["proto/"])
        .unwrap();
}
```

## 네이티브 라이브러리 링킹
```rust
use pkg_config;
use cc;

fn main() {
    // pkg-config 사용
    pkg_config::probe_library("openssl").unwrap();
    
    // C 코드 컴파일
    cc::Build::new()
        .file("src/native/helper.c")
        .include("src/native")
        .compile("helper");
    
    // 정적 링킹
    println!("cargo:rustc-link-lib=static=helper");
    println!("cargo:rustc-link-search=native={}", out_dir);
}
```

## 기능 감지
```rust
use std::process::Command;

fn main() {
    // CPU 기능 확인
    let supports_avx = detect_avx_support();
    if supports_avx {
        println!("cargo:rustc-cfg=has_avx");
    }
    
    // 시스템 라이브러리 버전 확인
    let openssl_version = get_openssl_version();
    if openssl_version >= "1.1.0" {
        println!("cargo:rustc-cfg=modern_openssl");
    }
}

fn detect_avx_support() -> bool {
    // CPU 기능 감지 로직
}
```

## 조건부 컴파일 설정
```rust
fn main() {
    let target_os = env::var("CARGO_CFG_TARGET_OS").unwrap();
    
    match target_os.as_str() {
        "windows" => {
            println!("cargo:rustc-link-lib=user32");
            println!("cargo:rustc-cfg=windows_specific");
        }
        "macos" => {
            println!("cargo:rustc-link-lib=framework=CoreFoundation");
        }
        _ => {}
    }
}
```

## 빌드 의존성
```toml
[build-dependencies]
cc = "1.0"
bindgen = "0.70"
prost-build = "0.13"
walkdir = "2.5"
```

## 에셋 처리
```rust
use std::fs;
use walkdir::WalkDir;

fn main() {
    let out_dir = env::var("OUT_DIR").unwrap();
    
    // 에셋 파일 임베딩
    let mut assets = Vec::new();
    for entry in WalkDir::new("assets") {
        let entry = entry.unwrap();
        if entry.file_type().is_file() {
            let path = entry.path();
            let contents = fs::read(path).unwrap();
            assets.push((path.to_str().unwrap(), contents));
        }
    }
    
    // Rust 코드로 변환
    generate_asset_module(&out_dir, assets);
}
```

## 버전 정보 임베딩
```rust
use git2::Repository;

fn main() {
    let repo = Repository::open(".").unwrap();
    let head = repo.head().unwrap();
    let commit = head.peel_to_commit().unwrap();
    
    println!("cargo:rustc-env=GIT_HASH={}", commit.id());
    println!("cargo:rustc-env=BUILD_DATE={}", chrono::Utc::now());
}
```

## Rust 특성 활용
- **컴파일 타임 실행**: 빌드 중 코드 생성
- **환경 변수**: 빌드 설정 전달
- **조건부 컴파일**: 플랫폼별 최적화
- **FFI 통합**: 네이티브 코드 연결

## 주의사항
- 빌드 시간 증가
- 크로스 컴파일 복잡성
- 재현 가능한 빌드 보장
- 의존성 최소화

## 관련 개념
- [[096_feature_flags]] - 기능 플래그
- [[097_cross_compilation]] - 크로스 컴파일
- [[063_ffi]] - Foreign Function Interface