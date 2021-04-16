# 레거시 코드 활용 전략

## 소프트웨어 변경

### 소프트웨어 코드를 변경하는 네 가지 이유

- 이런 저런 내용이 나오지만, 결국 어떤 동작을 변경할 때 다른 동작들에는 영향이 없어야 한다는 이야기.
- 그리고 안전한 변경을 위해 영향 범위를 정확히 이해하는 것이 중요.

### 위험한 변경

> 클래스와 메서드를 새로 생성하지 않으면, 기존의 클래스와 메서드는 갈수록 비대해져서 결국 코드를 이해하기 어려운 지경에 이른다. 대규모 시스템에서 변경 작업을 할 때는 작업 대상을 이해하기 위해 어느 정도 시간이 걸릴 것을 감안해야 한다. 이때 좋은 시스템과 나쁜 시스템의 차이는 좋은 시스템일 경우 시스템에 익숙해질수록 마음이 편안해지고 변경 작업에 확신을 갖게 되지만, 나쁜 시스템의 경우 시스템을 알게 될수록 코드 변경 작업이 마치 호랑이를 피하기 위해 절벽에서 뛰어내리는 것 같은 느낌을 받게 된다는 점이다.

## 피드백 활용

### 시스템 변경의 방식

- 시스템 변경의 방식을 크게 2가지로 나눔.
- 편집 후 기도하기(Edit and Pray)
- 보호 후 수정하기(Cover and Modify)
- 편집 후 기도하기가 일반적.
- 이 방식에서 매우 신중하게 계획을 세우고, 대상 코드를 충분히 이해하는 등의 노력이 포함됨.
- 그러나 아무리 신중하게 주의를 기울여도 그에 비례해서 안정성이 높아진다는 보장은 없음.
- 이를 극복하기 위해 먼저 보호하는 것이 필요.

### 보호망

- 이 보호망은 다름 아닌 테스트.
- 작업 결과의 정확성을 보여주기 위함이기도 하며,
- 변경된 부분을 발견하기 위함이기도 함.
- 변경하고자 하는 부분만을, 의도대로 변경했는지 확인하는 것.

### 회귀 테스트

- 보통 회귀 테스트가 이를 위한 도구.
- 그러나 회귀 테스트는 애플리케이션 수준의 테스트.
- 피드백 주기가 길고, 빠른 문제 디버깅이 어려움.
- 커버리지도 높지 않은 것이 일반적.
- 그럼에도 불구하고 이런 상위 수준의 테스트는 다수 클래스들의 동작을 한 번에 확인할 수 있는 이점이 있음.

### 단위 테스트

- 단위테스트는 이 단점들을 보완.
- 실행 속도가 빠름.
- 좀 더 빠른 오류 위치 파악에 도움이 됨.

### 레거시 코드를 변경하는 순서

1. 변경 지점을 식별한다.
2. 테스트 루틴을 작성할 위치를 찾는다.
3. 의존 관계를 제거한다.
4. 테스트 루틴을 작성한다.
5. 변경 및 리팩토링을 수행한다.

### 테스트 작성 전 의존 관계의 제거

- 변경에 앞서 테스트를 작성해야 하는데,
- 깊고 거대한 의존성을 테스트 작성을 어렵게 함.
- 그래서 이 의존성을 제거하기 위한 리팩토링을 먼저 진행.
- 기본 타입 매개변수(primitive parameter)와 인터페이스 추출(extract interface)라 부르는 방법 사용.
- 이 같은 리팩토링을 테스트 없이 수행해도 충분히 안전하다고 저자가 말하는 게 인상적.