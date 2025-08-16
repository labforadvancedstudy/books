# WASI (WebAssembly System Interface)

## 핵심 개념
WebAssembly를 브라우저 밖에서 실행하기 위한 시스템 인터페이스

## WASI 타겟 설정
```toml
# Cargo.toml
[dependencies]
wasi = "0.13"

# 빌드 타겟
# rustup target add wasm32-wasip1
# cargo build --target wasm32-wasip1
```

## 파일 시스템 접근
```rust
use std::fs;
use std::io::{self, Read, Write};

fn main() -> io::Result<()> {
    // WASI는 샌드박스된 파일 시스템 접근 제공
    let contents = fs::read_to_string("/sandbox/input.txt")?;
    println!("File contents: {}", contents);
    
    // 파일 쓰기
    let mut file = fs::File::create("/sandbox/output.txt")?;
    file.write_all(b"Hello from WASI!")?;
    
    Ok(())
}
```

## 환경 변수와 인자
```rust
use std::env;

fn main() {
    // 명령줄 인자
    let args: Vec<String> = env::args().collect();
    println!("Arguments: {:?}", args);
    
    // 환경 변수
    for (key, value) in env::vars() {
        println!("{}: {}", key, value);
    }
    
    // 특정 환경 변수
    if let Ok(home) = env::var("HOME") {
        println!("Home directory: {}", home);
    }
}
```

## 네트워크 소켓 (WASI Preview 2)
```rust
use wasi::sockets::tcp::{TcpSocket, SocketAddr};
use wasi::io::streams::{InputStream, OutputStream};

async fn tcp_client() -> Result<(), Box<dyn std::error::Error>> {
    let socket = TcpSocket::new()?;
    let addr = SocketAddr::from(([127, 0, 0, 1], 8080));
    
    socket.connect(&addr).await?;
    
    let mut output = socket.output_stream()?;
    output.write(b"GET / HTTP/1.1\r\nHost: localhost\r\n\r\n")?;
    
    let mut input = socket.input_stream()?;
    let mut buffer = vec![0u8; 1024];
    let bytes_read = input.read(&mut buffer)?;
    
    println!("Response: {}", String::from_utf8_lossy(&buffer[..bytes_read]));
    
    Ok(())
}
```

## 시간과 클럭
```rust
use std::time::{Duration, SystemTime};

fn main() {
    // 현재 시간
    let now = SystemTime::now();
    println!("Current time: {:?}", now);
    
    // 경과 시간 측정
    let start = SystemTime::now();
    
    // 작업 수행
    std::thread::sleep(Duration::from_millis(100));
    
    let elapsed = start.elapsed().unwrap();
    println!("Elapsed: {:?}", elapsed);
}
```

## 난수 생성
```rust
use wasi::random::get_random_bytes;

fn generate_random() -> [u8; 32] {
    let mut buffer = [0u8; 32];
    get_random_bytes(&mut buffer).expect("Failed to get random bytes");
    buffer
}

// 또는 표준 라이브러리 사용
use rand::Rng;

fn main() {
    let mut rng = rand::thread_rng();
    let random: u32 = rng.gen();
    println!("Random number: {}", random);
}
```

## 표준 입출력
```rust
use std::io::{self, BufRead, Write};

fn main() -> io::Result<()> {
    // 표준 출력
    println!("Enter your name:");
    io::stdout().flush()?;
    
    // 표준 입력
    let stdin = io::stdin();
    let mut line = String::new();
    stdin.lock().read_line(&mut line)?;
    
    // 표준 에러
    eprintln!("Debug: User entered: {}", line.trim());
    
    Ok(())
}
```

## WASI 권한 시스템
```rust
// host 실행 시 권한 설정
// wasmtime run --dir=/tmp::/sandbox app.wasm

use std::fs;

fn main() {
    // /sandbox로 매핑된 /tmp에만 접근 가능
    match fs::read_dir("/sandbox") {
        Ok(entries) => {
            for entry in entries {
                println!("{:?}", entry.unwrap().path());
            }
        }
        Err(e) => eprintln!("Access denied: {}", e),
    }
}
```

## Component Model
```rust
// wit/world.wit
package example:hello;

world hello-world {
    import print: func(msg: string);
    export run: func() -> string;
}

// src/lib.rs
wit_bindgen::generate!({
    world: "hello-world",
});

struct Component;

impl HelloWorld for Component {
    fn run() -> String {
        print("Hello from component!");
        "Success".to_string()
    }
}

export_hello_world!(Component);
```

## 비동기 WASI
```rust
use wasi::io::poll::{Pollable, poll};
use wasi::clocks::monotonic_clock;

async fn wait_with_timeout(pollable: Pollable, timeout_ms: u64) -> bool {
    let timeout = monotonic_clock::subscribe(
        monotonic_clock::now() + timeout_ms * 1_000_000
    );
    
    let result = poll(&[pollable, timeout]).await;
    
    result[0] // true if pollable is ready, false if timeout
}
```

## WASI 런타임 비교
```rust
// Wasmtime
// wasmtime run app.wasm

// Wasmer
// wasmer run app.wasm

// WasmEdge (더 많은 확장 지원)
// wasmedge app.wasm

// 각 런타임마다 지원하는 WASI 기능이 다름
#[cfg(target_os = "wasi")]
fn detect_runtime() {
    if std::env::var("WASMTIME_VERSION").is_ok() {
        println!("Running on Wasmtime");
    } else if std::env::var("WASMER_VERSION").is_ok() {
        println!("Running on Wasmer");
    }
}
```

## Rust 특성 활용
- **표준 라이브러리**: WASI 타겟 지원
- **안전성**: 샌드박스 실행
- **이식성**: 플랫폼 독립적
- **성능**: 네이티브에 가까운 속도

## 제한사항
- 스레드 지원 제한적
- 네트워크 기능 미완성
- 파일 시스템 샌드박싱
- 시스템 콜 제한

## 관련 개념
- [[104_webassembly_bindings]] - WASM 바인딩
- [[108_edge_computing]] - 엣지 컴퓨팅
- [[109_wasm_components]] - WASM 컴포넌트