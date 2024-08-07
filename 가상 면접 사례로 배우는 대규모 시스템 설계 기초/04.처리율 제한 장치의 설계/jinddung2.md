(API 게이트웨이: 처리율 제한을 지원하는 미들웨어)

네트워크 시스템에서 처리율 제한 장치: 클라이언트 또는 서비스가 보내는 트래픽의 처리율을 제어하기 위한 장치

API 요청 횟수 > 제한 장치에 정의된 임계치 ? 임계치 초과한 모든 호출 처리 중단 : 그대로 처리

## 1단계 문제 이해 및 설계 범위 확정

처리율 제한 장치의 효과

- Dos 공격에 의한 자원 고갈 방지
- 비용 절감. 추가 요청에 대한 처리를 제한하여 서버를 많이 두지 않아도 됨
- 서버 과부하 방지

## 2단계 개략적 설게안 제시 및 동의 구하기

클라이언트-서버 통신 모델 사용하자.

**처리율 제한 장치는 어디에 둘 것인가?**

- 클라이언트 측에 둔다? → 클라이언트 요청은 쉽게 위변조 가능 + 모든 클라이언트 요청 막는 것은 불가
- 서버측에 둔다? → API를 만들기 보다는 처리율 제한.미들웨어를 만들어서 통제

**처리율 제한 알고리즘**

- 토큰 버킷 알고리즘 - 클라우드 서비스 제공자
  - 클라우드 자원의 과도한 사용을 방지하고, 공정한 자원 배분을 보장
  - [AWS](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-request-throttling.html)
  - [Google Cloud](https://cloud.google.com/armor/docs/rate-limiting-overview?hl=ko)
- 누출 버킷 알고리즘
- 고정 윈도 카운터 알고리즘
- 이동 윈도 로깅 알고리즘 - 대규모 웹 서비스
  - 처리율 제한을 통해 API 남용을 방지하고 서비스 안정성을 확보
  - [Twitter](https://developer.x.com/en/docs/twitter-api/rate-limits)
- 이동 윈도 카운터 알고리즘 (고정 윈도 카운터 + 이동 윈도 로깅)

## 3단계 상세 설계

**처리율 제한 규칙**

**처리율 한도 초과 트래픽의 처리**

- 어떤 요청이 한도 제한에 걸리면 API는 HTTP 429 응답하기
- 경우에 따라 한도 제한에 걸린 메시지를 큐에 보관

**처리율 제한 장치가 사용하는 HTTP 헤더**

- X-Ratelimit-Remaining: 윈도 내에 남은 처리 가능 요청의 수
- X-Ratelimit-Limit: 매 윈도마다 클라이언트가 전송할 수 있는 요청의 수
- X-Ratelimit-Retry-After: 한도 제한에 걸리지 않으려면 몇 초 뒤에 요청을 다시 보내야 하는지 알림

---

상세 설계 부분

**분산 환경에서의 처리율 제한 장치의 구현**

- 여러 대의 서버와 병렬 스레드를 지원하도록 시스템을 확장하는 것은 또 다른 문제

**경쟁 조건**

- 경쟁 조건 문제를 해결하는 가장 널리 알려진 해결책은 락이다. 하지만 락은 시스템의 성능을 상당히 떨어뜨린다는 문제 발생
- 루아 스크립트와 정렬 집합이라 불리는 레디스 자료구조를 쓰는 것

**동기화 이슈**

- 동기화는 분산 환경에서 고려해야 할 또 다른 중요한 요소
- 수백만 사용자를 지원하려면 한 대의 처리율 제한 장치 서버로는 충분하지 않을 수 있음
- 처리율 제한 장치 서버를 여러 대 두게 되면 동기화가 필요해진다.
- 고정 세션 → 확장 가능하지도 않고 유연하지도 않음
- 레디스와 같은 중앙 집중형 데이터 저장소

**성능 최적화**

- 여러 데이터 센터를 지원 → 클라이언트가 데이터센터에 멀면 latency 증가 → 많은 데이터 센터 필요
- 제한 장치 간에 데이터 동기화할 때 일관성 모델 사용

**모니터링**

- 체택된 처리율 제한 알고리즘이 효과적
- 정의한 처리율 제한 규칙이 효과적

**4단계 마무리**

- 경성(hard) : 요청의 개수는 임계치를 절대 넘어설 수 없다.
- 연성(soft) : 요청 개수는 잠시 동안은 임계치를 넘어설 수 있다.
- 다양한 계층에서의 처리율 제한
- 처리율 제한을 회피하는 방법

---

## 토론 주제 모아보기

**처리율 한도 초과 트래픽의 처리할 때 어떤 요청이 한도 제한에 걸릴 때
API HTTP 429 응답vs 메시지 큐에 보관**

기준: 요청의 중요도(결제, 주문), 실시간 처리 요구사항

### 대규모 웹 서비스에서 슬라이딩 윈도우 로그 알고리즘을 선택한 이유

**슬라이딩 윈도우 로그 알고리즘**은 각 요청의 타임스탬프를 로그에 기록하고, 슬라이딩 윈도우 내의 요청 수를 계산하여 현재 요청이 허용 가능한지 여부를 결정합니다.

### 클라우드 서비스 제공자가 토큰 버킷 알고리즘을 사용하는 이유

클라우드 서비스 제공자는 **토큰 버킷 알고리즘**을 사용합니다. 이 알고리즘은 일정 시간 동안 고정된 양의 토큰을 생성하고, 각 요청 시 토큰을 소비합니다.

이 알고리즘은 특히 높은 유연성과 성능을 제공하기 때문에 클라우드 환경에서의 동시성 제어에 유리합니다 ([Qwiklabs](https://www.cloudskillsboost.google/focuses/21571?parent=catalog)).

현재 요청이 허용 가능한지 여부를 결정합니다. 알고리즘의 선택 이유는 다음과 같습니다.

1. **정확성**: 최근 요청 수를 정확하게 추적하여 공정한 요청 허용을 보장합니다.
2. **유연성**: 실시간으로 변동하는 요청 패턴에 빠르게 적응합니다.
3. **버스트 트래픽 처리**: 짧은 시간에 많은 요청이 들어올 때 효과적으로 제어할 수 있습니다 ([Codementor](https://www.codementor.io/@arpitbhayani/system-design-sliding-window-based-rate-limiter-157x7sburi)).
