# L4: 동시성과 병렬성 - Fearless Concurrency의 진실

## Core Insight
"Fearless Concurrency"는 마케팅 슬로건이 아니다. Rust는 소유권 시스템과 타입 시스템을 통해 데이터 레이스를 컴파일 타임에 방지한다. 멀티스레드 프로그래밍의 공포를 제거하고, 동시성을 일반 프로그래밍처럼 만든다.

---

## Send와 Sync - 스레드 안전성의 수호자

### 보이지 않는 마커 트레이트

```rust
// 자동으로 구현되는 마커 트레이트
pub unsafe auto trait Send {}  // 다른 스레드로 전송 가능
pub unsafe auto trait Sync {}  // 여러 스레드에서 참조 가능
```

무슨 뜻인가?
- **Send**: 소유권을 다른 스레드로 이동할 수 있다
- **Sync**: 여러 스레드가 동시에 &T로 접근할 수 있다

### 자동 추론의 마법

```rust
use std::thread;

// i32는 Send + Sync
fn share_number() {
    let number = 42;
    thread::spawn(move || {
        println!("다른 스레드: {}", number);  // OK!
    });
}

// Rc는 !Send (참조 카운팅이 스레드 안전하지 않음)
use std::rc::Rc;
fn cant_share_rc() {
    let rc = Rc::new(5);
    // thread::spawn(move || {
    //     println!("{}", rc);  // 컴파일 에러!
    // });
}
```

컴파일 에러 메시지:
```
error[E0277]: `Rc<i32>` cannot be sent between threads safely
   = help: the trait `Send` is not implemented for `Rc<i32>`
```

## Arc와 Mutex - 공유 상태의 우아한 해법

### 문제: 여러 스레드가 같은 데이터를 수정하려면?

C++의 악몽:
```cpp
int counter = 0;
// 스레드 1
counter++;  // 읽기 → 증가 → 쓰기
// 스레드 2
counter++;  // 동시에! 데이터 레이스!
```

### Arc - Atomic Reference Counting

```rust
use std::sync::Arc;
use std::thread;

fn share_immutable() {
    let data = Arc::new(vec![1, 2, 3]);
    
    let handles: Vec<_> = (0..3).map(|i| {
        let data = Arc::clone(&data);  // 참조 카운트 증가
        thread::spawn(move || {
            println!("스레드 {}: {:?}", i, data);
        })
    }).collect();
    
    for handle in handles {
        handle.join().unwrap();
    }
}  // 모든 Arc가 drop되면 데이터 해제
```

### Mutex - Mutual Exclusion

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn shared_counter() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();  // 잠금 획득
            *num += 1;
        });  // 여기서 자동으로 잠금 해제 (RAII)
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("결과: {}", *counter.lock().unwrap());  // 10
}
```

### RwLock - 읽기는 많이, 쓰기는 가끔

```rust
use std::sync::{Arc, RwLock};
use std::thread;

struct Cache {
    data: Arc<RwLock<HashMap<String, String>>>,
}

impl Cache {
    fn read(&self, key: &str) -> Option<String> {
        let data = self.data.read().unwrap();  // 여러 읽기 동시 가능
        data.get(key).cloned()
    }
    
    fn write(&self, key: String, value: String) {
        let mut data = self.data.write().unwrap();  // 배타적 쓰기
        data.insert(key, value);
    }
}
```

## 채널과 메시지 패싱

### "Do not communicate by sharing memory; share memory by communicating"

```rust
use std::sync::mpsc;  // Multiple Producer, Single Consumer
use std::thread;

