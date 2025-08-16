# L1: Rust 첫 걸음 - Hello World부터 시작하는 여정

## Core Insight
모든 Rust 프로그램은 작은 의식(ritual)으로 시작한다. `cargo new`, `fn main()`, `println!`. 이 간단한 주문들 속에 Rust의 철학이 모두 담겨 있다.

---

## Cargo New의 마법

터미널을 열고 타이핑한다:

```bash
$ cargo new hello_rust
     Created binary (application) `hello_rust` package
```

무슨 일이 일어났을까?

```
hello_rust/
├── Cargo.toml      # 프로젝트의 영혼
├── src/
│   └── main.rs     # 당신의 캔버스
└── .gitignore      # Git을 위한 배려
```

### Cargo.toml - 프로젝트의 신분증

```toml
[package]
name = "hello_rust"
version = "0.1.0"
edition = "2021"

[dependencies]
```

이 몇 줄이 말하는 것:
- **name**: 당신의 창조물의 이름
- **version**: 시맨틱 버저닝 (0.1.0 = 아직 실험 중)
- **edition**: Rust 2021 문법 사용 (3년마다 업데이트)
- **dependencies**: 거인의 어깨 위에 서기

다른 언어들과 비교:
- C: Makefile 지옥
- C++: CMake 악몽
- Python: requirements.txt + setup.py + pipenv + ...
- Rust: 그냥 Cargo.toml

## 첫 번째 프로그램

```rust
fn main() {
    println!("Hello, world!");
}
```

실행:
```bash
$ cargo run
   Compiling hello_rust v0.1.0
    Finished dev [unoptimized + debuginfo] target(s) in 0.54s
     Running `target/debug/hello_rust`
Hello, world!
```

### fn main() - 모든 것의 시작

```rust
fn main() {
    // 여기가 우주의 중심
}
```

왜 `main`일까? 
- OS가 프로그램을 시작할 때 찾는 약속된 이름
- C에서 물려받은 전통
- 하지만 Rust의 main은 특별하다: 반환 타입이 `()` (unit type)

다른 형태의 main:

```rust
// Result를 반환하는 main
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let file = std::fs::read_to_string("없는파일.txt")?;
    Ok(())
}

// async main (tokio 사용)
#[tokio::main]
async fn main() {
    // 비동기 세계로
}
```

## println! - 매크로의 첫 만남

왜 `println!`에 느낌표가 있을까?

```rust
println!("Hello");           // 매크로
println("Hello");            // 이런 함수는 없다
print!("World");            // 개행 없음
eprintln!("Error!");        // stderr로 출력
dbg!(&my_variable);         // 디버깅용
```

### 매크로 vs 함수

함수의 한계:
```rust
// 불가능: 가변 개수 인자
fn my_print(format: &str, args: ???) { 
    // 어떻게 구현?
}
```

매크로의 힘:
```rust
println!("이름: {}, 나이: {}", name, age);
println!("벡터: {:?}", vec![1, 2, 3]);
println!("16진수: {:#x}", 255);
```

컴파일 타임에 매크로가 확장된다:
```rust
// 이 코드가
println!("답: {}", 42);

// 이렇게 변환됨 (간소화)
std::io::_print(format_args!("답: {}\n", 42));
```

## 변수와 불변성 - Let의 철학

### 기본은 불변

```rust
fn main() {
    let x = 5;
    println!("x = {}", x);
    x = 6;  // 컴파일 에러!
}
```

```
error[E0384]: cannot assign twice to immutable variable `x`
```

왜 기본이 불변일까?

1. **안전성**: 실수로 값을 바꿀 수 없다
2. **최적화**: 컴파일러가 더 공격적으로 최적화
3. **동시성**: 불변 데이터는 공유해도 안전
4. **명확성**: 변경되는 것만 `mut` 표시

### 가변성이 필요할 때

```rust
fn main() {
    let mut count = 0;
    
    for i in 1..=10 {
        count += i;
    }
    
    println!("합: {}", count);  // 55
}
```

`mut`는 의도의 선언이다: "이 변수는 변할 거야"

### Shadowing - 같은 이름, 다른 변수

```rust
fn main() {
    let x = 5;              // x: i32
    let x = x + 1;          // 새로운 x: i32
    let x = x * 2;          // 또 새로운 x: i32
    let x = "이제 문자열";   // 완전히 다른 타입의 x
    
    println!("x = {}", x);  // "이제 문자열"
}
```

Shadowing vs mut:
- Shadowing: 새 변수 생성, 타입 변경 가능
- mut: 같은 변수 수정, 타입 고정

실전 활용:
```rust
let input = "  42  ";
let input = input.trim();      // &str
let input: i32 = input.parse().unwrap();  // i32
// 같은 이름으로 변환 과정 표현
```

## 타입 시스템 - 첫 만남

### 타입 추론의 아름다움

```rust
let x = 5;           // 컴파일러: "아, i32구나"
let y = 2.5;         // 컴파일러: "f64네"
let z = true;        // 컴파일러: "bool이군"
let s = "hello";     // 컴파일러: "&str이야"
```

하지만 때로는 명시가 필요:

```rust
let guess: u32 = "42".parse().unwrap();
//        ^^^^ 이게 없으면 컴파일러가 헷갈림
```

### 기본 타입들

