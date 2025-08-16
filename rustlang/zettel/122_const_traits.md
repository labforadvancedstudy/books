# Const Traits

## 핵심 개념
컴파일 타임에 트레이트 메서드를 실행할 수 있게 하는 기능

## 기본 const 트레이트
```rust
#![feature(const_trait_impl)]

#[const_trait]
trait MyTrait {
    fn compute(&self) -> i32;
}

struct MyStruct(i32);

impl const MyTrait for MyStruct {
    fn compute(&self) -> i32 {
        self.0 * 2
    }
}

// 컴파일 타임 실행
const RESULT: i32 = MyStruct(21).compute();  // 42
```

## 표준 라이브러리 const 트레이트
```rust
#![feature(const_default_impls)]

// Default 트레이트의 const 구현
struct Config {
    timeout: u64,
    retries: u32,
}

impl const Default for Config {
    fn default() -> Self {
        Config {
            timeout: 30,
            retries: 3,
        }
    }
}

const DEFAULT_CONFIG: Config = Config::default();
```

## const 제네릭 바운드
```rust
#![feature(const_trait_impl, const_fn_trait_bound)]

const fn process<T: ~const Display>(value: T) -> String {
    // 컴파일 타임에 Display 사용
    format!("{}", value)
}

const MESSAGE: String = process(42);
```

## const Drop
```rust
#![feature(const_trait_impl)]

struct Resource {
    id: u32,
}

impl const Drop for Resource {
    fn drop(&mut self) {
        // 컴파일 타임 정리 로직
        // (실제로는 const에서 부작용 없음)
    }
}

const fn use_resource() {
    let _r = Resource { id: 1 };
    // 스코프 끝에서 const drop 호출
}
```

## const 클로저 트레이트
```rust
#![feature(const_closures, const_trait_impl)]

const fn apply_const<F: ~const Fn(i32) -> i32>(f: F, x: i32) -> i32 {
    f(x)
}

const DOUBLED: i32 = apply_const(|x| x * 2, 21);  // 42
```

## 조건부 const 구현
```rust
#![feature(const_trait_impl)]

trait Compute {
    fn calculate(&self) -> i32;
}

impl<T> const Compute for T
where
    T: ~const Add<Output = T> + ~const Mul<Output = T>,
{
    fn calculate(&self) -> i32 {
        // const 컨텍스트에서만 유효한 연산
    }
}
```

## const 이터레이터
```rust
#![feature(const_trait_impl, const_for)]

struct ConstArray<const N: usize> {
    data: [i32; N],
}

impl<const N: usize> const IntoIterator for ConstArray<N> {
    type Item = i32;
    type IntoIter = ConstIter<N>;
    
    fn into_iter(self) -> Self::IntoIter {
        ConstIter {
            array: self.data,
            index: 0,
        }
    }
}

const fn sum_array<const N: usize>(arr: ConstArray<N>) -> i32 {
    let mut sum = 0;
    for val in arr {
        sum += val;
    }
    sum
}
```

## const 트레이트 객체
```rust
#![feature(const_trait_impl)]

trait ConstCompute: ~const Sized {
    fn compute(&self) -> i32;
}

const fn use_trait_object(obj: &dyn ~const ConstCompute) -> i32 {
    obj.compute()
}
```

## 매크로와 const 트레이트
```rust
macro_rules! const_impl {
    ($type:ty) => {
        impl const MyTrait for $type {
            fn method(&self) -> i32 {
                // const 구현
                42
            }
        }
    };
}

const_impl!(u32);
const_impl!(i64);
```

## 제한사항 우회
```rust
#![feature(const_trait_impl)]

// 현재 제한: const 트레이트에서 가변 참조 불가
trait ConstSafe {
    fn process(&self) -> i32;  // &mut self 불가
}

// 우회: Cell/RefCell 사용 (미래에는 const_mut_refs)
use std::cell::Cell;

struct Counter {
    value: Cell<i32>,
}

impl const ConstSafe for Counter {
    fn process(&self) -> i32 {
        // Cell로 내부 가변성
        self.value.get() + 1
    }
}
```

## 성능 이점
```rust
// 컴파일 타임 계산
const fn factorial(n: u64) -> u64 {
    match n {
        0 | 1 => 1,
        _ => n * factorial(n - 1),
    }
}

// 런타임 오버헤드 없음
const FACT_10: u64 = factorial(10);  // 3628800

// 조건부 컴파일 최적화
const fn optimize_for_size() -> bool {
    cfg!(feature = "size-opt")
}

const BUFFER_SIZE: usize = if optimize_for_size() {
    256
} else {
    1024
};
```

## Rust 특성 활용
- **컴파일 타임 실행**: 제로 런타임 비용
- **타입 안전**: const 컨텍스트 보장
- **점진적 안정화**: 기능별 게이트
- **최적화**: 상수 폴딩

## 미래 발전 방향
- const 가변 참조
- const 힙 할당
- const 트레이트 특수화
- 더 많은 표준 트레이트 const화

## 관련 개념
- [[032_const_generics]] - const 제네릭
- [[123_const_evaluation]] - const 평가
- [[124_compile_time_computation]] - 컴파일 타임 계산