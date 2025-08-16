# L2: 소유권의 마법 - 메모리 안전성의 비밀

## Core Insight
소유권(Ownership)은 Rust의 영혼이다. 가비지 컬렉터 없이, 수동 메모리 관리 없이, 메모리 안전성을 보장하는 마법. 이 마법의 비밀은 간단한 규칙 세 개에 있다.

---

## 도서관 책 대출 시스템

소유권을 이해하는 가장 좋은 방법은 도서관을 떠올리는 것이다.

```rust
// 도서관에 책이 있다
let book = String::from("러스트 프로그래밍");

// 철수가 책을 빌린다
let 철수_책 = book;

// 영희도 같은 책을 빌리려 한다
let 영희_책 = book;  // 컴파일 에러!
```

현실 도서관의 규칙:
1. 책은 한 번에 한 사람만 빌릴 수 있다
2. 빌린 사람이 반납해야 다음 사람이 빌릴 수 있다
3. 책을 잃어버리면 변상해야 한다

Rust의 소유권 규칙:
1. 각 값은 소유자(owner)라 불리는 변수를 가진다
2. 한 번에 하나의 소유자만 존재할 수 있다
3. 소유자가 스코프를 벗어나면 값은 버려진다(drop)

## 메모리의 두 얼굴: 스택과 힙

### 스택 - 빠른 기억

```rust
fn main() {
    let x = 5;        // 스택에 5 저장
    let y = x;        // 스택에 5 복사
    println!("{}", x);  // OK: x는 여전히 5
}
```

스택의 특징:
- 크기가 컴파일 타임에 결정됨
- 빠른 할당과 해제 (그냥 포인터 이동)
- 간단한 복사 (Copy trait)

### 힙 - 유연한 저장소

```rust
fn main() {
    let s1 = String::from("hello");  // 힙에 "hello" 저장
    let s2 = s1;                     // 소유권 이동!
    println!("{}", s1);               // 컴파일 에러!
}
```

힙의 특징:
- 런타임에 크기 결정
- 느린 할당과 해제 (메모리 관리자 호출)
- 포인터를 통한 접근

### String의 내부 구조

```rust
// String은 실제로 이렇게 생겼다
struct String {
    ptr: *const u8,  // 힙 메모리 주소
    len: usize,      // 현재 길이
    capacity: usize, // 할당된 용량
}
```

그림으로 보면:
```
스택                     힙
┌─────────┐         ┌─────────────┐
│ ptr     │-------->│ h e l l o   │
│ len: 5  │         └─────────────┘
│ cap: 5  │
└─────────┘
```

## Move, Copy, Clone의 삼각관계

### Move - 소유권 이전

```rust
let s1 = String::from("hello");
let s2 = s1;  // move 발생

// 메모리 상태:
// s1: 무효화됨 (더 이상 사용 불가)
// s2: 힙 데이터의 유일한 소유자
```

왜 move가 필요한가?

```rust
// 만약 move가 없다면...
{
    let s1 = String::from("hello");
    let s2 = s1;  // 만약 둘 다 소유자라면?
}  // s1과 s2가 모두 같은 메모리를 해제? Double free!
```

### Copy - 스택 복사

```rust
let x = 5;
let y = x;  // copy 발생
println!("x = {}, y = {}", x, y);  // OK!
```

Copy가 가능한 타입들:
- 모든 정수 타입 (i32, u64, ...)
- 부동소수점 (f32, f64)
- bool
- char
- Copy를 구현한 튜플 (예: (i32, i32))

### Clone - 명시적 복사

```rust
let s1 = String::from("hello");
let s2 = s1.clone();  // 힙 데이터까지 복사
println!("s1 = {}, s2 = {}", s1, s2);  // OK!
```

Clone의 비용:
```rust
let big_vec = vec![0; 1_000_000];
let copy = big_vec.clone();  // 100만 개 요소 복사!
// 성능을 생각하고 명시적으로 선택
```

## Drop의 우아한 춤

### RAII - Resource Acquisition Is Initialization

```rust
fn create_and_destroy() {
    let s = String::from("hello");
    // s 사용
}  // 여기서 자동으로 drop 호출, 메모리 해제
```

Drop trait:
```rust
struct CustomBox<T> {
    data: T,
}

impl<T> Drop for CustomBox<T> {
    fn drop(&mut self) {
        println!("CustomBox 해제 중...");
        // 정리 작업
    }
}

fn main() {
    let _box = CustomBox { data: 5 };
}  // "CustomBox 해제 중..." 출력
```

### 소멸 순서

```rust
fn main() {
    let x = String::from("첫 번째");
    let y = String::from("두 번째");
    let z = String::from("세 번째");
}  // drop 순서: z → y → x (역순!)
```

## 함수와 소유권

### 소유권 가져가기

```rust
fn take_ownership(s: String) {
    println!("{}", s);
}  // s가 여기서 drop

fn main() {
    let s = String::from("hello");
    take_ownership(s);
    // println!("{}", s);  // 에러! s는 이미 이동됨
}
```

### 소유권 돌려주기

```rust
fn give_ownership() -> String {
    let s = String::from("hello");
    s  // 소유권 반환
}

fn take_and_give_back(s: String) -> String {
    s  // 받은 걸 그대로 돌려줌
}
```

## 참조와 빌림

### 불변 참조 (&T)

