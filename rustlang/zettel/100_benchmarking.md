# Benchmarking with Criterion

## 핵심 개념
통계적으로 의미 있는 성능 측정과 회귀 감지

## Criterion 설정
```toml
[dev-dependencies]
criterion = { version = "0.5", features = ["html_reports"] }

[[bench]]
name = "my_benchmark"
harness = false
```

## 기본 벤치마크
```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn fibonacci(n: u64) -> u64 {
    match n {
        0 => 1,
        1 => 1,
        n => fibonacci(n-1) + fibonacci(n-2),
    }
}

fn criterion_benchmark(c: &mut Criterion) {
    c.bench_function("fib 20", |b| {
        b.iter(|| fibonacci(black_box(20)))
    });
}

criterion_group!(benches, criterion_benchmark);
criterion_main!(benches);
```

## 파라미터화된 벤치마크
```rust
use criterion::{BenchmarkId, Criterion, Throughput};

fn bench_with_input(c: &mut Criterion) {
    let mut group = c.benchmark_group("sorting");
    
    for size in [100, 1000, 10000].iter() {
        group.throughput(Throughput::Elements(*size as u64));
        group.bench_with_input(
            BenchmarkId::from_parameter(size),
            size,
            |b, &size| {
                let mut v: Vec<_> = (0..size).collect();
                b.iter(|| {
                    v.sort_unstable();
                    black_box(&v);
                });
            },
        );
    }
    group.finish();
}
```

## 비교 벤치마크
```rust
fn compare_algorithms(c: &mut Criterion) {
    let mut group = c.benchmark_group("search");
    let data: Vec<i32> = (0..1000).collect();
    let target = 750;
    
    group.bench_function("linear", |b| {
        b.iter(|| {
            data.iter().position(|&x| x == target)
        })
    });
    
    group.bench_function("binary", |b| {
        b.iter(|| {
            data.binary_search(&target)
        })
    });
    
    group.finish();
}
```

## 커스텀 측정
```rust
use criterion::measurement::WallTime;
use std::time::Duration;

fn custom_measurement(c: &mut Criterion<WallTime>) {
    c.bench_function("custom", |b| {
        b.iter_custom(|iters| {
            let start = std::time::Instant::now();
            for _ in 0..iters {
                // 측정할 코드
                expensive_operation();
            }
            start.elapsed()
        })
    });
}
```

## 비동기 벤치마크
```rust
use criterion::async_executor::FuturesExecutor;

fn bench_async(c: &mut Criterion) {
    let runtime = tokio::runtime::Runtime::new().unwrap();
    
    c.bench_function("async_operation", |b| {
        b.to_async(&runtime).iter(|| async {
            async_function().await
        })
    });
}
```

## 프로파일링 통합
```rust
use criterion::profiler::Profiler;
use pprof::ProfilerGuard;

struct FlamegraphProfiler<'a> {
    guard: Option<ProfilerGuard<'a>>,
}

impl<'a> Profiler for FlamegraphProfiler<'a> {
    fn start_profiling(&mut self, _benchmark_id: &str, _benchmark_dir: &Path) {
        self.guard = Some(ProfilerGuard::new(100).unwrap());
    }
    
    fn stop_profiling(&mut self, benchmark_id: &str, benchmark_dir: &Path) {
        if let Some(guard) = self.guard.take() {
            let report = guard.report().build().unwrap();
            let file = File::create(
                benchmark_dir.join(format!("{}.svg", benchmark_id))
            ).unwrap();
            report.flamegraph(&file).unwrap();
        }
    }
}
```

## 회귀 감지
```rust
// criterion.toml
[profile]
sample_size = 100
warm_up_time = "3s"
measurement_time = "10s"
noise_threshold = 0.05
confidence_level = 0.95
significance_level = 0.05
```

## 마이크로 벤치마크 함정
```rust
// 잘못된 예: 컴파일러가 최적화로 제거
fn bad_benchmark(b: &mut Bencher) {
    b.iter(|| {
        let x = 1 + 1;  // 결과 사용 안 함
    })
}

// 올바른 예: black_box로 최적화 방지
fn good_benchmark(b: &mut Bencher) {
    b.iter(|| {
        black_box(1 + 1)
    })
}
```

## 메모리 벤치마크
```rust
use dhat::{Dhat, DhatAlloc};

#[global_allocator]
static ALLOCATOR: DhatAlloc = DhatAlloc;

fn memory_benchmark() {
    let _profiler = Dhat::start_heap_profiling();
    
    // 메모리 집중 작업
    let v: Vec<_> = (0..1000000).collect();
    
    let stats = dhat::HeapStats::get();
    println!("Total allocated: {} bytes", stats.total_bytes);
    println!("Peak allocated: {} bytes", stats.peak_bytes);
}
```

## CI 통합
```yaml
# .github/workflows/bench.yml
- name: Run benchmarks
  run: cargo bench -- --output-format bencher | tee output.txt

- name: Store benchmark result
  uses: benchmark-action/github-action-benchmark@v1
  with:
    tool: 'cargo'
    output-file-path: output.txt
    alert-threshold: '200%'
    comment-on-alert: true
```

## Rust 특성 활용
- **통계적 엄격성**: 신뢰 구간 계산
- **인라인 최적화**: black_box로 제어
- **제로 비용**: 벤치마크 오버헤드 최소화
- **타입 안전**: 벤치마크 그룹 타입 체크

## 모범 사례
- 워밍업 시간 충분히
- 여러 입력 크기 테스트
- 시스템 노이즈 최소화
- 회귀 자동 감지 설정
- 실제 사용 패턴 반영

## 관련 개념
- [[099_profiling_tools]] - 프로파일링
- [[101_simd_intrinsics]] - SIMD 최적화
- [[103_memory_alignment]] - 메모리 정렬