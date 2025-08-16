# Polonius Borrow Checker

## 핵심 개념
Datalog 기반의 차세대 빌림 검사기로 더 정확한 생명주기 분석

## 현재 빌림 검사기의 한계
```rust
// 현재 빌림 검사기는 거부하지만 실제로는 안전한 코드
fn get_or_insert<'a>(
    map: &'a mut HashMap<u32, String>,
    key: u32,
) -> &'a String {
    match map.get(&key) {
        Some(value) => value,  // 에러: map이 불변으로 빌려짐
        None => {
            map.insert(key, String::new());
            map.get(&key).unwrap()
        }
    }
}
```

## Polonius가 해결하는 문제
```rust
// Polonius에서는 컴파일되는 코드
fn polonius_example<'a>(x: &'a mut Vec<i32>) -> &'a i32 {
    let r = &x[0];
    
    // 현재: r이 살아있어서 x 사용 불가
    // Polonius: r이 실제로 사용되지 않으면 x 재사용 가능
    if false {
        x.push(1);  // Polonius에서는 OK
    }
    
    r
}
```

## 위치 기반 생명주기
```rust
// Polonius는 각 프로그램 포인트에서 생명주기 추적
fn location_sensitive<'a>(cond: bool, x: &'a mut i32) -> &'a i32 {
    let y = &*x;  // 'y 생명주기 시작
    
    if cond {
        return y;  // 'y가 여기서 필요
    }
    
    // 'y가 더 이상 필요없음을 인식
    *x += 1;  // Polonius에서는 OK
    x
}
```

## Subset 관계 정밀화
```rust
// 더 정확한 subset 관계 추론
fn precise_subsets<'a, 'b>(
    x: &'a mut Vec<&'b str>,
    y: &'b str,
) {
    // Polonius는 'b ⊆ 'a를 더 정확히 추론
    x.push(y);
    
    // 불필요한 생명주기 제약 제거
    let z: &'a str = y;  // 더 유연한 변환
}
```

## Two-phase 빌림 개선
```rust
// Two-phase 빌림의 더 정확한 분석
fn two_phase_borrowing() {
    let mut v = vec![1, 2, 3];
    
    // 현재: 복잡한 two-phase 빌림 규칙
    // Polonius: 더 직관적인 분석
    v.push(v.len());  // 읽기와 쓰기가 겹치지만 OK
}
```

## Loan 추적
```rust
// Polonius의 loan 기반 분석
fn loan_tracking<'a>(x: &'a mut i32) -> &'a i32 {
    let loan1 = &*x;  // Loan L1 생성
    
    // L1이 활성화되지 않은 경로
    if false {
        *x = 5;  // Polonius: L1 비활성, OK
    }
    
    loan1  // L1 반환
}
```

## 에러 메시지 개선
```rust
// Polonius의 향상된 에러 메시지
fn better_errors() {
    let mut x = 5;
    let r1 = &x;
    let r2 = &mut x;  // 에러
    
    // Polonius 에러:
    // "r1이 라인 X에서 생성되어 라인 Y에서 사용되므로
    //  r2를 생성할 수 없습니다"
    
    println!("{}", r1);  // r1 사용 위치 명시
}
```

## Datalog 규칙
```prolog
// Polonius 내부 Datalog 규칙 예시
loan_live_at(L, P) :-
    loan_issued_at(L, P).

loan_live_at(L, P) :-
    loan_live_at(L, P_prev),
    cfg_edge(P_prev, P),
    !loan_killed_at(L, P).

error(P) :-
    loan_live_at(L1, P),
    loan_live_at(L2, P),
    loans_conflict(L1, L2).
```

## 실험적 기능 활성화
```rust
// rustc 플래그로 Polonius 활성화
// RUSTFLAGS="-Z polonius" cargo build

#![feature(polonius)]  // nightly 필요

fn experimental_polonius() {
    // Polonius 전용 패턴 사용
}
```

## 성능 영향
```rust
// Polonius의 컴파일 시간 고려사항
mod benchmarks {
    // 현재 빌림 검사기: O(n²) 최악의 경우
    // Polonius: O(n³) 이론적 복잡도
    // 실제: 대부분 코드에서 비슷하거나 빠름
    
    fn complex_lifetimes() {
        // 많은 생명주기 제약이 있는 코드
        // Polonius가 더 정확하지만 느릴 수 있음
    }
}
```

## 마이그레이션 전략
```rust
// 점진적 마이그레이션
#[cfg(polonius)]
fn new_pattern() {
    // Polonius에서만 작동하는 패턴
}

#[cfg(not(polonius))]
fn old_pattern() {
    // 현재 빌림 검사기용 우회 패턴
}
```

## Rust 특성 활용
- **논리 프로그래밍**: Datalog 기반 분석
- **정확성**: 더 많은 안전한 코드 허용
- **점진적 채택**: 기존 코드 호환
- **에러 개선**: 명확한 진단

## 영향받는 패턴
- Self-referential 구조체
- 복잡한 match 표현식
- Conditional 반환
- Loop-carried 의존성

## 관련 개념
- [[028_nll]] - Non-Lexical Lifetimes
- [[006_borrow_checker]] - 현재 빌림 검사기
- [[027_higher_ranked_lifetimes]] - 고차 생명주기