# Level 8: 수학의 수학

*수학이 처음부터 범주로 만들어져 있었다는 것을 발견하는 곳*

---

## 패턴의 패턴

이제 박사후연구원인 아마라가 불안한 무언가를 알아차린다. 수학의 모든 영역이 갖고 있다:
- 대상 (군, 공간, 환...)
- 대상 사이의 사상 (준동형사상, 연속함수...)
- 사상을 합성하는 방법
- 항등사상

같은 패턴, 어디에나. 이 패턴 자체의 수학이 있어야 한다.

범주론의 등장.

## 범주: 궁극의 추상화

범주는 다음으로 구성된다:
- 대상 (그것들이 무엇인지 묻지 마라)
- 대상 사이의 사상 (화살표)
- 사상의 합성
- 각 대상에 대한 항등사상

그게 전부다. 내부 구조는 없다. 대상은 그저... 사상이 가는 사이의 것들.

예:
- **Set**: 대상은 집합, 사상은 함수
- **Grp**: 대상은 군, 사상은 준동형사상
- **Top**: 대상은 공간, 사상은 연속사상
- **Cat**: 대상은 범주, 사상은 함자

잠깐, 뭐라고?

## 함자: 범주 사이의 사상

함자 F: C → D는 매핑한다:
- C의 대상을 D의 대상으로
- C의 사상을 D의 사상으로
- 합성과 항등원을 보존한다

예: 기본군 π₁: **Top** → **Grp**
- 공간을 군으로 보낸다
- 연속사상을 준동형사상으로 보낸다
- 위상수학이 대수가 된다!

함자는 범주론의 준동형사상이다. 범주들이 범주를 이룬다. 뱀이 자기 꼬리를 먹는다.

## 자연변환

때때로 두 함자가 "기본적으로 같다." 자연변환이 이것을 정밀하게 만든다.

주어진 F, G: C → D에 대해, 자연변환 α: F ⟹ G는 제공한다:
- 각 대상 X에 대해, 사상 αₓ: F(X) → G(X)
- 이것들은 C의 모든 사상과 가환한다

행렬식은 자연스럽다. 이중 쌍대는 원래 것과 자연동형이다. "자연스러운"이 마침내 정의를 갖는다!

## 요네다 보조정리

당신이 들어본 적 없는 가장 중요한 정리:

"대상은 다른 모든 대상과의 관계로 완전히 결정된다."

형식적으로: 함자 h^A(−) = Hom(−, A)는 A를 동형을 제외하고 결정한다.

당신은 당신의 관계들이다. 대상은 내부 구조가 없다 - 오직 사상의 패턴만이 중요하다.

이것은 심오하거나 미친 짓이다.

## 극한과 여극한

오래된 것들로부터 새로운 대상을 어떻게 만드는가? 극한과 여극한:
- 곱: A와 B 둘 다로 가는 "최선의" 대상
- 쌍대곱: A와 B 둘 다 가는 "최선의" 대상
- 등화자: 두 사상이 일치하는 곳
- 당김: 일반화된 교집합

"최선"은 보편성질을 의미한다 - 모든 것을 가환하게 만드는 유일한 사상이 있다.

범주론은 구성을 명세로 대체한다.

## 수반함자

어떤 함자는 쌍으로 온다 F ⊣ G 여기서:
Hom(F(X), Y) ≅ Hom(X, G(Y))

어디에나 있는 예:
- 자유 ⊣ 망각 (구조 추가 대 잊기)
- 곱 ⊣ 지수 (커링)
- Δ ⊣ 여극한 (상수 도표 대 접착)

수반은 어디에나 있다. 그들은 수학의 용수철과 기어다.

## 모나드: 합성 패턴

모나드는 함자 T: C → C로 다음을 갖는다:
- 자연변환 η: Id ⟹ T (단위원)
- 자연변환 μ: T² ⟹ T (곱셈)
- 이것들은 결합법칙과 단위원 법칙을 만족한다

모나드는 "대수적 이론"을 포착한다:
- 리스트 모나드: 자유 모노이드
- Maybe 모나드: 부분함수
- 연속 모나드: 제어 흐름

모든 수반이 모나드를 준다. 모나드는 어디에나 있다.

## 토포스: 집합처럼 행동하는 범주

