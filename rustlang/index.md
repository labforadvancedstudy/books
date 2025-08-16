# 2025년 Rust - 시스템 프로그래밍의 미래

## 왜 이 책인가?

2025년, 우리는 시스템 프로그래밍의 전환점에 서 있다. 40년 동안 C와 C++가 지배해온 영역에서, Rust가 새로운 패러다임을 제시하고 있다. 이 책은 단순한 Rust 문법 가이드가 아니다. 메모리 안전성과 성능이라는 두 마리 토끼를 동시에 잡는 것이 어떻게 가능한지, 그 마법의 원리를 파인만처럼 쉽게 풀어낸다.

## 책의 구조

각 레벨은 독립적으로 읽을 수 있지만, 전체가 하나의 거대한 이야기를 이룬다. L0에서 시작해 L9까지 올라가면서, 당신은 Rust가 단순한 프로그래밍 언어가 아니라 사고방식의 혁명임을 깨닫게 될 것이다.

---

## 목차

### [L0: 프로그래밍의 경험](./L0_experience.md)
*왜 Rust인가? 개인적 좌절과 깨달음*
- 세그폴트와의 전쟁
- "아빠, 왜 프로그램이 죽어요?"
- 컴파일러가 친구가 되는 순간
- 첫 번째 "fighting with the borrow checker"

### [L1: Rust 첫 걸음](./L1_first_steps.md)
*Hello World부터 시작하는 여정*
- cargo new의 마법
- println!이 매크로인 이유
- 변수와 가변성: let과 let mut의 철학
- 타입 추론의 아름다움

### [L2: 소유권의 마법](./L2_ownership.md)
*메모리 안전성의 비밀*
- 도서관 책 대출 시스템으로 이해하는 소유권
- move, copy, clone의 삼각관계
- 스택과 힙: 메모리의 두 얼굴
- Drop의 우아한 춤

### [L3: 타입 시스템의 힘](./L3_type_system.md)
*컴파일 타임에 버그 잡기*
- Option과 Result: null의 죽음
- 패턴 매칭: if-else의 진화
- 트레이트: 덕 타이핑의 정적 버전
- 제네릭: 하나의 코드, 모든 타입

### [L4: 동시성과 병렬성](./L4_concurrency.md)
*Fearless Concurrency의 진실*
- Send와 Sync: 스레드 안전성의 수호자
- Arc<Mutex<T>>: 공유 상태의 우아한 해법
- async/await: 콜백 지옥에서의 해방
- 채널과 메시지 패싱

### [L5: 실전 프로젝트 구축](./L5_real_projects.md)
*이론에서 실무로*
- CLI 도구 만들기
- 웹 서버 구축하기
- 시스템 프로그래밍 실습
- 에러 처리의 예술

### [L6: 생태계와 도구들](./L6_ecosystem.md)
*Rust 커뮤니티의 보물들*
- Cargo: 역사상 최고의 패키지 매니저
- rustfmt, clippy: 코드 품질의 자동화
- 크레이트 생태계 탐험
- 테스팅과 벤치마킹

### [L7: 고급 패턴과 최적화](./L7_advanced_patterns.md)
*전문가의 비밀*
- Zero-cost abstractions의 실체
- unsafe: 금지된 마법
- 매크로: 코드를 생성하는 코드
- const generics와 GAT

### [L8: 프로덕션 배포](./L8_production.md)
*실전에서 살아남기*
- 성능 프로파일링
- 메모리 최적화
- 크로스 컴파일
- CI/CD 파이프라인

### [L9: Rust의 철학과 미래](./L9_philosophy.md)
*언어를 넘어선 사상*
- 소유권이 가르쳐준 것들
- 시스템 프로그래밍의 민주화
- WebAssembly와 미래
- Rust가 바꾸는 세계

---

## 연결된 개념들

### Zettel 네트워크
이 책은 131개의 원자적 Rust 개념들([[zettel/000_index]])과 유기적으로 연결되어 있다. 각 장에서 관련 zettel을 참조하여 더 깊은 이해를 얻을 수 있다.

### 다른 HA 시리즈와의 연결
- [[../HA_programming_language]]: 프로그래밍 언어의 본질
- [[../HA_software_engineering]]: 소프트웨어 공학 원리
- [[../HA_computer]]: 컴퓨터 아키텍처 이해

---

## 이 책을 읽는 방법

1. **초보자**: L0부터 순서대로 읽어가며 Rust의 세계에 입문
2. **경험자**: 관심 있는 레벨로 바로 점프, zettel 참조로 깊이 더하기
3. **전문가**: L7-L9의 철학적 관점과 미래 전망에 집중

## 코드 예제

모든 예제 코드는 실제로 컴파일되고 실행된다. 각 장의 코드는 다음 저장소에서 확인할 수 있다:
- 각 레벨별 프로젝트 구조
- 단계별 빌드 과정
- 실제 에러 메시지와 해결법

---

*"The most important property of a program is whether it accomplishes the intention of its user."* - C.A.R. Hoare

하지만 Rust는 한 걸음 더 나아간다: "그 의도를 안전하게, 효율적으로, 그리고 우아하게 달성할 수 있는가?"

---

Level: L0-L9 (전체 스펙트럼)
Date: 2025-08-15
Tags: #rust #systems-programming #memory-safety #concurrency #2025