fn channel_example() {
    let (tx, rx) = mpsc::channel();
    
    thread::spawn(move || {
        let messages = vec![
            String::from("안녕"),
            String::from("from"),
            String::from("스레드"),
        ];
        
        for msg in messages {
            tx.send(msg).unwrap();
            thread::sleep(Duration::from_millis(500));
        }
    });
    
    // 메인 스레드에서 수신
    for received in rx {
        println!("받음: {}", received);
    }
}
```

### Multiple Producers

```rust
fn multiple_producers() {
    let (tx, rx) = mpsc::channel();
    
    for i in 0..3 {
        let tx = tx.clone();  // sender 복제
        thread::spawn(move || {
            tx.send(format!("스레드 {}", i)).unwrap();
        });
    }
    
    drop(tx);  // 원본 sender drop (중요!)
    
    for received in rx {
        println!("받음: {}", received);
    }
}
```

## async/await - 콜백 지옥에서의 해방

### 동기 vs 비동기

```rust
// 동기: 블로킹
fn fetch_sync(url: &str) -> String {
    // 네트워크 요청... 기다림...
    "response".to_string()
}

// 비동기: 논블로킹
async fn fetch_async(url: &str) -> String {
    // 네트워크 요청 시작
    // 다른 일 할 수 있음
    // 완료되면 재개
    "response".to_string()
}
```

### Future와 async/await

```rust
use tokio;  // 비동기 런타임

#[tokio::main]
async fn main() {
    let future1 = fetch_data(1);
    let future2 = fetch_data(2);
    
    // 동시 실행!
    let (result1, result2) = tokio::join!(future1, future2);
    
    println!("결과: {} {}", result1, result2);
}

async fn fetch_data(id: u32) -> String {
    tokio::time::sleep(Duration::from_secs(1)).await;
    format!("데이터 {}", id)
}
```

### 비동기 스트림

```rust
use tokio::stream::StreamExt;

async fn process_stream() {
    let mut stream = tokio::stream::iter(vec![1, 2, 3]);
    
    while let Some(value) = stream.next().await {
        println!("처리: {}", value);
    }
}
```

## 실전: 웹 크롤러

```rust
use std::sync::{Arc, Mutex};
use std::collections::{HashSet, VecDeque};
use std::thread;

struct Crawler {
    to_visit: Arc<Mutex<VecDeque<String>>>,
    visited: Arc<Mutex<HashSet<String>>>,
}

impl Crawler {
    fn new(start_url: String) -> Self {
        let mut to_visit = VecDeque::new();
        to_visit.push_back(start_url);
        
        Crawler {
            to_visit: Arc::new(Mutex::new(to_visit)),
            visited: Arc::new(Mutex::new(HashSet::new())),
        }
    }
    
    fn crawl(&self) {
        let mut handles = vec![];
        
        for _ in 0..4 {  // 4개 워커 스레드
            let to_visit = Arc::clone(&self.to_visit);
            let visited = Arc::clone(&self.visited);
            
            let handle = thread::spawn(move || {
                loop {
                    let url = {
                        let mut queue = to_visit.lock().unwrap();
                        queue.pop_front()
                    };
                    
                    let url = match url {
                        Some(url) => url,
                        None => {
                            thread::sleep(Duration::from_millis(100));
                            continue;
                        }
                    };
                    
                    {
                        let mut visited_set = visited.lock().unwrap();
                        if visited_set.contains(&url) {
                            continue;
                        }
                        visited_set.insert(url.clone());
                    }
                    
                    // 실제 크롤링
                    println!("크롤링: {}", url);
                    let new_urls = fetch_and_parse(&url);
                    
                    {
                        let mut queue = to_visit.lock().unwrap();
                        for new_url in new_urls {
                            queue.push_back(new_url);
                        }
                    }
                }
            });
            
            handles.push(handle);
        }
        
        // 워커들 기다리기
        for handle in handles {
            handle.join().unwrap();
        }
    }
}
```

## Rayon - 데이터 병렬 처리

```rust
use rayon::prelude::*;

