# Pattern Types

## 핵심 개념
패턴을 타입 시스템에 통합하여 더 정밀한 타입 표현

## 기본 패턴 타입
```rust
#![feature(pattern_types)]

// 패턴을 타입으로 사용
type NonZeroEven = i32 is (x if x != 0 && x % 2 == 0);

fn process_even(n: NonZeroEven) -> i32 {
    // n은 반드시 0이 아닌 짝수
    n / 2  // 안전하게 나누기
}
```

## 열거형 패턴 타입
```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}

// Ok 변형만 허용하는 타입
type OnlyOk<T> = Result<T, !> is Ok(_);

fn unwrap_safe(result: OnlyOk<String>) -> String {
    match result {
        Ok(val) => val,
        // Err 케이스 불필요 - 컴파일러가 알고 있음
    }
}
```

## 범위 패턴 타입
```rust
// 특정 범위의 값만 허용
type Percentage = u8 is 0..=100;
type ValidPort = u16 is 1..=65535;

fn set_opacity(opacity: Percentage) {
    // opacity는 0-100 사이 보장
    apply_opacity(opacity as f32 / 100.0);
}
```

## 구조체 패턴 타입
```rust
struct User {
    id: u64,
    name: String,
    age: u8,
}

// 성인 사용자만 나타내는 타입
type AdultUser = User is User { age: 18.., .. };

fn serve_alcohol(user: AdultUser) {
    // user.age >= 18 보장됨
    println!("Serving to {}", user.name);
}
```

## 튜플 패턴 타입
```rust
// 정규화된 벡터 (길이가 1인 3D 벡터)
type UnitVector = (f32, f32, f32) is (x, y, z) 
    if (x*x + y*y + z*z - 1.0).abs() < 0.001;

fn reflect(vector: UnitVector, normal: UnitVector) -> UnitVector {
    // 두 벡터 모두 단위 벡터임이 보장됨
    calculate_reflection(vector, normal)
}
```

## 가드와 패턴 타입
```rust
// 복잡한 가드 조건
type ValidEmail = String is s if s.contains('@') && s.len() < 255;

fn send_email(to: ValidEmail, subject: String) {
    // 이메일 유효성이 타입으로 보장됨
}

// 패턴 타입 변환
fn parse_email(s: String) -> Option<ValidEmail> {
    if s.contains('@') && s.len() < 255 {
        Some(s as ValidEmail)  // 안전한 캐스팅
    } else {
        None
    }
}
```

## 제네릭 패턴 타입
```rust
// 비어있지 않은 벡터
type NonEmptyVec<T> = Vec<T> is v if !v.is_empty();

fn get_first<T>(vec: NonEmptyVec<T>) -> &T {
    &vec[0]  // 패닉 없이 안전
}

fn get_last<T>(vec: NonEmptyVec<T>) -> &T {
    vec.last().unwrap()  // 안전한 unwrap
}
```

## 패턴 타입 세분화
```rust
type Integer = i32;
type Positive = Integer is x if x > 0;
type Even = Integer is x if x % 2 == 0;
type PositiveEven = Integer is x if x > 0 && x % 2 == 0;

// 타입 관계: PositiveEven ⊆ Positive ∩ Even

fn refine(n: Positive) -> Option<PositiveEven> {
    if n % 2 == 0 {
        Some(n as PositiveEven)
    } else {
        None
    }
}
```

## 함수 시그니처의 패턴 타입
```rust
// 반환 타입에 패턴 타입 사용
fn create_valid_port(n: u16) -> Result<ValidPort, String> {
    if n >= 1 && n <= 65535 {
        Ok(n as ValidPort)
    } else {
        Err("Invalid port number".to_string())
    }
}

// 여러 패턴 타입 조합
fn safe_divide(
    numerator: i32,
    denominator: i32 is n if n != 0
) -> i32 {
    numerator / denominator  // 0으로 나누기 불가능
}
```

## 패턴 타입과 트레이트
```rust
trait Validate {
    type Valid: Pattern;
    
    fn validate(self) -> Option<Self::Valid>;
}

impl Validate for String {
    type Valid = String is s if s.len() > 0 && s.len() < 100;
    
    fn validate(self) -> Option<Self::Valid> {
        if self.len() > 0 && self.len() < 100 {
            Some(self as Self::Valid)
        } else {
            None
        }
    }
}
```

## 런타임 검증 제거
```rust
// 컴파일 타임에 검증 완료
fn optimized_function(n: NonZeroEven) -> i32 {
    // 런타임 체크 불필요
    let half = n / 2;  // 0 체크 불필요
    let squared = half * half;
    squared
}

// 기존 방식 (런타임 체크 필요)
fn old_function(n: i32) -> Option<i32> {
    if n != 0 && n % 2 == 0 {
        Some((n / 2) * (n / 2))
    } else {
        None
    }
}
```

## Rust 특성 활용
- **타입 안전**: 불변식을 타입으로 표현
- **제로 비용**: 런타임 검증 제거
- **표현력**: 더 정밀한 타입
- **최적화**: 컴파일러 최적화 향상

## 사용 사례
- 도메인 모델링
- API 계약 강제
- 상태 머신
- 검증된 데이터 타입

## 관련 개념
- [[016_pattern_matching]] - 패턴 매칭
- [[126_refinement_types]] - 세분화 타입
- [[127_dependent_types]] - 의존 타입