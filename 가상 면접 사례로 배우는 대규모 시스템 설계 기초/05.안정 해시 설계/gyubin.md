# 5. 안정 해시
 
## 안정 해시
안정 해시를 알기 전에 보편적으로 서버에 부하를 균등하게 나누는 방법은 다음과 같다.

서버에 부하를 균등하게 나누는 보편적인 방법은 hash(key) % N(서버 풀의 크기)의 해시 함수로 서버의 인덱스를 구하여 사용하는 것이다. 이 방법은 서버 풀이 고정되어 있을 때는 잘 동작한다. 하지만 서버가 추가되거나 기존 서버의 문제로 인해 중단이 되거나 삭제가 되는 경우에 서버 풀 크기인 N 값이 바뀌게 되고 그 결과 서버의 인덱스까지 바뀌게 되는 해시 키 재배치(rehash) 문제가 발생하고 최종적으로 캐시미스 문제로 이어지게 된다.

위와 같은 문제를 해결하면서 요청 또는 데이터를 서버에 균등하게 나눠 분산 시스템의 확장성과 안정성을 높이는 기술이 안정 해시이다.

#### 안정 해시 함수의 특성
- 가상 노드 사용
    - 실제 노드는 여러 개의 가상 노드로 표현되며, 데이터 키를 더 고르게 분포되도록 한다.
- 해시 링 구조
    - 해시 공간을 원형으로 간주하고 각 노드와 데이터 키를 이 해시 링 위에 배치한다.
- 부분적 키 재배치
    - 일반 해시 함수와 다르게 노드의 추가 및 제거 시 일부 키만 새로운 노드로 재배치시켜 재배치되는 키의 수를 최소화한다.
#### 분산 시스템에서 자주 사용되는 해시 함수
- MurmurHash
    - MurmurHash는 비암호학적 해시 함수로, 빠른 해싱 성능을 제공.
    - 분산 시스템 및 데이터베이스에서 많이 사용.
    - 사용처
        - mongoDB
            - document key, index key에 사용
        - Apache Spark
            - 데이터 분할 및 파티셔닝에 사용
        - Apache Kafka
            - 파티션 할당에 사용
        - Elasticsearch
            - document ID에 사용
- CityHash
    - Google에서 개발한 비암호학적 해시 함수로, 매우 빠른 해싱 성능을 제공.
    - 대규모 데이터 분산 시스템에서 자주 사용.
- chris moos의 hash-ring
    - SHA-1, MD5 해시 함수 지원
    - 디스코드에서 사용하다가 chris moos의 hash-ring을 기반으로 ExHashRing을 만듦.
    - https://github.com/chrismoos/hash-ring
    - https://discord.com/blog/how-discord-scaled-elixir-to-5-000-000-concurrent-users