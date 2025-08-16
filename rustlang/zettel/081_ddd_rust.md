# Domain Driven Design in Rust

## 핵심 개념
Domain Driven Design을 Rust의 타입 시스템으로 표현하는 패턴

## 값 객체 (Value Objects)
```rust
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
pub struct EmailAddress(String);

impl EmailAddress {
    pub fn new(email: &str) -> Result<Self, ValidationError> {
        // 이메일 검증 로직
        if email.contains('@') {
            Ok(Self(email.to_string()))
        } else {
            Err(ValidationError::InvalidEmail)
        }
    }
}
```

## 엔티티 (Entities)
```rust
pub struct User {
    id: UserId,  // 식별자
    email: EmailAddress,  // 값 객체
    name: UserName,
    created_at: DateTime<Utc>,
}

impl User {
    pub fn change_email(&mut self, email: EmailAddress) {
        self.email = email;
        // 도메인 이벤트 발생
    }
}
```

## 애그리거트 (Aggregates)
```rust
pub struct Order {
    id: OrderId,
    items: Vec<OrderItem>,
    total: Money,
}

impl Order {
    // 애그리거트 루트를 통한 접근
    pub fn add_item(&mut self, item: OrderItem) -> Result<(), DomainError> {
        // 비즈니스 규칙 검증
        if self.items.len() >= 100 {
            return Err(DomainError::TooManyItems);
        }
        self.items.push(item);
        self.recalculate_total();
        Ok(())
    }
    
    fn recalculate_total(&mut self) {
        // 내부 일관성 유지
    }
}
```

## 도메인 서비스
```rust
pub struct PricingService;

impl PricingService {
    pub fn calculate_discount(
        order: &Order,
        customer: &Customer,
    ) -> Money {
        // 여러 애그리거트에 걸친 비즈니스 로직
    }
}
```

## Rust 특성 활용
- **newtype pattern**: 원시 타입 래핑으로 타입 안전성
- **Result 타입**: 도메인 검증 실패 명시적 처리
- **소유권**: 애그리거트 경계 강제
- **트레이트**: 도메인 행동 추상화

## 장점
- 컴파일 타임 도메인 규칙 검증
- 잘못된 상태 표현 불가능
- 명시적 에러 처리

## 관련 개념
- [[082_hexagonal_architecture]] - 포트와 어댑터
- [[083_repository_pattern]] - 영속성 추상화
- [[084_cqrs_rust]] - 명령과 조회 분리