```rust
// 정수 타입 (기본: i32)
let a: i8 = -128;          // -128 ~ 127
let b: u8 = 255;           // 0 ~ 255
let c: i32 = -2_147_483_648;  // 언더스코어로 읽기 쉽게
let d: u64 = 18_446_744_073_709_551_615;

// 부동소수점 (기본: f64)
let pi: f32 = 3.141592;
let e: f64 = 2.718281828459045;

// 불린
let is_rust_awesome = true;

// 문자 (유니코드!)
let heart = '❤';
let crab = '🦀';  // Rust의 마스코트
```

### 타입 안전성의 철벽

```rust
fn main() {
    let x: i32 = 5;
    let y: i64 = 10;
    let z = x + y;  // 컴파일 에러!
}
```

```
error[E0308]: mismatched types
```

다른 언어들은 암묵적 변환을 한다. Rust는 거부한다:

```rust
let z = x + y as i32;  // 명시적 캐스팅
// 또는
let z = x as i64 + y;  // 더 안전한 방향으로
```

## 첫 번째 함수

```rust
fn add(x: i32, y: i32) -> i32 {
    x + y  // return 키워드 없음!
}

fn main() {
    let result = add(5, 3);
    println!("5 + 3 = {}", result);
}
```

### 표현식 vs 구문

Rust의 비밀: 거의 모든 것이 표현식이다.

```rust
fn get_value() -> i32 {
    let x = {
        let y = 3;
        y + 1  // 세미콜론 없음 = 표현식
    };  // x = 4
    
    if true {
        10
    } else {
        20
    }  // 이것도 표현식, 10 반환
}
```

세미콜론의 마법:
```rust
fn example() -> i32 {
    5     // 표현식: 5를 반환
}

fn example2() -> i32 {
    5;    // 구문: ()를 반환 → 컴파일 에러!
}
```

## 제어 흐름

### if - 표현식으로서의 조건문

```rust
let number = 7;

// 전통적 방식
if number < 5 {
    println!("작다");
} else if number > 10 {
    println!("크다");
} else {
    println!("적당하다");
}

// 표현식으로 사용
let description = if number < 5 {
    "작은 수"
} else if number > 10 {
    "큰 수"
} else {
    "중간 수"
};
```

### loop - 무한의 시작

```rust
let mut counter = 0;

let result = loop {
    counter += 1;
    
    if counter == 10 {
        break counter * 2;  // 값과 함께 탈출
    }
};

println!("결과: {}", result);  // 20
```

### for - 안전한 반복

```rust
// 범위 반복
for i in 1..=5 {
    println!("{}", i);  // 1, 2, 3, 4, 5
}

// 컬렉션 반복
let arr = [10, 20, 30];
for element in arr {
    println!("{}", element);
}

// 인덱스와 함께
for (i, val) in arr.iter().enumerate() {
    println!("arr[{}] = {}", i, val);
}
```

## 첫 번째 구조체

```rust
struct User {
    username: String,
    email: String,
    active: bool,
    sign_in_count: u64,
}

fn main() {
    let user1 = User {
        email: String::from("ferris@rust.org"),
        username: String::from("ferris"),
        active: true,
        sign_in_count: 1,
    };
    
    println!("사용자: {}", user1.username);
}
```

### 메서드 추가하기

```rust
impl User {
    // 연관 함수 (static method)
    fn new(username: String, email: String) -> User {
        User {
            username,
            email,  // 필드 초기화 단축 문법
            active: true,
            sign_in_count: 0,
        }
    }
    
    // 메서드
    fn greet(&self) {
        println!("안녕하세요, {}입니다!", self.username);
    }
}

fn main() {
    let user = User::new(
        String::from("ferris"),
        String::from("ferris@rust.org")
    );
    
    user.greet();
}
```

## 에러 처리 입문

### panic! - 복구 불가능한 에러

```rust
fn main() {
    panic!("크래시!");
    println!("이 줄은 실행되지 않음");
}
```

### Result - 복구 가능한 에러

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
    
    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("파일 열기 실패: {:?}", error);
        }
    };
}
```

더 간단하게:
```rust
let f = File::open("hello.txt").unwrap();  // Ok면 값, Err면 panic
let f = File::open("hello.txt").expect("파일이 없습니다");  // 메시지 지정
```

## 마무리: 첫 걸음의 의미

지금까지 배운 것:
- Cargo로 프로젝트 관리
- 기본 문법과 타입
- 불변성 기본 원칙
- 표현식 중심 사고
- 안전한 에러 처리

하지만 이것은 시작일 뿐이다. 다음 장에서는 Rust의 킬러 피처, **소유권**을 만날 것이다.

작은 프로그램 하나:

```rust
use std::io;

fn main() {
    println!("숫자 맞추기 게임!");
    
    let secret = 42;
    
    loop {
        println!("추측한 숫자를 입력하세요:");
        
        let mut guess = String::new();
        io::stdin().read_line(&mut guess)
            .expect("입력 실패");
        
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => {
                println!("숫자를 입력해주세요!");
                continue;
            }
        };
        
        match guess.cmp(&secret) {
            std::cmp::Ordering::Less => println!("너무 작아요!"),
            std::cmp::Ordering::Greater => println!("너무 커요!"),
            std::cmp::Ordering::Equal => {
                println!("정답!");
                break;
            }
        }
    }
}
```

이 간단한 게임에 Rust의 핵심이 모두 들어있다.

---

## Connections
→ [[L2_ownership]] - 이제 진짜 Rust를 배울 시간
→ [[zettel/016_pattern_matching]] - 패턴 매칭의 힘
→ [[zettel/014_result_type]] - 에러 처리의 철학
← [[L0_experience]] - 왜 Rust인가?

---

Level: L1 (기초 요소)
Date: 2025-08-15
Tags: #rust #basics #cargo #types #control-flow