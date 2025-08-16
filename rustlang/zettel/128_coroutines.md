# Coroutines in Rust

## 핵심 개념
중단 및 재개 가능한 함수로 async/await의 기반 메커니즘

## 기본 코루틴 (제너레이터)
```rust
#![feature(coroutines, coroutine_trait)]
use std::ops::{Coroutine, CoroutineState};
use std::pin::Pin;

let mut generator = || {
    yield 1;
    yield 2;
    yield 3;
    return "done";
};

let mut pinned = Pin::new(&mut generator);
match pinned.as_mut().resume(()) {
    CoroutineState::Yielded(1) => println!("Got 1"),
    _ => panic!(),
}
```

## 상태를 가진 코루틴
```rust
#![feature(coroutines)]

fn fibonacci() -> impl Coroutine<Yield = u64, Return = ()> {
    || {
        let mut a = 0u64;
        let mut b = 1u64;
        
        loop {
            yield a;
            let next = a + b;
            a = b;
            b = next;
        }
    }
}

// 사용
let mut fib = fibonacci();
let mut fib = Pin::new(&mut fib);

for _ in 0..10 {
    match fib.as_mut().resume(()) {
        CoroutineState::Yielded(n) => println!("{}", n),
        CoroutineState::Complete(()) => break,
    }
}
```

## 양방향 통신
```rust
#![feature(coroutines)]

fn interactive() -> impl Coroutine<String, Yield = String, Return = ()> {
    |mut input: String| {
        loop {
            let response = format!("Echo: {}", input);
            input = yield response;
            
            if input == "quit" {
                return;
            }
        }
    }
}

let mut gen = interactive();
let mut gen = Pin::new(&mut gen);

let input = "hello".to_string();
match gen.as_mut().resume(input) {
    CoroutineState::Yielded(response) => println!("{}", response),
    _ => {}
}
```

## async/await 내부 구현
```rust
// async 함수는 실제로 코루틴으로 변환됨
async fn async_function() -> i32 {
    let x = fetch_data().await;
    process(x).await
}

// 대략적으로 이렇게 변환됨:
fn async_function_generator() -> impl Coroutine<Output = i32> {
    || {
        let x = {
            let fut = fetch_data();
            loop {
                match fut.poll() {
                    Poll::Ready(val) => break val,
                    Poll::Pending => yield,
                }
            }
        };
        
        let result = {
            let fut = process(x);
            loop {
                match fut.poll() {
                    Poll::Ready(val) => break val,
                    Poll::Pending => yield,
                }
            }
        };
        
        result
    }
}
```

## 코루틴 상태 머신
```rust
#![feature(coroutines)]

enum State {
    Start,
    Running,
    Paused,
    Finished,
}

fn state_machine() -> impl Coroutine<Yield = State, Return = ()> {
    || {
        yield State::Start;
        
        for i in 0..5 {
            yield State::Running;
            
            if i % 2 == 0 {
                yield State::Paused;
            }
        }
        
        yield State::Finished;
    }
}
```

## 코루틴과 이터레이터
```rust
#![feature(coroutines, iter_from_coroutine)]
use std::iter::from_coroutine;

let generator = || {
    yield 1;
    yield 2;
    yield 3;
};

let iter = from_coroutine(generator);
let vec: Vec<_> = iter.collect();
assert_eq!(vec, vec![1, 2, 3]);
```

## 스트림 구현
```rust
use futures::stream::Stream;

struct CoroutineStream<G> {
    gen: Pin<Box<G>>,
}

impl<G> Stream for CoroutineStream<G>
where
    G: Coroutine<Yield = T, Return = ()>,
{
    type Item = T;
    
    fn poll_next(
        mut self: Pin<&mut Self>,
        cx: &mut Context<'_>,
    ) -> Poll<Option<Self::Item>> {
        match self.gen.as_mut().resume(()) {
            CoroutineState::Yielded(val) => Poll::Ready(Some(val)),
            CoroutineState::Complete(()) => Poll::Ready(None),
        }
    }
}
```

## 코루틴 조합
```rust
#![feature(coroutines)]

fn zip_coroutines<G1, G2>(mut g1: G1, mut g2: G2) 
    -> impl Coroutine<Yield = (G1::Yield, G2::Yield)>
where
    G1: Coroutine,
    G2: Coroutine,
{
    move || {
        let mut g1 = Pin::new(&mut g1);
        let mut g2 = Pin::new(&mut g2);
        
        loop {
            let v1 = match g1.as_mut().resume(()) {
                CoroutineState::Yielded(v) => v,
                _ => return,
            };
            
            let v2 = match g2.as_mut().resume(()) {
                CoroutineState::Yielded(v) => v,
                _ => return,
            };
            
            yield (v1, v2);
        }
    }
}
```

## 에러 처리
```rust
#![feature(coroutines)]

fn fallible_generator() -> impl Coroutine<Yield = Result<i32, &'static str>> {
    || {
        yield Ok(1);
        yield Ok(2);
        yield Err("Something went wrong");
        yield Ok(3);  // 도달하지 않을 수 있음
    }
}

// 사용
let mut gen = fallible_generator();
let mut gen = Pin::new(&mut gen);

loop {
    match gen.as_mut().resume(()) {
        CoroutineState::Yielded(Ok(val)) => println!("Got: {}", val),
        CoroutineState::Yielded(Err(e)) => {
            eprintln!("Error: {}", e);
            break;
        }
        CoroutineState::Complete(()) => break,
    }
}
```

## 코루틴 수명 관리
```rust
struct CoroutineHolder<'a> {
    gen: Pin<Box<dyn Coroutine<Yield = i32, Return = ()> + 'a>>,
}

impl<'a> CoroutineHolder<'a> {
    fn new<G>(gen: G) -> Self
    where
        G: Coroutine<Yield = i32, Return = ()> + 'a,
    {
        CoroutineHolder {
            gen: Box::pin(gen),
        }
    }
    
    fn next(&mut self) -> Option<i32> {
        match self.gen.as_mut().resume(()) {
            CoroutineState::Yielded(val) => Some(val),
            CoroutineState::Complete(()) => None,
        }
    }
}
```

## Rust 특성 활용
- **제로 비용**: 상태 머신으로 컴파일
- **메모리 안전**: 수명 추적
- **타입 안전**: yield/return 타입 체크
- **조합 가능**: 코루틴 합성

## 사용 사례
- 이터레이터 구현
- 상태 머신
- 협력적 멀티태스킹
- 스트림 처리

## 관련 개념
- [[019_async_await]] - async/await
- [[129_generators]] - 제너레이터
- [[130_stackless_coroutines]] - 스택리스 코루틴