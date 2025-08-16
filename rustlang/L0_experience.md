# L0: 프로그래밍의 경험 - 왜 Rust인가?

## Core Insight
프로그래밍은 인간의 의도를 기계가 이해할 수 있는 명령으로 번역하는 행위다. 40년간 이 번역 과정에서 우리는 메모리 안전성과 성능 중 하나를 포기해야 했다. Rust는 "둘 다 가능하다"고 말한다.

---

## 세그폴트와의 전쟁

2015년 여름, 나는 C++로 게임 엔진을 만들고 있었다. 

```cpp
// 악명 높은 코드
Player* player = getPlayer(id);
// ... 100줄의 코드 ...
player->updatePosition();  // 💥 Segmentation fault
```

디버거를 켜고, 브레이크포인트를 찍고, 메모리를 추적했다. 6시간 후 발견한 버그: 다른 스레드에서 player를 delete했다. 

이런 경험이 있는가? 
- 금요일 저녁, 프로덕션에서 터지는 세그폴트
- valgrind가 빨간색으로 물든 화면
- "use after free", "double free", "buffer overflow"
- 고객이 신고하기 전까지 모르는 메모리 누수

## "아빠, 왜 프로그램이 죽어요?"

딸아이가 만든 첫 C 프로그램:

```c
char name[10];
printf("이름을 입력하세요: ");
scanf("%s", name);  // "김수한무거북이와두루미" 입력
printf("안녕하세요 %s님!\n", name);
// 💥 Buffer overflow
```

어떻게 설명해야 할까? "컴퓨터는 네가 10글자만 쓸 거라고 믿었는데, 넌 그 약속을 어겼어." 

하지만 왜 컴퓨터는 스스로를 보호하지 못할까? 왜 프로그래머가 모든 것을 기억해야 할까?

## 첫 번째 Rust 프로그램

```rust
fn main() {
    let mut name = String::new();
    println!("이름을 입력하세요: ");
    std::io::stdin().read_line(&mut name).unwrap();
    println!("안녕하세요 {}님!", name.trim());
}
```

무슨 일이 일어났나?
- String은 자동으로 크기가 늘어난다
- 메모리는 자동으로 해제된다
- 버퍼 오버플로우? 불가능하다
- 세그폴트? 컴파일이 안 된다

## 컴파일러가 친구가 되는 순간

처음에는 Rust 컴파일러가 적처럼 느껴진다:

```rust
fn main() {
    let s = String::from("hello");
    let s2 = s;
    println!("{}", s);  // 컴파일 에러!
}
```

```
error[E0382]: borrow of moved value: `s`
 --> src/main.rs:4:20
  |
2 |     let s = String::from("hello");
  |         - move occurs because `s` has type `String`
3 |     let s2 = s;
  |              - value moved here
4 |     println!("{}", s);
  |                    ^ value borrowed here after move
```

처음엔 화가 났다. "왜 내 코드를 실행도 안 해보고 거부하는 거야?"

하지만 시간이 지나면서 깨달았다. 컴파일러는 이렇게 말하고 있었다:
- "3번 줄에서 s의 소유권을 s2에게 넘겼잖아"
- "4번 줄에서 s를 쓰려고? 이미 s2 것인데?"
- "이거 C++였으면 런타임에 터졌을 거야"

## Fighting with the Borrow Checker

유명한 "borrow checker와의 싸움" 단계:

### 1단계: 부정 (1주차)
"이건 말도 안 돼! 내 코드는 완벽해!"

### 2단계: 분노 (2주차)
"도대체 왜 안 되는 거야! Clone을 도배하면 되잖아!"

```rust
let data = expensive_computation();
let data2 = data.clone();  // 포기
let data3 = data.clone();  // 이해 못함
let data4 = data.clone();  // 화남
```

### 3단계: 타협 (3주차)
"아... 레퍼런스를 쓰면 되는구나"

```rust
let data = expensive_computation();
process(&data);  // 빌려주기
transform(&data);  // 또 빌려주기
```

### 4단계: 우울 (4주차)
"나는 프로그래밍을 할 줄 모르는 사람이었구나..."

### 5단계: 수용과 깨달음 (5주차)
"잠깐... 이 에러들이 모두 실제 버그였네?"

