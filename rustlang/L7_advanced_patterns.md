# L7: 고급 패턴과 최적화 - 전문가의 비밀

## Core Insight
Rust의 고급 기능들은 단순한 문법적 설탕이 아니다. Zero-cost abstractions, unsafe의 올바른 사용, 매크로 메타프로그래밍은 시스템 프로그래밍의 한계를 넘어서는 도구들이다.

---

## Zero-Cost Abstractions의 실체

### 추상화 비용이 0이라는 의미

C++ 창시자 Bjarne Stroustrup:
> "What you don't use, you don't pay for. And further: What you do use, you couldn't hand code any better."

Rust에서의 예:

```rust
// 고수준 추상화
let sum: i32 = (1..=1000000)
    .filter(|x| x % 2 == 0)
    .map(|x| x * x)
    .sum();

// 컴파일러가 생성하는 코드 (개념적)
let mut sum = 0i32;
for x in 1..=1000000 {
    if x % 2 == 0 {
        sum += x * x;
    }
}
```

### 실제 어셈블리 비교

```rust
// Iterator 버전
pub fn iter_sum(v: &[i32]) -> i32 {
    v.iter().sum()
}

// 수동 루프 버전
pub fn manual_sum(v: &[i32]) -> i32 {
    let mut sum = 0;
    for i in 0..v.len() {
        sum += v[i];
    }
    sum
}

// 생성되는 어셈블리는 거의 동일!
// SIMD 명령어 사용, 루프 언롤링 등 최적화 적용
```

### 제네릭 단형화 (Monomorphization)

```rust
fn generic_function<T: Display>(val: T) {
    println!("{}", val);
}

// 사용
generic_function(42);      // generic_function_i32 생성
generic_function("hello"); // generic_function_str 생성

// 컴파일러가 실제로 생성하는 코드
fn generic_function_i32(val: i32) {
    println!("{}", val);
}

fn generic_function_str(val: &str) {
    println!("{}", val);
}
```

## Unsafe - 금지된 마법

### Unsafe가 허용하는 5가지

```rust
unsafe {
    // 1. 원시 포인터 역참조
    let raw_ptr = &mut 10 as *mut i32;
    *raw_ptr = 20;
    
    // 2. unsafe 함수/메서드 호출
    let vec = Vec::with_capacity(10);
    vec.set_len(5);  // unsafe!
    
    // 3. unsafe 트레이트 구현 접근
    // Send, Sync 등
    
    // 4. 가변 정적 변수 접근
    static mut COUNTER: i32 = 0;
    COUNTER += 1;
    
    // 5. union 필드 접근
}
```

### 안전한 추상화 구축

```rust
use std::slice;

fn split_at_mut<T>(slice: &mut [T], mid: usize) -> (&mut [T], &mut [T]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr();
    
    assert!(mid <= len);
    
    unsafe {
        (
            slice::from_raw_parts_mut(ptr, mid),
            slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}

// 사용자는 안전하게 사용
let mut data = vec![1, 2, 3, 4, 5];
let (left, right) = split_at_mut(&mut data, 2);
```

### FFI - C와의 상호작용

```rust
extern "C" {
    fn abs(input: i32) -> i32;
    fn sqrt(input: f64) -> f64;
}

// Rust 함수를 C에서 호출 가능하게
#[no_mangle]
pub extern "C" fn rust_function(x: i32) -> i32 {
    x * 2
}

// 안전한 래퍼
pub fn safe_sqrt(x: f64) -> Option<f64> {
    if x >= 0.0 {
        Some(unsafe { sqrt(x) })
    } else {
        None
    }
}
```

## 매크로 - 코드를 생성하는 코드

### 선언적 매크로 (macro_rules!)

```rust
macro_rules! vec_of_strings {
    ($($x:expr),*) => {
        vec![$(String::from($x)),*]
    };
}

let fruits = vec_of_strings!["apple", "banana", "orange"];

// 재귀적 매크로
macro_rules! count_items {
    () => (0);
    ($head:tt $($tail:tt)*) => (1 + count_items!($($tail)*));
}

let count = count_items!(a b c d);  // 4
```