```rust
fn calculate_length(s: &String) -> usize {
    s.len()
}  // s는 drop되지 않음, 빌린 것일 뿐

fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);  // 빌려주기
    println!("길이: {}, 원본: {}", len, s1);  // s1 여전히 사용 가능!
}
```

### 가변 참조 (&mut T)

```rust
fn change(s: &mut String) {
    s.push_str(", world");
}

fn main() {
    let mut s = String::from("hello");
    change(&mut s);
    println!("{}", s);  // "hello, world"
}
```

### 참조 규칙 - Rust의 황금률

```rust
let mut s = String::from("hello");

// 규칙 1: 여러 개의 불변 참조 OK
let r1 = &s;
let r2 = &s;
println!("{}, {}", r1, r2);

// 규칙 2: 하나의 가변 참조만 가능
let r3 = &mut s;
// let r4 = &mut s;  // 에러!

// 규칙 3: 가변과 불변 참조 동시 불가
let r5 = &s;
// let r6 = &mut s;  // 에러!
```

이 규칙이 방지하는 것:
1. **데이터 레이스**: 동시에 쓰기 불가능
2. **반복자 무효화**: 순회 중 수정 불가능
3. **댕글링 참조**: 죽은 메모리 참조 불가능

## 슬라이스 - 부분 빌림

```rust
let s = String::from("hello world");

let hello = &s[0..5];   // "hello"
let world = &s[6..11];  // "world"

// 더 일반적인 문자열 슬라이스
let hello: &str = &s[..5];  // 처음부터 5까지
let world: &str = &s[6..];  // 6부터 끝까지
```

배열 슬라이스:
```rust
let a = [1, 2, 3, 4, 5];
let slice = &a[1..3];  // [2, 3]
```

## 실전 예제: 문자열 파서

```rust
struct Parser {
    input: String,
    position: usize,
}

impl Parser {
    fn new(input: String) -> Self {
        Parser { input, position: 0 }
    }
    
    // 다음 토큰 읽기 (빌림)
    fn peek(&self) -> Option<&str> {
        let remaining = &self.input[self.position..];
        remaining.split_whitespace().next()
    }
    
    // 토큰 소비하기 (가변 빌림)
    fn consume(&mut self) -> Option<String> {
        if let Some(token) = self.peek() {
            let token_owned = token.to_string();
            self.position += token.len();
            Some(token_owned)
        } else {
            None
        }
    }
}
```

## 생명주기 맛보기

```rust
// 컴파일러가 혼란스러워하는 경우
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() { x } else { y }
}  // 에러: 반환값이 x인지 y인지 모름

// 생명주기 명시
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

`'a`는 "최소한 'a만큼 살아있음"을 의미한다.

## 소유권 패턴 실전

### Builder 패턴

```rust
struct ServerBuilder {
    host: Option<String>,
    port: Option<u16>,
}

impl ServerBuilder {
    fn new() -> Self {
        ServerBuilder { host: None, port: None }
    }
    
    fn host(mut self, host: String) -> Self {
        self.host = Some(host);
        self  // 소유권 반환 (메서드 체이닝)
    }
    
    fn port(mut self, port: u16) -> Self {
        self.port = Some(port);
        self
    }
    
    fn build(self) -> Server {
        Server {
            host: self.host.unwrap_or_else(|| "localhost".to_string()),
            port: self.port.unwrap_or(8080),
        }
    }
}
```

### Interior Mutability 맛보기

```rust
use std::cell::RefCell;

struct Counter {
    value: RefCell<i32>,
}

impl Counter {
    fn new() -> Self {
        Counter { value: RefCell::new(0) }
    }
    
    fn increment(&self) {  // &self, not &mut self!
        *self.value.borrow_mut() += 1;
    }
}
```

## 소유권이 주는 보장

1. **메모리 안전성**: 세그폴트 없음
2. **스레드 안전성**: 데이터 레이스 없음
3. **예측 가능성**: 언제 메모리가 해제되는지 정확히 앎
4. **성능**: 가비지 컬렉터 오버헤드 없음

## 마무리: 소유권의 선물

처음엔 소유권이 족쇄처럼 느껴진다. 하지만 시간이 지나면 깨닫는다:

```rust
// C++에서
void process() {
    Resource* r = new Resource();
    // ... 복잡한 로직 ...
    // delete r를 잊어버림! 메모리 누수!
}

// Rust에서
fn process() {
    let r = Resource::new();
    // ... 복잡한 로직 ...
}  // 자동으로 정리됨, 잊을 수 없음!
```

소유권은 제약이 아니라 해방이다. 메모리 관리의 부담에서, 버그 찾기의 고통에서, 동시성의 공포에서 우리를 해방시킨다.

다음 장에서는 이 소유권 시스템 위에 구축된 Rust의 강력한 타입 시스템을 탐험할 것이다.

---

## Connections
→ [[L3_type_system]] - 타입으로 불변식 표현하기
→ [[zettel/001_ownership]] - 소유권 원자 개념
→ [[zettel/002_borrowing]] - 빌림의 세부 규칙
→ [[zettel/005_lifetime]] - 생명주기 심화
← [[L1_first_steps]] - 기초 문법

---

Level: L2 (핵심 메커니즘)
Date: 2025-08-15
Tags: #rust #ownership #memory-safety #borrowing #lifetime