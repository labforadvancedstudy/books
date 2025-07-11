# Level 4: 순간의 속도 - 변화를 잡다
*17살, 물리 시간*

> "미분은 순간을, 적분은 전체를 본다." - 수현의 깨달음

## 제논의 역설

고3 물리 시간. 선생님이 묻는다.

"화살이 날아간다. 각 순간 화살은 정지해 있다. 그런데 어떻게 움직이지?"

수현이는 당황한다. 맞다. 사진을 찍으면 화살은 멈춰 있다. 순간순간은 정지인데 어떻게 전체는 운동일까?

"이게 제논의 역설이에요. 2500년 된 문제죠."

"그럼 답은요?"

"미적분학이 답입니다."

## 평균속도에서 순간속도로

자유낙하 실험. 공을 떨어뜨린다.
- 1초 후: 5m
- 2초 후: 20m  
- 3초 후: 45m

"1초에서 2초 사이 평균속도는?"
수현이가 계산한다. (20-5)/(2-1) = 15 m/s

"1.5초에서 2초 사이는?"
(20-11.25)/(2-1.5) = 17.5 m/s

"1.9초에서 2초 사이는?"
(20-18.05)/(2-1.9) = 19.5 m/s

"1.99초에서 2초 사이는?"
19.95 m/s

"정확히 2초일 때 순간속도는?"

수현이는 깨닫는다. 구간을 줄일수록 20 m/s에 가까워진다!

## 극한의 발견

칠판에 식이 적힌다:

v = lim(h→0) [f(t+h) - f(t)]/h

"이게 미분이에요. h가 0으로 갈 때의 극한값."

"h가 0이면 0/0 아니에요?"

"0이 되는 게 아니라 0으로 '다가가는' 거예요."

위치 함수: s(t) = 5t²
속도 함수: v(t) = 10t

2초일 때 순간속도 = 20 m/s. 정확히 맞다!

## 미분의 의미

수현이는 그래프를 그린다. y = x²

한 점에서의 접선의 기울기가 그 점에서의 미분값.
- x = 1일 때 기울기 2
- x = 2일 때 기울기 4  
- x = 3일 때 기울기 6

규칙이 보인다! y' = 2x

"미분은 변화율이에요." 선생님이 설명한다. "x가 조금 변할 때 y가 얼마나 변하는지."

## 라이프니츠 vs 뉴턴

"사실 미분은 두 사람이 동시에 발견했어요."

뉴턴: 물리를 위해. 유율(fluxion). 운동의 수학.
라이프니츠: 철학을 위해. 무한소(infinitesimal). dx, dy.

"누가 먼저예요?"

"300년간 싸웠어요. 영국은 뉴턴, 유럽은 라이프니츠."

수현이는 라이프니츠 기호가 더 좋다. dy/dx. 작은 변화의 비율. 직관적이다.

## 적분의 마법

"미분의 반대는?"

넓이 구하기 문제. y = x² 아래 넓이.

작은 직사각형으로 나눈다:
- 폭 0.1: 근사값
- 폭 0.01: 더 정확
- 폭 0.001: 더더욱 정확
- 폭 → 0: 정확한 값!

∫₀¹ x² dx = 1/3

"적분기호 ∫는 Sum의 S를 늘린 거예요. 무한히 많은 무한히 작은 조각의 합."

## 미적분학의 기본정리

수현이는 충격적인 사실을 배운다:

미분과 적분은 역연산이다!

F(x) = ∫ₐˣ f(t) dt 라면, F'(x) = f(x)

"속도를 적분하면 위치, 위치를 미분하면 속도!"

가속도 → (적분) → 속도 → (적분) → 위치
위치 → (미분) → 속도 → (미분) → 가속도

우주가 미적분으로 연결되어 있다.

## 응용의 폭발

수현이는 도처에서 미적분을 본다:

**경제학**: 한계비용 = 총비용의 미분
**생물학**: 개체수 변화율 = dP/dt = rP
**물리학**: F = ma = m(dv/dt)

"세상 모든 변화가 미분방정식이네요?"

그렇다. 뉴턴이 말했듯이 "자연은 미분방정식으로 쓰여 있다."

## e의 등장

복리 계산을 하다가 이상한 수를 만난다.

(1 + 1/n)ⁿ 에서 n → ∞

n = 1: 2
n = 10: 2.594...
n = 100: 2.705...  
n = 1000: 2.717...
n → ∞: 2.71828... = e

"이게 왜 중요해요?"

"eˣ를 미분하면 eˣ예요. 유일해요."

변화율이 자기 자신에 비례하는 것들:
- 방사성 붕괴
- 인구 증가
- 복리 이자

모두 e가 나타난다!

## 테일러 급수

"함수를 다항식으로 근사할 수 있어요."

sin x = x - x³/3! + x⁵/5! - ...

수현이는 계산해본다. x = 0.1일 때:
- x = 0.1
- x - x³/6 = 0.0998...
- 더 많은 항 = 0.09983341... = sin(0.1)

"초월함수를 다항식으로!"

무한급수가 정확한 값으로 수렴한다. 무한이 유한을 만든다.

## 오일러의 정체식

수학사 최고의 공식:

e^(iπ) + 1 = 0

"어떻게 이게 가능해요?"

테일러 급수로:
- e^(ix) = cos x + i sin x
- x = π 대입
- e^(iπ) = cos π + i sin π = -1 + 0i = -1

다섯 개의 가장 중요한 상수(0, 1, e, i, π)가 하나로!

수현이는 아름다움에 전율한다.

## 한계와 도약

미적분도 한계가 있다:
- 불연속 함수는?
- 미분 불가능한 점은?
- 발산하는 급수는?

"하지만 그 한계가 새로운 수학을 낳죠." 선생님이 말한다.

르베스그 적분, 초함수, 분수 미적분...

## 다리 놓기

이제 수현이는 안다:
- 순간은 극한으로 잡을 수 있다
- 미분은 변화율이다
- 적분은 누적이다
- 둘은 서로 역연산이다
- 자연은 미분방정식을 푼다

"미적분 전에는 세상이 멈춰 있었어요." 일기에 쓴다. "이제는 모든 게 흐르고 변해요."

대학 입시가 끝나면 더 깊은 수학을 배울 것이다. 벡터, 행렬, 추상대수...

"변화를 다루는 법을 배웠으니, 이제는 구조를 배울 차례."

---

*"미적분학은 신이 우주를 설계할 때 쓴 언어다."* - 40년 후의 수현

[계속: Level 5 - 공간의 춤 →](L5_Spaces_and_Structures_ko.md)