### DSL 구축

```rust
macro_rules! html {
    (div { $($content:tt)* }) => {
        format!("<div>{}</div>", html!($($content)*))
    };
    (p { $text:expr }) => {
        format!("<p>{}</p>", $text)
    };
    ($text:expr) => {
        $text.to_string()
    };
}

let page = html! {
    div {
        p { "Hello, World!" }
    }
};
```

### 절차적 매크로

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(MyTrait)]
pub fn my_trait_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let name = input.ident;
    
    let expanded = quote! {
        impl MyTrait for #name {
            fn my_method(&self) {
                println!("MyTrait for {}", stringify!(#name));
            }
        }
    };
    
    TokenStream::from(expanded)
}
```

## Const Generics - 컴파일 타임 계산

```rust
// 고정 크기 배열 추상화
struct Matrix<T, const M: usize, const N: usize> {
    data: [[T; N]; M],
}

impl<T: Default + Copy, const M: usize, const N: usize> Matrix<T, M, N> {
    fn new() -> Self {
        Matrix {
            data: [[T::default(); N]; M],
        }
    }
    
    fn transpose(self) -> Matrix<T, N, M> {
        let mut result = Matrix::<T, N, M>::new();
        for i in 0..M {
            for j in 0..N {
                result.data[j][i] = self.data[i][j];
            }
        }
        result
    }
}

// 컴파일 타임에 크기 체크!
let m1: Matrix<i32, 3, 4> = Matrix::new();
let m2: Matrix<i32, 4, 3> = m1.transpose();
```

## GAT - Generic Associated Types

```rust
trait Container {
    type Item<'a> where Self: 'a;
    
    fn get<'a>(&'a self, index: usize) -> Option<Self::Item<'a>>;
}

struct VecContainer<T> {
    data: Vec<T>,
}

impl<T> Container for VecContainer<T> {
    type Item<'a> = &'a T where Self: 'a;
    
    fn get<'a>(&'a self, index: usize) -> Option<Self::Item<'a>> {
        self.data.get(index)
    }
}
```

## 고급 생명주기

### Higher-Ranked Trait Bounds (HRTB)

```rust
fn higher_ranked<F>(f: F) 
where
    F: for<'a> Fn(&'a str) -> &'a str,  // 모든 생명주기에 대해
{
    let s = String::from("hello");
    let result = f(&s);
    println!("{}", result);
}

// 사용
higher_ranked(|s| &s[..2]);
```

### 생명주기 서브타이핑

```rust
fn longest<'a, 'b: 'a>(x: &'a str, y: &'b str) -> &'a str {
    // 'b는 최소한 'a만큼 살아있어야 함
    if x.len() > y.len() { x } else { y }
}
```

## 최적화 기법들

### 인라인 힌트

```rust
#[inline]
fn small_function() -> i32 {
    42
}

#[inline(always)]  // 강제 인라인
fn critical_path() -> i32 {
    // 핫 패스 코드
}

#[inline(never)]  // 인라인 금지
fn large_function() {
    // 큰 함수
}
```

### SIMD 활용

```rust
use std::arch::x86_64::*;

unsafe fn sum_simd(data: &[f32]) -> f32 {
    let mut sum = _mm_setzero_ps();
    
    let chunks = data.chunks_exact(4);
    let remainder = chunks.remainder();
    
    for chunk in chunks {
        let vals = _mm_loadu_ps(chunk.as_ptr());
        sum = _mm_add_ps(sum, vals);
    }
    
    // 수평 합
    sum = _mm_hadd_ps(sum, sum);
    sum = _mm_hadd_ps(sum, sum);
    
    let mut result = 0.0f32;
    _mm_store_ss(&mut result, sum);
    
    // 나머지 처리
    result + remainder.iter().sum::<f32>()
}
```

### 메모리 레이아웃 최적화

```rust
// 기본 레이아웃 (패딩 있음)
struct Unoptimized {
    a: u8,   // 1 byte + 7 padding
    b: u64,  // 8 bytes
    c: u8,   // 1 byte + 7 padding
}  // 총 24 bytes