토포스는 다음을 가진 범주다:
- 모든 유한 극한
- 지수 (함수 대상)
- 부대상 분류자 (진리값)

예:
- **Set**은 토포스다
- 공간 위의 층들이 토포스를 이룬다
- 함자 G → **Set**이 토포스를 이룬다 (G-집합)

토포스에서, 논리를 할 수 있다! 참/거짓이 사상이 된다. 그리고 다른 토포스는 다른 논리를 갖는다...

## 범주에서의 논리

고전 논리가 유일한 논리가 아니다:
- **Set**에서: 배중률이 성립한다
- 층에서: 배중률이 실패할 수 있다
- 구성적 토포스에서: 선택공리가 없다

각 토포스는 자신만의 내부 논리를 갖는다. 수학은 논리 위에 지어진 것이 아니다 - 논리가 수학적 구조로부터 나타난다!

## 집합론: 그저 또 다른 범주?

집합론이 한 세기 동안 기초를 지배했다. 하지만:
- 집합은 **Set**의 대상이다
- 함수는 사상이다
- 집합론적 구성은 범주적 극한이다

집합론이 근본적인가 아니면 많은 범주 중 하나일 뿐인가?

범주론이 제안한다: 지하실은 없다. 끝까지 범주다.

## 기초의 위기

범주는 무엇에 기초하는가?
- 집합? 하지만 **Set**은 그저 하나의 범주다
- 클래스? 그건 그저 더 큰 집합이다
- 아무것도 아닌? 순수 구조?

우리는 너무 추상화해서 땅을 잃었다. 수학은 자유롭게 떠다닌다, 기초 없이.

## 호모토피 유형론

최신 시도: 유형은 공간이고, 증명은 경로다:
- 유형(집합이 아닌)이 기본이다
- 동등성은 공간에서의 경로다
- 고차 동등성은 고차 경로다

수학, 논리, 그리고 계산이 통합된다. 증명하기가 경로 찾기가 된다. 공간이 논리다.

## 2-범주와 그 이상

범주는 대상 사이에 사상을 갖는다. 왜 사상 사이의 사상은 없는가?

2-범주가 추가한다:
- 0-세포 (대상)
- 1-세포 (사상)
- 2-세포 (사상 사이의 사상)

이것은 계속된다:
- 3-범주, 4-범주...
- ω-범주 (모든 유한 차원)
- (∞,1)-범주 (∞-차원, 하지만 고차 사상은 가역)

토끼굴은 얼마나 깊은가?

## 범주론 이야기

미적분을 범주적으로 유도하는 것을 생각해보라:
- 매끄러운 다양체가 범주를 이룬다
- 실수선 ℝ은 대상이다
- 매끄러운 함수 M → ℝ은 "관측가능량"이다
- 접벡터는 관측가능량의 미분이다
- 접다발이 자연스럽게 나타난다

우리는 순수한 관계로부터 미분기하를 재건했다. 좌표도 없고, 공식도 없다 - 그저 화살표뿐.

## L8이 할 수 있는 것

범주로, 우리는 할 수 있다:
- 수학 영역 사이에 정리 운반하기
- 보편성질 정의하기
- "자연스러운" 구성 이해하기
- 집합론 없이 수학하기
- 논리, 계산, 그리고 기하 연결하기

## 수학의 통일성

범주론이 드러낸다:
- 대수와 위상은 동등하다 (함자를 통해)
- 논리와 기하는 동등하다 (토포스를 통해)
- 계산과 증명은 동등하다 (유형론을 통해)

모든 수학은 하나의 수학이다, 다른 각도에서 본.

## 우리가 있는 곳

아마라는 이제 본다:
- 대상은 중요하지 않다, 오직 관계만
- 수학은 패턴의 패턴의 패턴을 연구한다
- 기초는 선택사항이다
- 논리는 구조로부터 나타난다
- 모든 것이 모든 것과 연결되어 있다

하지만 무한은 여전히 잠복해 있다. 고차 범주가 위로 나선을 그린다. 랭글랜즈 프로그램이 불가능한 연결을 약속한다...

---

*다음: L9 - 가장자리*

*수학이 더 이상 자신이 무엇을 하고 있는지 모른다고 인정하는 곳*

---
*[영어 원문 보기](../../../2_physical_emergence/HA_math/L8_The_Mathematics_of_Mathematics.md)*