```rust
fn broken_code() -> &str {
    let s = String::from("hello");
    &s  // 컴파일 에러: s는 함수 끝에서 사라짐
}  // C++에서는 dangling pointer
```

## 실제 경험: 멀티스레드 프로그램

C++에서의 악몽:

```cpp
std::vector<int> data;
std::thread t1([&data] {
    data.push_back(1);  // 💥 data race
});
std::thread t2([&data] {
    data.push_back(2);  // 💥 동시 수정
});
```

Rust에서의 평화:

```rust
let data = Arc::new(Mutex::new(Vec::new()));
let data1 = data.clone();

std::thread::spawn(move || {
    data1.lock().unwrap().push(1);  // 안전!
});

let data2 = data.clone();
std::thread::spawn(move || {
    data2.lock().unwrap().push(2);  // 안전!
});
```

컴파일러가 data race를 원천 차단한다. "Fearless Concurrency"는 마케팅 문구가 아니었다.

## 생산성의 역설

처음 한 달: "Rust는 너무 어려워. 생산성이 떨어져."
6개월 후: "디버깅 시간이 90% 줄었네?"
1년 후: "처음부터 제대로 작동하는 코드를 짜고 있어!"

### 숨겨진 비용들

C++에서의 "빠른" 개발:
- 코딩: 2시간
- 디버깅: 8시간
- 메모리 누수 찾기: 4시간
- 프로덕션 버그 수정: 20시간
- **총합: 34시간**

Rust에서의 "느린" 개발:
- 코딩과 컴파일러와 싸움: 6시간
- 디버깅: 1시간
- 메모리 누수: 없음 (불가능)
- 프로덕션 버그: 거의 없음
- **총합: 7시간**

## 개인적 깨달음

2020년, 나는 10만 줄짜리 C++ 프로젝트를 Rust로 다시 썼다.

결과:
- 코드 라인 수: 10만 → 6만 (40% 감소)
- 버그 리포트: 월 평균 47개 → 3개
- 성능: 15% 향상 (제로 코스트 추상화)
- 팀 만족도: "다시는 C++로 돌아가고 싶지 않아요"

## 파급 효과

Rust를 배운 후 달라진 것들:

1. **다른 언어를 보는 눈이 바뀌었다**
   - Python: "아, 그래서 GIL이 있구나"
   - Java: "GC가 있어도 메모리 누수가 가능하네"
   - JavaScript: "undefined를 만든 사람을 만나고 싶다"

2. **설계 사고방식이 바뀌었다**
   - "이 데이터를 누가 소유할까?"
   - "동시에 접근하면 어떻게 될까?"
   - "실패할 수 있는 모든 지점은 어디인가?"

3. **코드 리뷰가 달라졌다**
   - 전: "이 로직이 맞나요?"
   - 후: "이 타입 설계가 의도를 표현하나요?"

## 2025년의 Rust

오늘날 Rust는:
- Linux 커널의 공식 언어
- Windows의 핵심 컴포넌트 재작성
- Android 시스템 프로그래밍
- WebAssembly의 1등 시민
- 임베디드 시스템의 새로운 표준

하지만 더 중요한 것은 Rust가 가져온 패러다임 전환이다:
- **컴파일 타임 검증 > 런타임 검사**
- **타입 시스템으로 불변식 표현**
- **소유권으로 자원 관리 자동화**

## 마지막 고백

나는 여전히 가끔 borrow checker와 싸운다. 하지만 이제 안다. 그 싸움은 내가 실수를 하고 있다는 신호다. 컴파일러가 "너의 코드는 논리적으로 모순이야"라고 말하고 있는 것이다.

Rust는 단순한 프로그래밍 언어가 아니다. 더 나은 프로그래머가 되는 방법을 가르쳐주는 스승이다.

---

## Connections
→ [[L1_first_steps]] - 이제 시작해보자
→ [[zettel/001_ownership]] - 소유권의 원자적 이해
→ [[zettel/006_borrow_checker]] - 빌림 검사기의 작동 원리
← [[index]] - 목차로 돌아가기

---

Level: L0 (직접 경험)
Date: 2025-08-15
Tags: #rust #experience #memory-safety #personal-story #paradigm-shift