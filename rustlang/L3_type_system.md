# L3: 타입 시스템의 힘 - 컴파일 타임에 버그 잡기

## Core Insight
Rust의 타입 시스템은 단순한 분류 체계가 아니다. 프로그램의 불변식(invariant)을 표현하고, 잘못된 상태를 아예 표현할 수 없게 만드는 강력한 도구다. "Make invalid states unrepresentable"가 Rust의 모토다.

---

## Option과 Result - Null의 죽음

### 10억 달러의 실수

Tony Hoare, null의 발명자:
> "I call it my billion-dollar mistake... It has led to innumerable errors, vulnerabilities, and system crashes"

Java에서:
```java
String name = getName();  // null일 수도...
int length = name.length();  // NullPointerException!
```

Rust의 해답:
```rust
fn get_name() -> Option<String> {
    // Some(String) 또는 None
}

fn main() {
    let name = get_name();
    // let length = name.len();  // 컴파일 에러! Option은 len() 없음
    
    // 명시적으로 처리해야 함
    match name {
        Some(n) => println!("이름: {}", n),
        None => println!("이름 없음"),
    }
}
```

### Option<T> - 있을 수도, 없을 수도

```rust
enum Option<T> {
    Some(T),
    None,
}
```

실전 활용:
```rust
struct User {
    id: u32,
    email: String,
    phone: Option<String>,  // 선택적 필드
}

impl User {
    fn send_sms(&self, message: &str) -> Result<(), &str> {
        match &self.phone {
            Some(number) => {
                println!("{}로 SMS 전송: {}", number, message);
                Ok(())
            }
            None => Err("전화번호 없음"),
        }
    }
}
```

### Result<T, E> - 성공 또는 실패

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

파일 읽기 예제:
```rust
use std::fs::File;
use std::io::Read;

fn read_username_from_file() -> Result<String, std::io::Error> {
    let mut file = File::open("user.txt")?;  // ? 연산자!
    let mut username = String::new();
    file.read_to_string(&mut username)?;
    Ok(username)
}
```

### ? 연산자의 마법

```rust
// ? 연산자 없이
fn verbose_version() -> Result<String, std::io::Error> {
    let file = match File::open("user.txt") {
        Ok(f) => f,
        Err(e) => return Err(e),
    };
    // ... 더 많은 match ...
}

// ? 연산자로
fn concise_version() -> Result<String, std::io::Error> {
    let mut file = File::open("user.txt")?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    Ok(contents)
}

// 체이닝
fn one_liner() -> Result<String, std::io::Error> {
    std::fs::read_to_string("user.txt")
}
```

## 패턴 매칭 - if-else의 진화

### match의 완전성

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(u8, u8, u8),
}

fn process_message(msg: Message) {
    match msg {
        Message::Quit => println!("종료"),
        Message::Move { x, y } => println!("이동: ({}, {})", x, y),
        Message::Write(text) => println!("쓰기: {}", text),
        Message::ChangeColor(r, g, b) => println!("색상: #{:02x}{:02x}{:02x}", r, g, b),
    }
    // 모든 경우를 처리해야 컴파일됨!
}
```

### 패턴의 종류

```rust
let x = Some(5);

// 리터럴 패턴
match x {
    Some(5) => println!("정확히 5"),
    Some(n) => println!("다른 숫자: {}", n),
    None => println!("없음"),
}

// 범위 패턴
let score = 85;
match score {
    0..=59 => println!("F"),
    60..=69 => println!("D"),
    70..=79 => println!("C"),
    80..=89 => println!("B"),
    90..=100 => println!("A"),
    _ => println!("잘못된 점수"),
}

// 구조체 분해
struct Point { x: i32, y: i32 }
let p = Point { x: 0, y: 7 };

match p {
    Point { x: 0, y } => println!("y축 위의 점: {}", y),
    Point { x, y: 0 } => println!("x축 위의 점: {}", x),
    Point { x, y } => println!("일반 점: ({}, {})", x, y),
}
```

### if let과 while let

```rust
// match가 과한 경우
let config_max = Some(3u8);

// match 버전
match config_max {
    Some(max) => println!("최대값: {}", max),
    _ => (),
}

// if let 버전 (더 간결)
if let Some(max) = config_max {
    println!("최대값: {}", max);
}

// while let으로 반복
let mut stack = vec![1, 2, 3];
while let Some(top) = stack.pop() {
    println!("{}", top);
}
```

## 트레이트 - 덕 타이핑의 정적 버전

### 트레이트 정의와 구현

```rust
trait Animal {
    fn name(&self) -> &str;
    fn speak(&self) {  // 기본 구현
        println!("{} makes a sound", self.name());
    }
}

struct Dog {
    name: String,
}

struct Cat {
    name: String,
}

impl Animal for Dog {
    fn name(&self) -> &str {
        &self.name
    }
    
    fn speak(&self) {
        println!("{} barks!", self.name());
    }
}

impl Animal for Cat {
    fn name(&self) -> &str {
        &self.name
    }
    // speak는 기본 구현 사용
}
```

### 트레이트 바운드

```rust
// 제네릭 함수에 제약 추가
fn notify<T: Display + Clone>(item: &T) {
    println!("Breaking news: {}", item);
}