fn parallel_processing() {
    let numbers: Vec<i32> = (1..1000000).collect();
    
    // 순차 처리
    let sum: i32 = numbers.iter().sum();
    
    // 병렬 처리 (한 줄 추가!)
    let parallel_sum: i32 = numbers.par_iter().sum();
    
    // 병렬 맵
    let squares: Vec<_> = numbers
        .par_iter()
        .map(|&x| x * x)
        .collect();
    
    // 병렬 필터와 리듀스
    let result = numbers
        .par_iter()
        .filter(|&&x| x % 2 == 0)
        .map(|&x| x * x)
        .sum::<i32>();
}
```

## Atomic 타입들

```rust
use std::sync::atomic::{AtomicUsize, Ordering};
use std::sync::Arc;
use std::thread;

fn atomic_counter() {
    let counter = Arc::new(AtomicUsize::new(0));
    let mut handles = vec![];
    
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            for _ in 0..1000 {
                // 락 없이 원자적 증가!
                counter.fetch_add(1, Ordering::SeqCst);
            }
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("최종: {}", counter.load(Ordering::SeqCst));  // 10000
}
```

## 데드락 방지

```rust
// 데드락 가능한 코드
fn potential_deadlock() {
    let lock1 = Arc::new(Mutex::new(1));
    let lock2 = Arc::new(Mutex::new(2));
    
    let l1 = Arc::clone(&lock1);
    let l2 = Arc::clone(&lock2);
    thread::spawn(move || {
        let _a = l1.lock().unwrap();
        thread::sleep(Duration::from_millis(50));
        let _b = l2.lock().unwrap();  // lock1 → lock2 순서
    });
    
    let _b = lock2.lock().unwrap();
    thread::sleep(Duration::from_millis(50));
    let _a = lock1.lock().unwrap();  // lock2 → lock1 순서 (데드락!)
}

// 해결: 항상 같은 순서로 락 획득
fn avoid_deadlock() {
    // 또는 parking_lot의 deadlock detection 사용
    // 또는 lock-free 자료구조 사용
}
```

## Actor 모델

```rust
use tokio::sync::mpsc;

struct Actor {
    receiver: mpsc::Receiver<Message>,
    state: State,
}

enum Message {
    Increment,
    GetCount(mpsc::Sender<u32>),
}

impl Actor {
    async fn run(mut self) {
        while let Some(msg) = self.receiver.recv().await {
            match msg {
                Message::Increment => {
                    self.state.count += 1;
                }
                Message::GetCount(sender) => {
                    sender.send(self.state.count).await.unwrap();
                }
            }
        }
    }
}
```

## 성능 측정

```rust
use std::time::Instant;

fn benchmark_parallel() {
    let data: Vec<i32> = (1..10_000_000).collect();
    
    // 순차 처리
    let start = Instant::now();
    let _sum: i32 = data.iter().map(|&x| x * 2).sum();
    println!("순차: {:?}", start.elapsed());
    
    // 병렬 처리
    let start = Instant::now();
    let _sum: i32 = data.par_iter().map(|&x| x * 2).sum();
    println!("병렬: {:?}", start.elapsed());
}
```

## 마무리: Fearless의 의미

Fearless Concurrency가 주는 것:

1. **컴파일 타임 보장**: 데이터 레이스 불가능
2. **명확한 소유권**: 누가 데이터를 소유하는지 명확
3. **자동 정리**: 락이 자동으로 해제됨
4. **성능**: 불필요한 동기화 없음

두려움 없이 동시성 코드를 작성할 수 있다. 컴파일되면, 안전하다.

---

## Connections
→ [[L5_real_projects]] - 실전 프로젝트에서 동시성 활용
→ [[zettel/078_send_sync]] - Send와 Sync 심화
→ [[zettel/019_async_await]] - async 프로그래밍 상세
→ [[zettel/067_mutex]] - Mutex 내부 구현
← [[L3_type_system]] - 타입으로 보장하는 안전성

---

Level: L4 (시스템 설계)
Date: 2025-08-15
Tags: #rust #concurrency #parallelism #async #fearless