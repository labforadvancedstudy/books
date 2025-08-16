# Effect System in Rust

## 핵심 개념
함수의 부작용을 타입 시스템으로 추적하고 제어하는 메커니즘

## 기본 이펙트 표현
```rust
#![feature(effects)]

// 이펙트 정의
effect async;
effect throws;
effect const;

// 이펙트가 있는 함수
async fn fetch_data() -> String { ... }
const fn compile_time() -> i32 { ... }
fn may_fail() -> Result<i32, Error> { ... }
```

## 이펙트 다형성
```rust
#![feature(effect_polymorphism)]

// 이펙트 제네릭 함수
fn map<T, U, F, effect E>(
    option: Option<T>,
    f: F
) -> Option<U>
where
    F: FnOnce(T) -> U with E,
{
    match option {
        Some(val) => Some(f(val) with E),
        None => None,
    }
}

// async 이펙트와 함께 사용
async fn async_double(x: i32) -> i32 { x * 2 }
let result = map(Some(5), async_double);  // async Option<i32>

// const 이펙트와 함께 사용
const fn const_double(x: i32) -> i32 { x * 2 }
const RESULT: Option<i32> = map(Some(5), const_double);
```

## 이펙트 추론
```rust
// 컴파일러가 이펙트 자동 추론
fn auto_effect(cond: bool) -> i32 {
    if cond {
        async_operation().await  // async 이펙트 추론
    } else {
        42
    }
}

// 명시적 이펙트 표기
fn explicit() -> i32 with async + throws {
    let data = fetch().await?;
    process(data)
}
```

## Try 이펙트
```rust
#![feature(try_trait_v2_residual)]

// Try 이펙트로 에러 전파
fn process_data(input: &str) -> Result<Data, Error> with try {
    let parsed = parse(input)?;  // try 이펙트
    let validated = validate(parsed)?;
    Ok(transform(validated))
}

// Option과 Result 모두 지원
fn find_user(id: u64) -> Option<User> with try {
    let db = get_database()?;
    let user = db.find_user(id)?;
    Some(user)
}
```

## 이펙트 바운드
```rust
// 이펙트 제약
trait Service {
    fn call(&self) -> Response with async + throws;
}

// 이펙트 없는 구현
impl Service for CachedService {
    fn call(&self) -> Response with const {
        self.cached_response.clone()
    }
}

// 여러 이펙트 구현
impl Service for RemoteService {
    fn call(&self) -> Response with async + throws {
        self.http_client.get("/api").await?
    }
}
```

## 이펙트 세트
```rust
// 이펙트 조합 정의
effect IO = async + throws;
effect Pure = const + !throws;

fn io_operation() -> Data with IO {
    let result = fetch().await?;
    process(result)
}

fn pure_computation(x: i32) -> i32 with Pure {
    x * 2 + 1
}
```

## 조건부 이펙트
```rust
// 제네릭 파라미터에 따른 이펙트
fn conditional<T: Display, const ASYNC: bool>() -> String 
    with { if ASYNC { async } else { const } }
{
    if ASYNC {
        fetch_string().await
    } else {
        "static string".to_string()
    }
}
```

## 이펙트 전파
```rust
// 이펙트 자동 전파
fn wrapper<F, T>(f: F) -> T
where
    F: FnOnce() -> T with auto,  // 호출자의 이펙트 상속
{
    println!("Before");
    let result = f() with auto;
    println!("After");
    result
}

// async 함수와 사용
async fn async_work() -> i32 { 42 }
let future = wrapper(async_work);  // async i32

// const 함수와 사용
const fn const_work() -> i32 { 42 }
const VALUE: i32 = wrapper(const_work);  // const 평가
```

## 이펙트 핸들러
```rust
#![feature(effect_handlers)]

// 커스텀 이펙트 핸들러
effect State<T>;

fn with_state<T, F, R>(initial: T, f: F) -> R
where
    F: FnOnce() -> R with State<T>,
{
    handle State<T> {
        get() => initial.clone(),
        set(new) => initial = new,
    } in {
        f()
    }
}

// 사용
fn stateful_computation() -> i32 with State<i32> {
    let current = State::get();
    State::set(current + 1);
    State::get()
}
```

## 이펙트와 트레이트
```rust
// 트레이트 메서드의 이펙트
trait Transform {
    fn transform(&self, input: Data) -> Output with effect;
}

// 구현체별 다른 이펙트
impl Transform for AsyncTransformer {
    fn transform(&self, input: Data) -> Output with async {
        self.async_process(input).await
    }
}

impl Transform for PureTransformer {
    fn transform(&self, input: Data) -> Output with const {
        Self::pure_transform(input)
    }
}
```

## 이펙트 취소
```rust
// no_effect 속성으로 이펙트 제거
#[no_effect(async)]
fn block_on<F: Future>(fut: F) -> F::Output {
    // async를 동기로 변환
    let mut fut = pin!(fut);
    loop {
        match fut.poll(&mut Context::from_waker(noop_waker())) {
            Poll::Ready(val) => return val,
            Poll::Pending => std::thread::yield_now(),
        }
    }
}
```

## Rust 특성 활용
- **타입 안전**: 이펙트 컴파일 타임 체크
- **제로 비용**: 런타임 오버헤드 없음
- **조합 가능**: 이펙트 합성
- **추론**: 자동 이펙트 유도

## 장점
- 부작용 명시적 표현
- 순수 함수 보장
- 에러 처리 통합
- 성능 최적화 힌트

## 관련 개념
- [[019_async_await]] - 비동기 이펙트
- [[122_const_traits]] - const 이펙트
- [[132_algebraic_effects]] - 대수적 이펙트