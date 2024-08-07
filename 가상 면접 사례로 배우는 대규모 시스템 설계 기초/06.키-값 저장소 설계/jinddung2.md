## 다루는 내용

키-값의 핵심은 키. 키는 짧을수록 좋음.

# 문제 이해 및 설계 범위 확정

- 읽기-쓰기-메모리 사용량 사이에서 균형 찾기
- 데이터 일관성의 가용성 사이에 타협적 결정을 내린 설계

### 단일 서버 키-값 저장소

- 키-값 저장소를 메모리에 해시 테이블로 저장
- 메모리 용량을 줄이기 위한 방법
  - 데이터 압축
  - 자주 쓰이는 데이터만 메모리에 두고 나머지는 디스크에 저장

### **CAP 정리**

CAP 정리는 데이터 일관성, 가용성, 파티션 감내라는 세 가지 요구사항을 동시에 만족하는 분산 시스템을 설계하는 것은 불가능하다는 정리다.

- 데이터 일관성: 분산 시스템에 접속하는 모든 클라이언트는 어떤 노드에 접속했느냐에 관계없이 언제나 같은 데이터를 보게 되어야 한다.
- 가용성: 분산 시스템에 접속하는 클라이언트는 일부 노드에 장애가 발생하더라도 항상 응답을 받을 수 있어야 한다.
- 파티션 감내: 파티션 감내는 네트워크에 파티션이 생기더라도 시스템은 계속 동작하여야 한다는 것을 뜻한다.
- CP 시스템: 일관성과 파티션 감내를 지원하는 키-값 저장소. 가용성을 희생한다. ex) 몽고DB
- AP 시스템: 가용성과 파티션 감내를 지원하는 키-값 저장소. 데이터 일관성을 희생한다. ex) 카산드라
- CA 시스템: 일관성과 가용성을 지원하는 키-값 저장소. 네트워크 장애는 필하수 없음 → 파티션 감내는 지원하지 않는다.

## **이상적 상태**

n1 <> n2 <> n3 간 복제가 되는 환경

## **실세계의 분산 시스템**

분산 시스템은 파티션 문제를 피할 수 없다.

파티션 문제가 발생하면 일관성과 가용성 사이에서 하나를 선택해야 한다.

가용성 대신 일관성을 선택 → n1과 n2에 대해 쓰기 연산을 중단시켜야 하는데, 그렇게 하면 가용성이 깨진다.

일관성 대신 가용성을 선택 → 설사 낡은 데이터를 반환할 위험이 있더라도 계속 읽기 연산을 허용해야 한다.

## 시스템 컴포넌트

### **데이터 파티션**

**고려해야 할 두 가지 문제**

- 데이터를 여러 서버에 고르게 분산할 수 있는가?
- 노드가 추가되거나 삭제될 때 데이터의 이동을 최소화할 수 있는가?

**안정 해시 장점**

- 규모 확장 자동화: 시스템 부하에 따라 서버가 자동 추가 및 삭제
- 다양성: 각 서버의 용량에 맞게 가상 노드의 수를 조정. 고성능 서버는 더 많은 가상 노드를 갖도록 설정

**데이터 다중화**

고가용성과 안정성 확보하기 위해 데이터를 N(튜닝 가능한 값)개 서버에 비동기적으로 다중화

노드를 선택할 때 같은 물리 서버를 중복 선택하지 않아야 함

안정성을 담보하기 위해 데이터의 사본은 다른 센터의 서버에 보관. 센터들은 고속 네트워크로 연결.

**데이터 일관성**

여러 노드에 다중화된 데이터는 적절히 동기화가 되어야 함.

정족수 합의 프로토콜을 사용하면 읽기/쓰기 연산 모두 일관성 보장.

W=1 → Not 데이터가 한 대 서버에 기록된다는 뜻 But 중재자는 최소 한 대 서버로부터 쓰기 성공 응답이 필요

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/440ca016-b2bb-4b23-9715-d6e87f7d0da2/8e46cf27-14d7-4d08-aba5-6e3096d2ed76/Untitled.png)

**일관성 모델**

데이터 일관성의 수준을 결정

- 강한 일관성: 모든 읽기 연산은 가장 최근에 갱신된 결과를 반환한다(=클라이언트는 낡은 데이터를 안본다).
  → 모든 사본에 현재 쓰기 연산의 결과가 반영될 때 까지 해당 데이터에 대한 읽기/쓰기 금지
  (=고가용성에 적합X)
- 약한 일관성: 읽기 연산은 가장 최근에 갱신된 결과를 반환하지 못할 수도 있다.
- 최종 일관성: 약한 일관성의 한 형태로, 갱신 결과가 결국에는 모든 사본에 반영되는 모델이다.
  ex) 다이나모, 카산드라

비 일관성 해소 기법: 데이터 버저닝

- 데이터를 다중화 → 가용성은 높아지나 사본 간 일관성이 깨질 가능성 증가
- 버저닝과 벡터 시계가 이러한 문제를 해소하기 위한 기술
  (낙관적 락과 비슷한거 같은데?)

**장애 처리 →** 장애 감지와 장애 해소 전략

**장애 감지**

가십 프로토콜(gossip protocol)

- 각 노드는 주기적으로 자신의 박동 카운터 증가
- 어떤 멤버의 박동 카운터 값이 지정된 시간동안 갱신되지 않으면 장애 상태로 간주

**일시 장애 처리**

가십 프로토콜로 장애를 감지한 시스템은 가용성을 보장하기 위해 필요한 조치

- 엄격한 정족수: 읽기와 쓰기 연산 금지
- 느슨한 정족수: 정족수 요구사항을 강제하는 대신, 쓰기 연산을 수행할 W개의 건강한 서버와 읽기 연산을 수행할 R개의 건강한 서버를 해시 링에서 선택. 장애 상태인 서버는 무시

네트워크나 서버 문제로 장애 상태인 서보로 가는 요청은 다른 서버가 잠시 맡아 처리 → 그동안 발생한 변경사항은 해당 서버가 복구되었을 때 일괄 반영하여 데이터 일관성 보존 → 이를 위해 임시로 쓰기 연산을 처리한 서버에 관한 단서를 남김(=임시 위탁 기법)

**영구 장애 처리**

반-엔트로피 프로토콜 구현하여 사본들을 동기화

**데이터 센터 장애처리**

**쓰기경로**
