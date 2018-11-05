# 책 소개

- [생각한다면 과학자처럼](http://book.daum.net/detail/book.do?bookid=KOR9791160502763)
- 데이비드 헬펀드
- 과학자 직업을 가진 저자가 평소 자신의 생각하는 방식들을 소개함.

# 확률을 계산하는 간단한 규칙들

## 퀘이사의 특별한 배열

Halton Arp 박사는 근처 은하들과 나란히 배열된 퀘이사들을 발견했는데, 이들은 1:3:4의 비율 간격으로 일직선 상에 배열됨. 이런 배열이 우연으로 보기는 어렵고, 따라서 무언가 새로운 물리 법칙이 있음에 틀림없다고 주장함. 하지만 정말 그럴까?

## 드문 일은 생겨난다

1. 패턴을 찾아내는 것이 자연스러운 진화의 결과이긴 하지만, 이것이 강박적으로 되는 것은 비생산적.
2. 사전<sup>priori</sup> 계산과 사후<sup>posteriori</sup> 계산의 차이를 이해해야 함.
3. 일어나지 않을 법한 일들이 생기는 현상을 좀 더 정확하게 바라볼 수 있음.
4. 물론, 사후 탐색이 어떤 새로운 발견이나 원인을 제시해줄 수 있긴 함.
5. 하지만 이는 사전 계산으로 이어져야만 하고, 새로운 데이터를 통해 검증받아야 함.

## 드문 일들

기초적인 확률 계산들을 이용하면, 우리의 잘못된 직관들을 바로 잡을 수 있음. 개인적으로 인상 깊었던 드문 일들을 기록함.

### 60명의 생일

Q. 수업을 들으러 온 학생이 60명이다. 생일의 총 가짓수는 365이다. 적어도 두 학생의 생일이 같은 경우는 얼마나 되는가? 만약, 20:1의 배당률을 제시한다면 수락하겠는가? 그러니까, 1/20의 확률정도가 되겠는가?

A. 이 문제는 여집합을 활용하는 편이 쉽다. 아래의 식을 이용하면, 1 - 0.0059 = 99.4가 된다. 즉, 이길 가능성이 200:1이다.

```
P(적어도 두 명 이상의 생일이 같다)
= 1 - P(모두의 생일이 다르다)
= 1 - [1 * 364/365 * 363/365 * … * (365 - n - 1)/365]
= 1 - \sum_{x=0}^{n}{\frac{(365 - n)!}{365^n}}
```

### 7번 연속 주가 예측 성공

Q. 2주마다 나에게 주가 예측 메일이 온다. 그런데 7번 받은 메일이 모두 예측에 성공했다! 8번째 메일을 열어보았는데, 메일을 더 받고 싶으면 300달러를 내야한다고 한다. 300달러를 내겠는가?

A. 이 확률은 1/2^7이며, 따라서 1퍼센트 미만이다. 놀라운가? 하지만, 만약 이 메일이 맨 처음에 2,000명에게 보내졌다면 어떠한가? 7번째 메일을 받았다면, 나와 같은 사람이 32명이나 된다. 메일을 보내는 사람은 2,000명의 메일만을 수집하는 수고만으로 꽤 많은 수익을 올릴 수 있다. (물론 스팸에 걸리지 않는다면 말이다)

### 그 사람의 이름도 조너선

Q. 조너선은 오늘 새 직장에 첫 출근한다. 출근후 옆에 앉은 여자와 저녁 식사를 하게 되었는데, 그녀의 이름은 브라이니다. 그가 만난 사람 중에도 같은 이름이 있었다. 그러려니 하고 얘기를 나누었는데, 그와 생일이 똑같다는 것을 알게 된다. 그런데 더 놀라운 것은, 그녀의 동생의 이름이 나와 같은 조너선이다!

A. 이 사례 또한, 확률적으로 따지고 보면 별로 놀라운 일이 아니다. 하지만 같은 이름이 동생이 아니라, 아버지라면? 혹은 어머니라면? 강아지라면? 삼촌이라면? 일어날 수 있는 우연의 수는 매우 많다. 하지만, 우연들의 집합을 {나, 그녀, 남동생}으로 한정해버렸다. 전형적인 사후 통계 사례이다.