// 최적화된 레이아웃
#[repr(C)]
struct Optimized {
    b: u64,  // 8 bytes
    a: u8,   // 1 byte
    c: u8,   // 1 byte + 6 padding
}  // 총 16 bytes

// 패킹
#[repr(packed)]
struct Packed {
    a: u8,   // 1 byte
    b: u64,  // 8 bytes
    c: u8,   // 1 byte
}  // 총 10 bytes (정렬 없음, 느릴 수 있음)
```

## 커스텀 알로케이터

```rust
use std::alloc::{GlobalAlloc, Layout};

struct MyAllocator;

unsafe impl GlobalAlloc for MyAllocator {
    unsafe fn alloc(&self, layout: Layout) -> *mut u8 {
        // 커스텀 할당 로직
        std::alloc::alloc(layout)
    }
    
    unsafe fn dealloc(&self, ptr: *mut u8, layout: Layout) {
        // 커스텀 해제 로직
        std::alloc::dealloc(ptr, layout)
    }
}

#[global_allocator]
static ALLOCATOR: MyAllocator = MyAllocator;
```

## 비동기 최적화

```rust
use futures::stream::{self, StreamExt};

async fn parallel_fetch(urls: Vec<String>) -> Vec<Result<String, Error>> {
    stream::iter(urls)
        .map(|url| async move {
            fetch_url(&url).await
        })
        .buffer_unordered(10)  // 최대 10개 동시 실행
        .collect()
        .await
}

// 비동기 뮤텍스 최적화
use tokio::sync::RwLock;
use std::sync::Arc;

struct Cache {
    data: Arc<RwLock<HashMap<String, String>>>,
}

impl Cache {
    async fn get_or_compute(&self, key: &str) -> String {
        // 먼저 읽기 시도 (빠름)
        {
            let data = self.data.read().await;
            if let Some(value) = data.get(key) {
                return value.clone();
            }
        }
        
        // 없으면 쓰기 (느림, 하지만 필요할 때만)
        let mut data = self.data.write().await;
        data.entry(key.to_string())
            .or_insert_with(|| compute_value(key))
            .clone()
    }
}
```

## const fn - 컴파일 타임 계산

```rust
const fn fibonacci(n: u32) -> u32 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}

const FIB_10: u32 = fibonacci(10);  // 컴파일 타임에 계산!

// const 제네릭과 함께
struct Array<T, const N: usize> {
    data: [T; N],
}

impl<T, const N: usize> Array<T, N> {
    const fn len(&self) -> usize {
        N
    }
}
```

## 마무리: 고급 기능의 철학

고급 패턴들이 주는 힘:
1. **성능과 추상화의 양립**: Zero-cost로 고수준 코드 작성
2. **안전한 시스템 프로그래밍**: unsafe를 안전하게 감싸기
3. **메타프로그래밍**: 반복 제거, DSL 구축
4. **컴파일 타임 보장**: const fn과 제네릭으로 런타임 오버헤드 제거

이 모든 기능은 "더 안전하고, 더 빠르고, 더 표현력 있는" 코드를 위해 존재한다.

---

## Connections
→ [[L8_production]] - 프로덕션 배포 준비
→ [[zettel/022_zero_cost_abstractions]] - Zero-cost 심화
→ [[zettel/021_unsafe_rust]] - Unsafe 상세
→ [[zettel/032_const_generics]] - Const generics 심화
← [[L6_ecosystem]] - 생태계와 도구

---

Level: L7 (고급 기법)
Date: 2025-08-15
Tags: #rust #advanced #unsafe #macros #optimization