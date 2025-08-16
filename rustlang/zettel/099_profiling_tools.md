# Profiling Tools for Rust

## 핵심 개념
성능 병목 지점을 찾기 위한 프로파일링 도구와 기법

## perf 사용
```bash
# 프로파일 심볼 포함 빌드
cargo build --release
# or with debug symbols
RUSTFLAGS="-g" cargo build --release

# perf 기록
perf record -g ./target/release/myapp
perf report

# 특정 이벤트 측정
perf stat -e cache-misses,cache-references ./target/release/myapp
```

## Flamegraph 생성
```bash
# 설치
cargo install flamegraph

# 플레임그래프 생성
cargo flamegraph --bin myapp

# 또는 수동으로
perf record -F 99 -g ./target/release/myapp
perf script | stackcollapse-perf.pl | flamegraph.pl > flame.svg
```

## 코드 계측
```rust
use std::time::Instant;

fn measure_function() {
    let start = Instant::now();
    
    expensive_operation();
    
    let duration = start.elapsed();
    println!("Operation took: {:?}", duration);
}

// 매크로로 자동화
macro_rules! measure {
    ($name:expr, $block:block) => {{
        let start = Instant::now();
        let result = $block;
        println!("{} took: {:?}", $name, start.elapsed());
        result
    }};
}

let result = measure!("database query", {
    db.query("SELECT * FROM users")
});
```

## pprof 통합
```rust
// Cargo.toml
[dependencies]
pprof = { version = "0.13", features = ["flamegraph"] }

// main.rs
use pprof::protos::Message;

fn profile_cpu() {
    let guard = pprof::ProfilerGuardBuilder::default()
        .frequency(1000)
        .blocklist(&["libc", "libgcc", "pthread"])
        .build()
        .unwrap();
    
    // 프로파일링할 코드
    expensive_computation();
    
    // 리포트 생성
    let report = guard.report().build().unwrap();
    let file = File::create("profile.pb").unwrap();
    report.write_to_writer(&mut file).unwrap();
}
```

## Valgrind/Cachegrind
```bash
# 캐시 미스 분석
valgrind --tool=cachegrind ./target/release/myapp

# 결과 분석
cg_annotate cachegrind.out.<pid>

# 특정 함수 집중 분석
cg_annotate --auto=yes --show=expensive_function cachegrind.out.<pid>
```

## 힙 프로파일링
```rust
// jemalloc 프로파일링
#[global_allocator]
static ALLOC: jemallocator::Jemalloc = jemallocator::Jemalloc;

// 프로파일 활성화
export MALLOC_CONF="prof:true,prof_prefix:jeprof.out"
./target/release/myapp

// 분석
jeprof --show_bytes ./target/release/myapp jeprof.out.*
```

## Tracy 실시간 프로파일러
```rust
use tracy_client::{Client, span};

fn traced_function() {
    let _span = span!("traced_function");
    
    // 함수 로직
    expensive_work();
    
    {
        let _inner = span!("inner_operation");
        inner_work();
    }
}

// 메모리 추적
fn track_allocation() {
    let data = vec![0u8; 1024 * 1024];
    Client::running()
        .unwrap()
        .emit_alloc(data.as_ptr(), data.len());
}
```

## 커스텀 메트릭
```rust
use std::sync::atomic::{AtomicU64, Ordering};

struct Metrics {
    function_calls: AtomicU64,
    cache_hits: AtomicU64,
    cache_misses: AtomicU64,
}

impl Metrics {
    fn record_call(&self) {
        self.function_calls.fetch_add(1, Ordering::Relaxed);
    }
    
    fn report(&self) {
        let calls = self.function_calls.load(Ordering::Relaxed);
        let hits = self.cache_hits.load(Ordering::Relaxed);
        let misses = self.cache_misses.load(Ordering::Relaxed);
        
        println!("Calls: {}, Hit rate: {:.2}%", 
            calls, 
            (hits as f64 / (hits + misses) as f64) * 100.0
        );
    }
}
```

## 프로파일 가이드 최적화
```toml
[profile.release]
lto = true
codegen-units = 1

# PGO (Profile Guided Optimization)
# 1단계: 프로파일 수집
[profile.release-pgo-generate]
inherits = "release"
debug = true

# 2단계: 프로파일 사용
[profile.release-pgo-use]
inherits = "release"
```

## 샘플링 프로파일러
```rust
use sample_prof::{Profiler, Report};

let mut profiler = Profiler::new(100); // 100Hz 샘플링
profiler.start();

// 작업 수행
do_work();

profiler.stop();
let report = profiler.report();
report.write_flamegraph("profile.svg")?;
```

## Rust 특성 활용
- **제로 비용 추상화**: 프로파일링 오버헤드 최소화
- **인라인 힌트**: 최적화 가이드
- **조건부 컴파일**: 프로파일링 코드 제거
- **안전한 동시성**: 락 경합 분석

## 프로파일링 체크리스트
1. 릴리즈 모드로 빌드
2. 대표적인 워크로드 사용
3. 여러 번 측정하여 평균
4. 시스템 노이즈 최소화
5. 병목 지점부터 최적화

## 관련 개념
- [[100_benchmarking]] - 벤치마킹
- [[101_simd_intrinsics]] - SIMD 최적화
- [[102_cache_optimization]] - 캐시 최적화