// where 절로 복잡한 바운드 표현
fn complex_function<T, U>(t: &T, u: &U) -> String
where
    T: Display + Clone,
    U: Clone + Debug,
{
    format!("{:?}", u)
}
```

### 표준 트레이트들

```rust
// Display - 사용자 친화적 출력
impl Display for Point {
    fn fmt(&self, f: &mut Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}

// Debug - 디버깅용 출력
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

// PartialEq - 동등성 비교
#[derive(PartialEq)]
struct Version {
    major: u32,
    minor: u32,
    patch: u32,
}
```

## 제네릭 - 하나의 코드, 모든 타입

### 제네릭 함수

```rust
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    
    largest
}

fn main() {
    let numbers = vec![34, 50, 25, 100, 65];
    let chars = vec!['y', 'm', 'a', 'q'];
    
    println!("최대 숫자: {}", largest(&numbers));
    println!("최대 문자: {}", largest(&chars));
}
```

### 제네릭 구조체

```rust
struct Point<T> {
    x: T,
    y: T,
}

// 다른 타입도 가능
struct MixedPoint<T, U> {
    x: T,
    y: U,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

// 특정 타입에만 메서드 추가
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

## 고급 타입 기법

### 타입 별칭

```rust
type Kilometers = i32;
type Result<T> = std::result::Result<T, std::io::Error>;

fn read_file() -> Result<String> {
    std::fs::read_to_string("file.txt")
}
```

### 뉴타입 패턴

```rust
struct Meters(f64);
struct Feet(f64);

impl Meters {
    fn to_feet(&self) -> Feet {
        Feet(self.0 * 3.28084)
    }
}

fn main() {
    let height = Meters(1.8);
    // let wrong = height + Feet(5.0);  // 컴파일 에러! 타입 안전
    let height_ft = height.to_feet();
}
```

### 팬텀 타입

```rust
use std::marker::PhantomData;

struct Id<T> {
    value: u32,
    _phantom: PhantomData<T>,
}

struct User;
struct Post;

fn get_user(id: Id<User>) -> User {
    // User ID만 받음
    User
}

fn get_post(id: Id<Post>) -> Post {
    // Post ID만 받음
    Post
}

fn main() {
    let user_id = Id::<User> { value: 1, _phantom: PhantomData };
    let post_id = Id::<Post> { value: 1, _phantom: PhantomData };
    
    let user = get_user(user_id);
    // let post = get_post(user_id);  // 컴파일 에러! 타입 불일치
}
```

## 상태 기계를 타입으로

```rust
// 타입으로 상태 표현
struct Draft;
struct PendingReview;
struct Published;

struct Post<State> {
    content: String,
    state: PhantomData<State>,
}

impl Post<Draft> {
    fn new() -> Self {
        Post {
            content: String::new(),
            state: PhantomData,
        }
    }
    
    fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
    
    fn request_review(self) -> Post<PendingReview> {
        Post {
            content: self.content,
            state: PhantomData,
        }
    }
}

impl Post<PendingReview> {
    fn approve(self) -> Post<Published> {
        Post {
            content: self.content,
            state: PhantomData,
        }
    }
    
    fn reject(self) -> Post<Draft> {
        Post {
            content: self.content,
            state: PhantomData,
        }
    }
}

impl Post<Published> {
    fn content(&self) -> &str {
        &self.content
    }
}

fn main() {
    let mut post = Post::<Draft>::new();
    post.add_text("블로그 내용");
    
    let post = post.request_review();
    // post.add_text("더 추가");  // 컴파일 에러! PendingReview는 수정 불가
    
    let post = post.approve();
    println!("게시됨: {}", post.content());
}
```

## 트레이트 오브젝트와 동적 디스패치

```rust
trait Draw {
    fn draw(&self);
}

struct Button {
    label: String,
}

struct TextField {
    placeholder: String,
}

impl Draw for Button {
    fn draw(&self) {
        println!("버튼: {}", self.label);
    }
}

impl Draw for TextField {
    fn draw(&self) {
        println!("텍스트필드: {}", self.placeholder);
    }
}

// 동적 디스패치
struct Screen {
    components: Vec<Box<dyn Draw>>,
}

impl Screen {
    fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

## 연관 타입

```rust
trait Iterator {
    type Item;  // 연관 타입
    
    fn next(&mut self) -> Option<Self::Item>;
}

struct Counter {
    count: u32,
}

impl Iterator for Counter {
    type Item = u32;  // 구체적 타입 지정
    
    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;
        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}
```

## 마무리: 타입이 주는 자유

Rust의 타입 시스템은 제약이 아니라 자유다:

1. **불가능을 불가능하게**: 잘못된 상태를 아예 표현할 수 없음
2. **의도를 코드로**: 타입이 곧 문서
3. **리팩토링의 자신감**: 컴파일되면 작동한다
4. **성능 보장**: 제로 코스트 추상화

다음 장에서는 이 타입 시스템이 어떻게 "Fearless Concurrency"를 가능하게 하는지 살펴볼 것이다.

---

## Connections
→ [[L4_concurrency]] - 타입으로 보장하는 동시성 안전성
→ [[zettel/009_trait_system]] - 트레이트 심화
→ [[zettel/014_result_type]] - Result 패턴 상세
→ [[zettel/016_pattern_matching]] - 패턴 매칭 고급
← [[L2_ownership]] - 소유권과 타입

---

Level: L3 (설계 원리)
Date: 2025-08-15
Tags: #rust #type-system #option #result #traits #generics