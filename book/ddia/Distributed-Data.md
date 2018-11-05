# Part II. Distributed Data

Part I 이 데이터가 단일 노드에 저장될 때의 관점이라면, Part II는 여러 머신에서 일어나는 저장과 조회에 관한 것. 여러 머신에 데이터를 분산하려는 이유는 여러 가지가 있음.

- Scalability: 데이터 볼륨, 읽기 부하, 쓰기 부하가 커지면 하나의 머신으로는 다루기 어려움.
- Fault tolerance/high availabilty: 한 개의 장비가 고장나더라도 작업을 계속할 수 있는 가용성.
- Latency: 사용자가 전세계에 걸쳐 있다면, 지역적으로 가까운 데이터센터들을 제공할 수 있음.

## Scaling to Higher Load

### Shared-memory Architecture

- 더 많은 부하를 다루는 가장 쉬운 방법은 좀 더 강한 머신을 사는 것.
- 많은 CPU, RAM 칩, 디스크들은 하나의 OS에 묶일 수 있고,
- 서로 연결<sup>interconnect</sup>되어 CPU가 접근하게 할 수 있음.
- 문제점은 비용이 많이 증가한다는 것.
- 또한, 장애 내성이 높지 않다.

### Shared-disk Architecture

- 여러 머신들 사이에서 공유되는 디스크 배열에 데이터를 저장하는 것.
- 이들 디스크들은 빠른 네트워크로 연결됨.
- 데이터 웨어하우징 작업부하를 위해 사용되지만,
- 경합과 잠금 오버헤드는 확장성을 제한하는 요소가 됨.

### Shared-nothing Architectures

이 책에서는 이 접근법에 초점을 둠. 최선의 선택이어서가 아니라, 어플리케이션 개발자들이 신경써야 하는 것이 가장 많은 접근법이기 때문.

- 수평적 스케일링 또는 스케일링 아웃이라고 불리는 방법.
- DB 소프트웨어를 실행시키는 각 머신 또는 가상 머신은 노드<sup>node</sup>라고 부름.
- 각 노드는 자신의 CPU, RAM, 디스크를 독립적으로 사용.
- 노드 간의 조율은 소프트웨어 레벨에서 이뤄지고, 평범함 네트워크를 사용함.

### Replication Versus Partitioning

데이터가 여러 노드에 분산되는 방식에는 2가지가 존재. 이 둘은 별개의 메커니즘이지만, (당연하게도) 함께 사용될 수 있음.

**Replication**

- 서로 다른 위치에 있는 몇 개의 서로 다른 노드에 동일한 데이터의 복제본을 유지하는 것.
- 이는 redundancy를 제공함. 즉, 한 노드가 가용하지 않더라도, 다른 노드를 통해 데이터를 제공 받을 수 있음.
- 또한, 성능을 높여주기도.

**Partitioning**

- 큰 DB를 파티션이라고 불리는 몇 개의 부분집합으로 나누는 것.
- 이 부분집합은 서로 다른 노드에 할당됨.

# Replication

데이터 레플리케이션이란, 같은 데이터의 복제본을, 네트워크로 연결된 여러 머신에 유지하는 것. 레플리케이션의 이유는 여러 가지가 있음.

- 지역적으로 사용자에게 가까운 데이터를 제공하기 위해서.
- 일부가 고장나더라도 시스템은 계속 동작할 수 있게 하기 위해서.
- 읽기 전용 머신을 제공해서 읽기 성능을 높이기 위해서.

레플리케이션의 어려움은 복제된 데이터의 변경을 다루는 것에 있음. 노드간의 변경을 복제하는, 널리 알려진 3가지 알고리즘은 single-leader, multi-leader, leaderless(대부분의 분산 데이터베이스가 이 중 하나를 사용함).

레플리케이션 시 고려해야 하는 트레이드 오프는 다양함. 예컨대, 동기/비동기 복제, 실패한 레플리카를 어떻게 다룰 것인가 등.

## Leaders and Followers

DB의 복사본을 저장하는 각 노드를 가리켜 레플리카<sup>replica</sup>라고 부름. 여러 레플리카가 있다면, 아래 질문은 필수불가결.

> how do we ensure that all the data ends up on all the replicas?

DB에 대한 모든 쓰기는 모든 레플리카에서도 함께 처리되어야 함. 그렇지 않으면 같은 데이터를 가지고 있을 수 없음. 가장 흔한 해결책은 leader-based replication. 이는 active/passive 또는 master-slave replication이라고도 알려져 있음.

![](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0501.png)

1. 레플리카 중 하나가 리더로 지정됨. 클라이언트의 요청을 이 리더가 받게 됨.
2. 나머지 레플리카는 팔로워(read replicas, slaves, secondaries, hot standbys). 리더가 새로운 데이터를 저장할 때면, 이 변경사항을 레플리케이션 로그<sup>replication log</sup> 또는 변경 스트림<sup>change stream</sup>의 일부로써 모든 팔로워에게 전달함.
3. 읽기는 리더나 팔로워 중 누구에게나 할 수 있음. 하지만 쓰기는 리더에게만 이뤄져야 함.

많은 관계형 DB들이 이 기능을 기본으로 제공함. 또한, Kafka나 RabbitMQ와 같은 분산 메시지 브로커들도 제공. 네트워크 파일시스템이나 복제된 블럭 디바이스들도 이를 제공.

### Synchronous Versus Asynchronous Replication

아래 이미지는 동기 그리고 비동기 레플리케이션을 보여줌. 참고로, [PostgreSQL Replication Master Server Configuration](https://www.postgresql.org/docs/10/static/runtime-config-replication.html#RUNTIME-CONFIG-REPLICATION-MASTER)을 보면, 특정 노드만을 동기적 레플리케이션 대상으로 지정 가능함.

![](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0502.png)

**semi-synchronous**

동기로 할 경우 대부분 빠른 시간(1초 이내)에 레플리케이션이 끝나지만, 얼마나 더 걸릴 수도 있는지는 모르는 일. 예컨대, 팔로워 서버가 고장 나서 몇 분간 회복되고 있을 수도 있음. 그렇다고 모든 노드를 비동기로 하면, 리더 노드가 고장 났을 때, 최신 복제본을 가진 다른 노드가 있음을 보장하지 못함. 따라서, 1개의 팔로워만을 동기로 레플리케이션하는 것이 실용적. 최신 복제본을 가진 노드를 적어도 2개를 유지하면서, 팔로워들의 고장이 리더에게 미치는 영향을 최소화. 이를 가리켜 *semi-synchronous*라고 부름.

많은 리더 기반 레플리케이션이 완전한 비동기로 설정된다고 함. 내구성이 약해지는 문제<sup>weakining durability</sup>에도 불구. 특히, 팔로워가 많거나 지역적으로 분산된 경우에 더더욱.

데이터를 잃지 않으면서도 좋은 성능과 가용성을 보장하는 방법은 계속 연구중이고, chain replication은 그 결과물 중 하나. Microsoft Azure Storage 등에서 사용된다고 함.

### Setting Up New Followers

새로운 팔로워를 셋업해야 할 수 있음. 단순히 데이터를 복사하는 것으로는 충분치 않음. 데이터를 복사하는 중간에 리더 노드에 새로운 변경사항이 생길 수 있기 때문. 잠금을 사용하는 것은 가용성 측면에서 좋은 선택은 아님. 대신, 아래와 같이 해볼 수 있다.

1. 지속적으로 리더 DB의 스냅샷을 생성.
2. 새로운 팔로워 노드로 스냅샷을 복사.
3. 팔로워는 리더에게 특정 스냅샷 이후로 생긴 모든 데이터 변경을 요청. *log sequence number*(PostgreSQL) 또는 *binlog coordinates*(MySQL)라고 불리는 레플리케이션 로그의 정확한 위치를 알고 있어야 함.
4. 모든 변경을 다 따라잡았다면 클러스터로 투입.

위 절차는 DB마다 상당한 차이를 보일 수도 있음.

### Handling Node Outages

개별 노드가 고장나더라도 전체 시스템은 계속 동작하면서, 노드의 고장이 미치는 영향을 최소화하는 방법은 뭘까? 그러니까, 리더 기반 레플리케이션이 높은 가용성을 가지게 하려면 어떻게 해야 하는가?

#### Follower Failure: Catch-Up Recovery

팔로워 노드에 크래시가 일어나거나, 리더로부터의 네트워크가 단절되는 등의 문제가 있을 수 있음. 반영을 실패한 데이터 로그 지점부터 다시 반영을 시작하거나, 데이터 로그가 부족하다면 새로 추가된 데이터 로그를 리더로부터 받아오면 됨.

#### Leader Failure: Failover

페일오버<sup>failover</sup>는 까다로운 일. 보통 아래 절차를 밟음.

1. 리더가 고장났는지 여부를 판단.
   - 완전하게 고장을 파악할 수 있는 방법은 없으나,
   - 일반적으로 타임아웃을 사용.
2. 새로운 리더 선출.
   - 선거<sup>election</sup>(남아 있는 레플리카들에 의해 결정),
   - 또는 임명<sup>appoint</sup>(미리 지정된 컨트롤러 노드에 의해 결정)을 통해 선출됨.
   - 가장 좋은 후보는 가장 최신 데이터를 가진 레플리카.
3. 새로운 리더를 사용하도록 시스템 재설정.
   - 클라이언트는 새로운 리더에게 쓰기 요청을 보내야 함.
   - 만약, 오래된 리더가 다시 돌아오면, 시스템이 이를 팔로워라고 인식시켜 주어야 함.

페일오버는 다음과 같은 문제를 일으킬 수 있음.

- 비동기 레플리케이션이라면, 이전 리더의 데이터가 일부 유실될 수 있음. 가장 흔한 해결책(?)은 이 데이터를 버리는 것. 하지만, 이는 클라이언트의 내구성 기대를 위반.
- DB 데이터가 공유되는 또 다른 시스템이 있다면, 이러한 데이터 버림은 위험할 수 있음. [Github availability this week](https://blog.github.com/2012-09-14-github-availability-this-week/) 사례 참고.
- 두 개의 노드가 모두 스스로를 리더라고 생각하는 문제도 있음. 이렇게 되면 데이터가 유실되거나 충돌되기도 함. 이를 피하는 방법 중의 하나로, 두 개의 리더가 감지되면 하나를 중단시킴(단, 두 개의 노드가 모두 중단되는 위험도 있음).
- 리더가 죽었는지를 판별하는 타임아웃은 얼마가 적정한가? 긴 타임아웃은 복구하는 시간이 길다는 것을 의미. 반면, 너무 짧으면 (순간적인 부하 증가나 네트워크의 일시적 단절 등의 이유로) 불필요한 페일오버를 일으킬 수 있음.

어느 하나 쉬운 해결책이 없으며, 이런 이유로 어떤 운영팀은 수동 페일오버를 선호하기도 함. 노드 실패, 신뢰할 수 없는 네트워크, 레플리카 일관성에 대한 트레이드 오프, 내구성, 가용성, 응답 지연은 모두 분산 시스템이 가진 근본적 문제. 8장과 9장에서 이들을 좀 더 깊이 있게 다룰 예정.

### Implementation of Replication Logs

리더 기반 레플리케이션은 어떻게 동작하는가?

#### Statement-Based Replication

리더가 모든 쓰기 요청(명령문<sup>statement</sup>)을 로깅하고, 이를 팔로워들에게 보냄. 관계형 DB의 경우 `INSERT`, `DELETE` 등의 명령문이 여기에 해당. 가장 합리적인 선택으로 보이지만 아래와 같은 문제들이 발생할 수 있음.

- `NOW()`, `RAND()` 등의 비결정적 함수는 레플리카에서 다른 값을 만들어 냄.
- 명령문이 자동증가 컬럼을 사용하거나, `UPDATE ... WHERE ...`과 같이 존재하는 데이터에 의존하는 경우, 명령문은 레플리카에서 동일한 순서로 실행되어야 함. 이는 복수의 트랜잭션을 동시에 실행시키지 못하게 하는 제약.

이런 문제를 회피할 수 있긴 함. 에를 들어, 비결정적인 함수들의 호출을 고정된 명령문으로 치환해서 로그로 남길 수도 있음. 하지만 이외에도 여러 가지 엣지 케이스<sup>edge case</sup>들이 많아서 다른 방식들이 많이 사용됨. MySQL 5.1 이전 버전에서는 이 방식이 기본으로 사용되었음. 하지만, 현재는 로우 기반 레플리케이션이 기본값. VoltDB는 명령어 기반 레플리케이션을 사용하긴 하지만, 트랜잭션이 결정적이도록 요구.

#### Write-Ahead Log (WAL) Shipping

저장소 엔진은 모든 쓰기를 로그에 덧붙임. 로그 구조화 저장소 엔진은 이 방식이 기본이고, B 트리인 경우에도 이 WAL에 모든 변경이 가장 먼저 기록됨. 이 데이터를 레플리케이션 하는 것. 단점은 매우 저수준의 데이터라는 것. 즉, 레플리케이션이 특정 저장소와 강하게 결합됨. 버전 차이도 이런 문제를 만들 수 있음. 따라서, 다운 타임 없이는 버전 업그레이드가 불가.

#### Logical (Row-Based) Log Replication

한 가지 대안은 레플리케이션과 저장소를 위한 각각의 로그 포맷을 사용하는 것. 이를 가리켜 *logical log*라고 부름. 저장소 엔진의 물리적 데이터 표현과 구분하기 위함. 이 논리적 로그는 DB 테이블에 대한 쓰기를 표현하는 순차적 레코드.

- 삽입된 로우 − 모든 컬럼들에 대한 새로운 값을 로그가 포함함.
- 삭제된 로우 − 로우를 충분히 식별할 수 있을 정도의 정보를 가짐.
- 갱신된 로우 − 로우를 충분히 식별할 수 있을 정도의 정보와 모든 컬럼에 대한 새로운 값을 포함.

여러 행을 수정하는 트랜잭션은 이런 로그 레코드들을 여러 개 생성. 그리고 트랜잭션이 커밋되었음을 가리키는 레코드를 뒤이어 남김. [MySQL의 *binlog*](https://dev.mysql.com/doc/internals/en/binary-log-overview.html)는 이 접근법을 사용함.

저장소 엔진 내부와의 결합도가 낮기 때문에, 버전 차이는 물론이거니와 서로 다른 저장소 엔진 간에도 호환이 가능. 외부 애플리케이션이 파싱하는 것도 쉽다. 오프라인 분석을 위해 데이터 웨어하우스 같은 외부 시스템에 데이터를 보낼 때 유리. 종종 이를 가리켜 *change data capture*라고 부름.

#### Trigger-Based Replication

지금까지 살펴본 방식보다도 더 높은 유연성이 요구된다면, 애플리케이션 코드가 레플리케이션의 역할을 수행하게 할 수도 있음. 많은 관계형 데이터베이스에서는 *trigger* 그리고 *stored procedure*가 사용됨. *trigger*는 데이터 변경이 발생하면 등록된 커스텀 애플리케이션 코드를 자동으로 실행시켜줌. 레플리케이션에 바로 이 *trigger*를 사용할 수 있음. 하지만, 일반적으로 훨씬 더 큰 부하를 수반함. 버그도 많을 수 있고, 한계도 많음.

## Problems with Replication Lag

앞서 계속 설명된 것 처럼, 레플리케이션의 이유는 아래와 같음.

1. fault tolerance (한 머신이 고장나도 다른 머신이 작업을 대체)
2. scalability (하나의 머신에서 다룰 수 있는 것 보다 큰 요청을 처리)
3. latency (사용자에게 지역적으로 가까운 곳에 레플리카를 위치)

이를 위해 여러 팔로워를 추가할 수 있으며, 이 때는 레플리케이션을 동기로 할지 비동기로 할지 선택해야 함.

1. synchronous − 팔로워 중 하나라도 문제 있는 경우 쓰기가 동작할 수 없음. 팔로워가 늘어날 수록 이 문제의 가능성은 커짐.
2. asnychronous − 팔로워 노드에서 최신 데이터를 제공함을 보장하지 못함. 이러한 비일관성은 일시적이긴 함. 결과적 일관성<sup>eventual consistency</sup>.

참고로, "eventually"라는 용어는 의도적으로 모호함을 가짐. 그러니까, 레플리케이션 지연<sup>replication lag</sup>이 얼마나 될지에 대한 일반적인 제약이 없다는 의미. 네트워크 상황이나 가용한 OS 리소스 상태에 따라 수초에서 수분까지 걸릴 수도.

동기 옵션은 사용 X. 최대 1대까지만을 동기로 고려. 뒤이어 다루는 내용도 모두 비동기를 사용하는 것을 전제로 하고 있음.

### Reading Your Own Writes

아래 그림은 쓰기 데이터가 바로 보이지 않을 수도 있는 비일관성을 나타냄.

![](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0503.png)

이를 해결하기 위해 *read-after-write consistency*(혹은 *read-your-writes consistency*)가 필요함. 사용자가 스스로가 만든 변경 사항은 항상 최신으로 보여주는 것. 구현하는 방법은 몇 가지가 있다.

- 자신이 수정할 수 있는 대상은 리더로부터 불러들임. 예컨대, A라는 사용자가 접속한 경우, A의 프로필은 리더로부터 가져오고, 그 외의 것은 팔로워로부터 가져옴.
- 하지만, 데이터가 여러 사용자에 의해 수정될 수 있다면 위 접근법은 비효과적. 대부분 리더로부터 가져오게 될 테니까. 대신, 마지막 갱신 시간을 추적해서, 갱신된 지 1분 이내라면 리더로부터 데이터를 가져올 수도 있음.
- 마지막으로 갱신이 일어난 타임스탬프까지 팔로워들이 따라잡도록 할 수도 있음. 만약, 어떤 레플리카가 충분히 최신이 아니라면, 다른 레플리카로 리드 요청을 처리하거나, 레플리카가 충분히 따라잡을 때까지 요청을 대기. 참고로, 이때의 타임스탬프는 논리적 타임스탬프<sup>logical timestamp</sup>(쓰기 순서 등)일 수 있음.

데이터센터가 지역적으로 분산되어 있는 경우에는 이 복잡성이 더 커짐. 또한, 같은 사용자가 여러 디바이스를 통해 접근하는 경우에도 복잡성은 높아진다.

### Monotonic Reads

같은 리드 요청이 여러 팔로워들에 분산될 때도 비일관성 문제가 발생함.

![](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0504.png)

Monotonic reads는 한 가지 해결책. 최신 데이터를 보지는 못할 수도 있지만, 한 번 읽어 들인 시점보다 과거의 데이터를 제공하지는 않는 것. 강한 일관성<sup>strong consistency</sup> 보다는 약하고, 결과적 일관성<sup>eventual consistency</sup> 보다는 강한 보장.

이를 달성하는 한 가지 방법은 각 사용자는 항상 같은 레플리카를 보게 하는 것. 예컨대, 사용자 ID의 해시(랜돔이 아니고)에 기반하여 레플리카를 선택하게 함. 그리고 만약 대상 레플리카가 장애가 났다면, 다른 레플리카로 재 라우팅.

### Consistent Prefix Reads

A, B가 순서대로 발생했지만, 레플리케이션 지연에 의해 B, A 순서로 이벤트가 발생한 것 처럼 보이는 현상. 이 문제는 파티션 된(혹은 샤드가 된) 데이터베이스의 특수한 문제라고 함. 쓰기 순서를 전역적으로 보장할 수 있는 방법이 없기 때문.

![](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0505.png)

Consistent prefix reads는 한 가지 해결책. 쓰기가 발생한 순서대로 데이터를 읽게 하는 것. 이를 위해, 관련 있는 데이터는 같은 파티션에 둘 수도 있음. 하지만 일부 애플리케이션에서는 이를 효율적으로 수행하기가 어려움. 이런 인과적 의존성을 추적할 수 있는 알고리즘도 있음. 뒤에서 다룰 예정.

### Solutions for Replication Lag

- "레플리케이션 지연이 증가되면 애플리케이션에 문제가 있을까?"를 먼저 질문.
- 아무런 문제도 없다고 할 수도 있음. 그러면 끝.
- 문제가 있다면, 위에서 살펴 본 좀 더 강한 보장을 시스템에서 제공.
- 하지만, 이는 애플리케이션 코드의 복잡성을 높이고 잘못되기도 쉽다.
- 그런 면에서는 개발자가 이 문제에 신경을 안 쓰게 하는 것이 가장 좋아 보임.
- 단일 노드에서 트랜잭션이 존재하는 이유이기도 함.
- 하지만, 분산된 시스템에서는 이러한 보장이 되지 않음. 성능과 가용성 측면에서 너무 비싸다며 오히려 포기한 상태.
- 대신, 결과적 일관성이 확장 가능한 시스템에서는 필수 불가결이라고 주장.
- 어느 정도는 사실이지만, 책의 나머지 부분에 대해서는 조금씩 다른 관점들을 다룰 예정.

## Multi-Leader Replication

지금까지 리더 기반 레플리케이션 모델에 대해서 알아봄. 이번에는 다중 리더<sup>multi-leader</sup>(master-master 또는 active/active) 레플리케이션에 대해서 살펴볼 예정. 리더가 하나이면 이 리더가 문제가 있을 경우 쓰기가 불가. 리더가 복수이면 이 문제를 어느 정도 극복. 이 리더들은 기본적으로 리더처럼 동작하지만, 다른 리더에 대해서는 팔로워처럼 동작함.

### Use Cases for Multi-Leader Replication

하나의 데이터센터에서 다중 리더 설정을 하는 것을 일반적으로 비합리적. 복잡성이 너무 높기 때문. 하지만, 몇 가지 이득이 되는 경우가 있음.

#### Multi-Datacenter Operation

각 데이터 센터에 한 개의 리더를 두는 것. 그림은 [여기](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0506.png) 참고. 리더에 사용자의 쓰기 요청이 들어오면, 이를 우선 자신의 데이터 센터에 있는 레플리카로 보내고, 동시에 다른 데이터 센터에 있는 *confilict resoultion*에게도 전달함. 충돌 문제가 없다면 그 데이터 센터의 리더에게 전달.

아래는 데이터 센터가 여러 개일 때 단일 리더와 복수 리더 설정의 차이.

*Performance*

- 읽기의 차이는 없을 것.
- 단일 리더 − A 데이터 센터에 가까운 쓰기 요청이, 리더가 있는 B 데이터 센터에 먼저 전달되고, 다시 A 데이터 센터로 레플리케이션 되는 시간적 비용은 상당함.
- 복수 리더 − 쓰기 요청은 각 지역의 데이터 센터로 보내지므로, 단일 리더에 비해 성능은 좋음.

*Tolerance of datacenter outages*

- 단일 리더 − 리더가 있던 데이터 센터에 장애가 나면, 다른 데이터 센터의 한 팔로워를 리더로 승격시켜야 함.
- 복수 리더 − 장애가 나지 않은 데이터 센터는 계속 운영이 가능. 물론, 데이터 센터가 정상화 되면, 레플리케이션을 따라 잡아야 함.

*Tolerance of network problem*

- 데이터 센터 간의 네트워크는 주로 퍼블릭. 이는 로컬 네트워크에 비해 낮은 신뢰성을 가짐.
- 단일 리더 − 쓰기 요청이 이 퍼블릭 링크를 통해 동기적으로 이뤄지므로 신뢰성이 낮을 수 있음.
- 복수 리더 − 비동기적으로 레플리케이션이 이뤄짐. 일시적 네트워크 문제는 견뎌낼 수 있음.

일부 데이터베이스는 복수 리더 설정을 기본으로 제공함. 또는 MySQL에는 Tungsten, PosgtreSQL에는 BDR, Oracle에는 GoldenGate 등 외부 도구들도 존재.

#### Clients With Offline Operation

애플리케이션이 인터넷으로부터 연결이 끊겼음에도 동작해야 할 필요가 있을 때, 복수 리더 레플리케이션을 사용할 수 있다고 함. 하지만, 중간의 내용을 보면, "데이터센터 간의 네트워크 연결이 신뢰하기 어려운 상황에서"라고 되어 있음. 개인적으로는 이 표현이 올바르다고 생각함.

일정 시간 네트워크가 동작하지 않을 수 있고, 이로 인해 얼마 간의 데이터를 동기화해야 하는 작업이 필요한데, 이 작업이 만만한 것은 아님. 이런 설정을 쉽게 할 수 있도록 도와주는 도구들이 있음. 예컨대, CouchDB는 이런 운영 모드를 위해 설계되었다고 함.

#### Collaborative Editing

복수의 사용자가 동시에 하나의 문서를 편집하는 것과 데이터베이스 레플리케이션은 비슷한 점이 많음. 오프라인 편집 사용 사례도 같은 맥락. 큰 단위(e.g 문서)의 락 대신, 작은 단위(e.g 단일 키스트로크)를 유지하고, 충돌 해결 전략을 함께 제공.

### Handling Writes Conflicts

복수 리더 레플리케이션의 최대 문제. 쓰기 충돌. 표로 나타내면 다음과 같음.

| A 데이터센터 마스터                              | B 데이터센터 마스터                          |
| ------------------------------------------------ | -------------------------------------------- |
| `insert into user (id, name) values (1, 'foo');` |                                              |
|                                                  | A로부터 레코드 `1`이 레플리케이션 됨         |
|                                                  | `update user set name = 'bar' where id = 1;` |
| `update user set name = 'baz' where id = 1;`     |                                              |
| B로부터 레코드 `1`이 레플리케이션 된다면?        |                                              |

*"Q. B로부터 레코드 `1`이 레플리케이션 된다면, `name`은 `bar`인가 `baz`인가?"*

단일 리더 데이터베이스에서는, 첫 번째 쓰기가 완료될 때 까지, 두 번째 작성자를 대기시킬 수 있음. 혹은, 두 번째 작성자에게 재시도를 안내하면서 진행중이던 트랜잭션을 종료시킬 수도. 반면, 복수 리더 설정에서는, 두 개의 쓰기가 모두 성공함. 충돌은 일정 시간이 지난 뒤 비동기적으로 감지될 뿐. 사용자에게 충돌 해결을 요구하기에는 너무 늦어버렸을 수도.

#### Synchronous Versus Asynchronous Conflict Detection

특정 리더에 쓰기 요청이 들어오면, 다른 데이터센터의 리더에 레플리케이션 될 때까지 쓰기 완료를 대기시킬 수 있음. 그러나 위에서 이야기한 것처럼, 네트워크 상황에 따라 쓰기의 신뢰성이 낮아질 수 있음. 책에서는 권장하지 않는 방식.

#### Conflict Avoidance

아예 충돌을 회피하는 방법을 취할 수도 있음. 예를 들어, 레코드 별로 쓰기가 가능한 리더를 할당하고, 같은 리더만 접근을 허용. 복수 리더 레플리케이션의 충돌을 다루는 많은 구현들이 부실하기 때문에 자주 사용되는 접근 방식.

하지만, 레코드에 할당된 리더를 변경해야 할 수도. 특정 데이터 센터에 장애가 발생했거나, 사용자가 다른 지역으로 이동하는 바람에 지역적으로 가까운 데이터 센터가 바뀌었다던가 하는 등의 이유로 말이다. 충돌 회피가 깨질 수도 있는 것. 하지만 개인적으로는 충돌 회피가 왜 깨지는지 이해X. 라우팅 설정을 바꾸는 사이에 쓰기 요청이 들어오는 경우를 말하는 건가?

#### Converging Toward a Consistent State

단일 리더 데이터베이스에서는 쓰기가 순서대로 이뤄짐. 같은 필드에 여러 번의 갱신이 있다면, 마지막 쓰기가 그 필드의 마지막 값임. 하지만, 복수 리더 설정에서는 쓰기의 순서가 정의되지 않음. A 데이터 센터에는 1이라는 값이, B 데이터 센터에는 2라는 값이 있는 상태에서, 어떻게 결과적으로 모든 레플리카가 같은 값을 가지도록 보장할 수 있을까?

수렴 충돌 해결<sup>convergent conflict resolution</sup>을 해결하는 방법에는 여러 가지가 있음.

- 각 쓰기에 고유 ID(e.g 타임스탬프, 긴 랜덤 숫자, UUID, 키와 값의 해시)를 부여. 가장 높은 ID를 가진 쓰기가 승자. 나머지는 모두 버림. 타임스탬프가 사용되었다면, 이런 기법을 가리켜 LWW(Last Write Wins)라고 함. 이 방법이 널리 퍼져 있긴 하지만, 데이터 유실이 발생할 수 있음.
- 각 레플리카에 고유 ID를 부여. 그리고 쓰기를 수행할 레플리카를 선택할 때, 가장 높은 숫자가 부여된 노드를 선택. 이 또한 데이터 유실 가능성 내포.
- 어떻게든 값을 병합. 예컨대, 1과 3이라는 값이 각 리더에 있다면, 1/3이라는 값으로 만듦.
- 명시적인 데이터 구조에 충돌을 기록하고, 어플리케이션에서 이를 해결하도록 함. 사용자에게 충돌 해결을 위임할 수도.

#### Custom Conflict Resolution Logic

올바른 충돌 해결은 애플리케이션에 따라 다르므로, 대부분의 복수 리더 레플리케이션 도구들은 애플리케이션 코드를 통해 충돌 해결 로직을 작성할 수 있게 함. 이 코드는 쓰기 또는 읽기 시점에 수행.

##### 쓰기 시점

- 레플리케이션 된 변경 로그에서 충돌이 감지되는 즉시 충돌 핸들러 호출.
- 백그라운드 프로세스로 빠르게 처리됨.
- 따라서, 사용자에게는 안내 되지 않는 것이 일반적.

##### 읽기 시점

- 충돌 나는 쓰기라고 해도 일단 저장.
- 이 충돌 데이터에 대한 읽기가 발생하면, 데이터의 여러 버전을 애플리케이션에 반환.
- 애플리케이션에서는 이를 직접 해결하고 그 결과를 데이터베이스로 보내줄 수도 있고,
- 충돌을 사용자에게 안내하여 해결을 유도할 수도 있음.

참고로, 충돌 해결의 단위는 전체 트랜잭션이 아니라, 개별 로우 또는 도큐먼트에 대해 이뤄지는 것이 일반적.

### Multi-Leader Replication Topologies

레플리케이션 토폴로지<sup>replication topologiy</sup>는 노드 간에 쓰기 레플리케이션이 이뤄지는 커뮤니케이션 경로를 표현. circular topology, start topology, all-to-all topology가 그 예. [여기 그림](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0508.png) 함께 참고.

- 가장 일반적인 토폴로지는 all-to-all.
- MySQL은 circular 만을 기본으로 지원.
- circular 그리고 star 토폴로지는 쓰기가 몇 개의 노드를 거쳐가기도. 이 때 발생할 수 있는 무한 루프를 방지하기 위해, 각 노드는 고유 식별자를 가지고 있으며, 이 식별자를 각 쓰기에 태깅함.
- circular 그리고 star 토폴로지는 한 노드에 장애가 생기면, 경로가 끊어지게 되고, 따라서 재설정 되어야 함.
- All-to-all에서도 문제가 있음. 바로 "추월<sup>overtake</sup>". [그림](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0509.png) 참고.
- 이런 순서 이슈를 해결하기 위해 버전 벡터<sup>version vectors</sup> 같은 것이 사용되기도 함(뒤이어 다뤄지는 내용). 하지만, 충돌 감지 기술은 많은 시스템에서 부실하게 구현되어 있다는 것에 주의.

## Leaderless Replication

'Sloppy Quorums and Hinted Handoff'에서 리더 없는 레플리케이션을 잘 정리하는 문단이 있어서 함께 기록.

> Databases with appropriately configured quorums can tolerate the failure of indivisual nodes without the need for failover. They can also tolerate individual nodes going slow, because requests don't have to wait for all n nodes to respond―they can return when w or r nodes have responded. These characteristics make databases with leaderless replication appealing for use cases that require high availability and low latency, and that can tolerate occasional stale reads.

관계형 데이터베이스 시대 동안 리더 없는 레플리케이션은 잊혀짐. 그러다가 Amazon이 인하우스 Dynamo 시스템에 이 모델을 사용하면서 점점 다른 곳에서도 사용. Riak, Cassandra, Voldemort가 그 예. 이 때문에 이 모델을 가리켜 Dynamo-style이라고도 부름.

이 모델에서는 클라이언트가 특정 노드가 아닌 임의의 레플리카로 쓰기 요청을 보냄. 또는, 코디네이터 노드가 이 역할을 대신 수행하거나.

### Writing to the Database When a node Is Down

리더 없는 레플리케이션은 노드의 장애복구<sup>failover</sup>가 필요 없음. 대신, 아래와 같은 방식으로 쓰기와 읽기 요청을 처리함. [여기 그림](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0510.png) 함께 참고.

- 클라이언트는 쓰기 요청을 모든 레플리카에게 병렬로 보냄.
- 이 중 한 노드가 업데이트로 인해 중단 되었다고 가정.
- 그러면, 클라이언트는 나머지 정상 노드로부터만 응답을 받음.
- 그러다가 중단 되었던 노드가 다시 클러스터로 복귀.
- 클라이언트는 읽기를 위해 모든 레플리카에게 병렬로 요청을 보냄.
- 중단 되었던 노드로부터는 오래된<sup>stale</sup> 데이터를 받을 수 있음.
- 클라이언트는 버전 번호를 통해 어떤 값이 최신인지를 스스로 판단.

#### Read Repair And Anti-Entropy

문제가 되던 노드가 정상화 되면, 이 노드는 최신 쓰기를 어떻게 따라잡을까? read repiar 그리고 anti-entropy process 이렇게 두 가지 방법이 주로 사용된다고 함.

먼저, read repair.

- 클라이언트가 읽기 요청을 여러 노드에 동시에 날렸다면,
- 최신 데이터와 오래된 데이터를 감지할 수 있으며,
- 오래된 데이터를 반환한 노드에게 최신 데이터를 보내 쓰기 요청.
- 자주 읽히는 값일 때 잘 동작한다고 함.

다음으로, anti-entropy process.

- 백그라운드 프로세스가 있고,
- 레플리카 간의 데이터 차이를 찾아다니다가,
- 누락된 데이터가 있으면 이를 복제함.
- 리더 기반 레플리케이션에서의 레플리케이션 로그와 다르게,
- 쓰기를 순서대로 복제하지 않음.
- 따라서, 데이터가 복제가 심각할 정도로 지연될 수도 있음.

위에서 언급했던, 코디네이터 노드가 없다면 read repair 방식은 애플리케이션 입장에서는 너무 부담. anti-entropy process 방식이 더 낫지 않을까 함. 데이터 복제가 심각할 정도로 지연된다는 것은 무엇일까. 레플리카가 많을 수록 데이터 복제의 지연은 문제가 될 가능성은 낮아 보임. 하지만, 데이터 크기가 많아질 수록 문제가 될 수도 있을 것 같기도 하고. 그렇다고, 이를 보완하는 방법이 아예 없을 것 같지도 않고. 여러모로 추가 확인이 필요해 보임.

#### Quorums for Reading And Writing

- 전체 노드의 갯수가 n,
- 쓰기가 가능한 노드의 갯수를 w,
- 읽기가 가능한 노드의 갯수를 r이라고 할 때,
- w + r > n을 만족시켜야 함.
- 일반적으로 n은 홀수로 두며,
- w = r = (n + 1) / 2 (반올림)으로 설정.
- [여기 그림](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0511.png)처럼, 적어도 하나의 노드로부터 최신 데이터를 읽어 들일 수 있기 때문.
- 여기서 중요한 건, w와 r은 얼마나 많은 노드로부터 응답을 기다려야 하는지를 결정하는 값이라는 것.

### Limitations of Quorum Consistency

좀 더 유연한 설정도 가능함.

- w + r ≦ n 값을 설정할 수도 있음.
- 오래된 값을 읽어 들일 가능성은 높아짐.
- 하지만, 응답 지연이 낮고 가용성은 높아짐.

참고로, w + r > n에서도 오래된 값이 나올 가능성이 있음. 책에서는 몇 가지 시나리오를 언급하지만, 잘 이해되지 않아 기록하지는 않음. 추후 다시 살펴볼 것. 어쨌든 실제로는 쿼럼 수가 항상 최신 값 읽기를 보장하지는 못한다는 것. Dynamo 스타일 데이터베이스는 일반적으로 결과적 일관성에 최적화 되어 있음.

#### Monitoring Staleness

- 데이터베이스가 최신 값을 반환하는지 모니터링 하는 것은 중요.
- 리더 기반 레플리케이션에서는 레플리케이션 지연 메트릭을 제공하는 것이 쉬움. 쓰기가 팔로워들에게 같은 순서로 적용되기 때문.
- 하지만, 리더 없는 레플리케이션에서는 쓰기가 적용되는 순서가 유동적. 만약, read repair만 사용한다면 지연을 추적하는 것은 더더욱 어려움.
- 리더 없는 레플리케이션에서의 이런 문제를 해결하기 위한 연구들이 진행중. 그러나 실제로 잘 쓰이지는 않는 상태.

### Sloppy Quorums and Hinted Handoff

1. w 또는 r 쿼럼을 만족시키지 못한다면, 에러를 반환하는 게 좋을까?
2. 아니면, n개 이외의 별도의 노드를 두고, 이 별도의 노드까지 포함하여 w를 만족시킨다면, 이를 허용해 주는 게 좋을까?

후자를 가리켜 *sloppy quorum*이라고 함. 굳이 번역하면, 날림 정족수? 그리고 이렇게 임시로 한 노드가 받은 쓰기를 다시 "집(n개로 지정됐었던)" 노드로 보내는 것을 가리켜, *hinted handoff*라고 부름.

sloppy quorums는 쓰기 가용성을 높일 때 유용함. 다만, w + r > n이 만족하더라도 최신 값을 읽어들이지 못할 수도 있음. 바깥 노드에 최신 데이터가 있을 수도 있기 때문. 따라서, sloppy quorums의 quroum은 전통적인 의미라기 보다, 내구성을 보장해주는 정도의 것.

모든 Dynamo 구현에서는 이것이 선택적임. Riak에서는 기본으로 활성화 되어 있고, Cassandra와 Voldemort에서는 기본으로 비활성화.

#### Multi-Datacenter Operation

리더 없는 레플리케이션은 복수 데이터 센터 운영에 적합함. 동시 쓰기의 충돌, 네트워크 단절, 응답 지연 스파이크 등에 내성을 갖도록 설계되었기 때문.

### Detecting Concurrent Writes

Dynamo 스타일 데이터베이스는 같은 키에 대한 여러 클라이언트의 동시 쓰기를 허용. 따라서, 강한 정족수<sup>quorum</sup>가 사용된다고 해도, 충돌은 발생할 수 있음. 다중 리더 레플리케이션에서도 이는 마찬가지. 다만, 차이점은 Dynamo 스타일에서는 read repair 또는 hinted handoff에서 발생. "[Concurrent writes in a Dynamo-style datastore: there is no well-defined ordering](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0512.png)"에서 이 문제를 잘 표현하고 있음. "Handling Write Conflicts"에서 한 번 다루긴 했지만, 이번에 좀 더 자세히 설명.

#### Last Write Wins (Discarding Concurrent Writes)

PostgreSQL 다중 리더 레플리케이션 사용 시 선택할 수 있는 옵션이었던 것으로 기억함.

- 레플리카들은 오직 최신 값 만을 저장.
- 최신 값을 결정하는 기준으로 타임스탬프(쓰기 시점에 부여) 등을 활용.
- 이런 충돌 해결 알고리즘을 가리켜 LWW(Last Write Wins)라고 부름.
- Cassandra에서는 이 방법만을 유일하게 지원함. Riak에서는 선택적 피처.
- 이 방식의 문제점은 내구성<sup>durability</sup>. 즉, 같은 키에 대한 여러 쓰기가 동시에 발생하면, 심지어 어떤 경우에는 동시가 아니더라도, 클라이언트에게는 성공했다고 안내되고, 실제로는 오직 하나의 최신 값만이 살아남음.
- 만약, LWW를 쓰면서 데이터 유실까지 방지하고 싶다면, 오직 한 번만 쓰고, 그 값은 불변으로 유지. 즉, 같은 키에 대한 동시 쓰기를 피하는 것.

#### The "Happens-Before" Relationship and Concurrency

갑자기 동시성<sup>concurrency</sup>에 대한 정의를 내리고 있음. 어쨌든, 시간이 아닌, 의존성을 기준으로 판단한다는 것.

> For defining concurrency, exact time doesn't matter: we simply call two operations concurrent if they are both unaware of each other, regardless of the physical time at which they occurred.

#### Capturing the Happens-Before Relationship

"[Capturing causal dependencies between two clients concurrently editing a shopping cart](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0513.png)"는 두 클라이언트가 cart라는 키에 아이템 목록을 동시에 저장하는 과정을 보여줌. 그림과 같은 방식을 이용하면, 값을 읽어들이지 않고도, 버전 번호만을 통해 동시 작업이었는지 여부를 판별할 수 있음. 읽기 시 반환된 버전의 값들을 병합하고, 이전 쓰기에서 응답 받은 버전 번호와 함께 쓰기를 요청. 서버는 해당 버전보다 위 번호를 가진 값들은 유지하고, 버전에 해당하는 값만을 갱신하고 새로운 버전을 부여. 감지하는 것 뿐만 아니라, 병합하는 것 까지 설명하고 있음에 유의.

#### Merging Concurrently Written Values

어떤 데이터도 조용히 사라지는 것을 방지하는 접근법. 하지만, 클라이언트가 충돌 데이터를 직접 다루는 부담이 발생함.

복수 리더 쓰기에서 다뤘던 것 처럼, 버전 번호나 타임스탬프를 이용할 수도 있음. 하지만, 이는 데이터 유실 가능성을 내재. 따라서, 쇼핑 카트 예제처럼, 값을 병합하는 방식으로 접근. 단, 위 예제와 다르게, 사람들은 아이템을 제거할 수도 있는데, 이 때는 단순 병합으로는 어렵고, 삭제된 아이템에 대해 DB에서는 마커를 남김. 버전 번호와 함께. 그리고 병합에 활용. 이런 마커를 *tombstone*이라고 부른다고 함.

참고로, Riak에서는 이 동시 값들을 가리켜 *siblings*라고 부름. 또한, CRDT라고 부르는 데이터 구조체를 제공하고, 자동으로 *siblings*을 병합시켜줌. 아무래도 애플리케이션에서 이런 일들을 직접 하는 것이 좋은지는 의문.

#### Version Vectors

"[Capturing causal dependencies between two clients concurrently editing a shopping cart](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0513.png)" 예시는 레플리카가 하나만 있을 때를 다룸. 하지만, 여러 대의 레플리카가 있을 수 있음(물론, 리더 없이). 이럴 경우에는 키 별로, 그리고 *레플리카 별로* 버전 번호를 사용. 이 번호는 당연히 어떤 값을 덮어 써야 하고, 어떤 값을 *sibling*으로 유지하는지 판별하기 위해 사용됨.

# Partitioning

레플리케이션은 지역적으로 가까운 데이터의 제공, 장애 내성, 읽기 성능 향상을 제공. 하지만, 매우 큰 데이터 셋을 가지고 있거나, 쿼리 처리량이 매우 높아야 하는 경우에는 레플리케이션 만으로는 어려움. 따라서, 데이터를 여러 파티션들로 나눌 필요가 있음. 샤딩이라고도 알려짐.

책에서 주로 다룰 내용은, 각각의 파티셔닝 접근법, 파티셔닝에서의 데이터 인덱싱, 리밸런싱, 그리고 요청 라우팅.

## Partitioning and Replication

"[Combining replication and partitioning: each node acts as leader for some partitions and follower for other partitions](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0601.png)"에 나와 있는 것과 같이, 레플리케이션과 파티셔닝을 조합할 수 있다는 얘기. 레플리케이션과 파티셔닝은 서로 거의 독립적이라서, 조합의 방법은 다양함.

## Partitioning of Key-Value Data

데이터를 파티션 하기로 했다면, 레코드를 저장할 노드를 어떤 기준으로 선택해야 할까?

파티셔닝의 목적은 데이터와 쿼리 부하를 여러 노드에 고르게 분산시키는 것. 따라서, 잘 분산되어 있다면,  분산된 노드의 수 만큼, 많은 양의 데이터를 다룰 수 있고, 읽기와 쓰기 처리량도 역시 늘어남. 반대로, 잘 분산되어 있지 않다면 파티셔닝의 효과는 줄어듦. 이렇게 잘 분산되어 있지 않은 경우를 가리켜 *skewed*, 불균형으로 인해 부하가 집중된 파티션을 가리켜 *hot spot*이라고 부름.

hot spot을 피하는 가장 단순한 방법은 레코드가 저장될 노드를 무작위로 고르는 것. 하지만, 이렇게 되면 나중에 레코드를 읽으려 할 때, 어디에 있는지 알 수 없게 되고(매핑 키를 두면 되지 않나?), 따라서 여러 노드에 동시에 질의를 던져야 함.

### Partitioning by Key Range

[A print encyclopedia is partitioned by key range](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0602.png)에서 보는 것처럼, 키를 여러 개의 구간으로 나누고, 구간별로 노드를 지정한 뒤, 값을 저장하는 것. 나중에 값을 찾을 때도, 키가 어느 구간에 속하는지만 알면, 해당 노드에 바로 질의할 수 있음. 예컨대, 타임스탬프를 기준으로 데이터가 각 노드에 분산되어 있고, 특정 월의 데이터가 필요하면 해당 노드에 바로 질의.

하지만, 데이터가 고르게 분배되지 않을 수도 있음. 예를 들어, 2017년 7월에만 데이터가 엄청 몰려 있을 수도. 즉, hot spot. 이런 문제를 피하기 위해, 파티션 경계는 데이터의 상황을 잘 고려해야 함. 따라서, 타임스탬프 대신, 노드 분배 기준을 기기 이름과 타임스탬프의 조합으로 할 수도 있음. 기기 이름 별로 데이터가 잘 분산된다는 가정하에 말이다. 참고로, 파티션 경계는 어드민이 직접 수동으로 결정하거나, 데이터베이스가 자동으로 선택하게 할 수도 있음. Bigtable, RethinkDB, MongoDB 등에서 이런 파티셔닝 전략을 사용한다고 함.

### Partitioning by Hash of Key

이런 skew와 hot spot 때문에, 많은 분산 데이터베이스들은 키의 파티션 결정에 해시 함수를 사용. [Partitioning by hash of key](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0603.png) 그림 참고.

해시 함수는 암호 수준이 높을 필요는 없음. MongoDB에서는 MD5를, Cassandra는 Murmur3를, Vodemort는 Fowler-Noll-Vo 함수를 사용. Java의 `Object.hashCode()`나 Ruby의 `Object#hash` 등 프로그래밍 언어에서 기본으로 제공하는 해시 함수는 적절치 않을 수도. [Java's hashCode is not safe for distributed system](https://martin.kleppmann.com/2012/06/18/java-hashcode-unsafe-for-distributed-systems.html)에서 보듯 같은 키지만, 해싱의 결과가 달라지는 경우가 있기 때문.

하지만, 범위 검색이 어려움. 데이터들이 순차적으로 분산되어 있지 않기 때문. 이로 인해, MongoDB에서는 범위 검색 요청을 모든 파티션에 보냄. Riak, Couchbase, Voldemort에서는 기본 키에 대한 범위 검색을 아예 지원하지 않음. Cassandra는 테이블에 대해 복수 개의 컬럼으로 구성된 *compound primary key*를 선언할 수 있게 함. 이 키의 첫 번째 요소는 파티션을 결정하는 데 쓰이고, 나머지 컬럼들은 Cassandra의 SSTable 데이터를 정렬하기 위한 연결 인덱스로 사용. 따라서, 첫 번째 컬럼 값이 정해져 있기만 하면, 나머지 컬럼을 이용한 범위 검색이 가능.

### Skewed Workloads and Relieving Hot Spots

키를 해싱하는 것 만으로는도 균형 잡기가 어려울 수 있음. 만약, 사용자의 아이디를 해싱해서 파티셔닝 하고, 특정 파티션의 특정 사용자의 데이터만 많아진다면, 이는 결국 불균형.

그리고 이런 불균형<sup>skew</sup>을 피하는 것은 애플리케이션의 책임(데이터 시스템이 해주지 않음). 예를 들어, 파티션 키의 뒷 부분에 랜돔 숫자를 붙여서 다른 곳으로 파티셔닝 되게 할 수도 있음. 하지만, 데이터 읽는 것이 어려워짐. 랜돔 숫자의 자릿수가 2자리였다면, 데이터 읽기 시 원래의 키에 100개의 숫자를 붙여 100개의 키를 만들고, 이 키에 대한 결과를 모두 조회해서 병합해야 함. 따라서, 주요 키(hot spot 대상)에 대해서 별도의 관리를 하고, 이 키에 대한 조회가 발생했을 때만 추가적인 작업을 수행하게 해야 함. 데이터가 많지 않은 대부분의 키에 대해서 이런 추가 작업을 하는 것은 불필요한 자원 낭비이므로.

## Partitioning and Secondary Indexes

보조 인덱스가 파티셔닝에 함께 개입되면 상황은 좀 더 복잡해짐. 보조 인덱스는 레코드를 유일하게 식별할 수 없고, 따라서 기본키 만큼의 명확한 파티션 구분이 어렵기 때문. 보조 인덱스로 파티셔닝을 하는 방법은 2가지. 문서 기반<sup>document-based</sup>, 그리고 용어 기반<sup>term-based</sup> 파티셔닝.

### Partitioning Secondary Indexes by Document

문서 기반 방식의 원리는 간단. "[Partitioning secondary indexes by document](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0604.png)" 그림을 참고. 간단히 설명하면, 파티션 별로 보조 인덱스를 독립적으로 관리하는 것. 이런 이유로 로컬 인덱스(글로벌 인덱스와 대조적)라고 불리기도 함.

주의할 것은, 보조 인덱스들이 파티션 마다 독립적으로 존재하기에, 모든 파티션에게 질의를 던지고 또 결과를 병합해야 한다는 것. 이런 방식을 *scatter/gatter*라고 부른다고 함. 당연하게도, 부담이 큼. 지연시간 증폭<sup>latency amplification</sup>이 나타날 수도 있고. 그럼에도 불구하고 MongoDB, Riak, Cassandra, Elasticsearch, SolrCloud, VoltDB 등에서 사용중. 그래도, 이 벤더들은 가능한 단일 파티션에서 보조 인덱스 질의가 수행될 수 있도록 파티션을 구성하라고 권장.

### Partitioning Secondary Indexes by Term

각 파티션에 보조 인덱스를 독립적으로 구축하는 대신, 모든 파티션에 대한 글로벌 인덱스를 생성할 수 있음(개인적으로는 글로벌이라는 용어가 좋아 보이지는 않음). "[Partitioning secondary indexes by term](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0605.png)" 그림을 보면 이해하기 쉬움. 보조 인덱스를 일정 범위로 나누고 각 파티션에 위치시키는 것. 여기서 보조 인덱스를 그대로 저장할 수도 있고(범위 검색에 유리), 해싱해서 저장할 수도 있음(고르게 분산). 참고로, *term*이라는 용어는 *full-text* 인덱스로부터 나왔다고 함. term은 문서에 존재하는 단어를 가리킴.

글로벌 인덱스의 장점은 읽기가 좀 더 효율적으로 된다는 것. 모든 파티션에 대해 매번 질의를 요청할 필요가 없기 때문. 단점은 글로벌 인덱스의 쓰기가 다소 느려지고 복잡해진다는 것. 인덱스가 항상 최신이려면, 이런 글로벌 인덱스 처리 방식으로 인해, 모든 파티션에 대한 분산 트랜잭션이 필요. 하지만, 데이터베이스들은 이를 지원하지 않고, 따라서 보조 인덱스는 종종 비동기로 갱신됨. 다소 지연이 발생할 수도 있음.

## Rebalancing Partitions

리밸런싱이 단순히 데이터를 이동시키는 것인 줄 알았는데, 책에서는 아래와 같이 정의함. 부하를 이동시키는 것이 꼭 데이터 이동을 의미하는 건 아닐 수도 있을 듯 한데 아무튼 궁금.

> All ot these changes call for data and requests to be moved from one node to another. THe process of moving load from one node in the cluster to another is called *rebalancing*.

그리고 이런 리밸런싱은 적어도 아래의 요구사항을 만족시켜야 한다고 함.

- 리밸런싱이 되면, 클러스터의 각 노드에 부하<sup>load</sup>가 고르게 분배되어야 함.
- 리밸런싱이 일어나는 동안 데이터베이스는 읽기와 쓰기 요청을 계속 받을 수 있어야 함.
- 필요한 만큼의 데이터만을 노드 간에 이동시켜야 함. 리밸런싱을 빠르게 끝내고, 네트워크와 디스크 I/O를 최소화하기 위해서.

#### Strategies for Rebalancing

##### How Not to Do It: Hash Mod N

앞선 파티셔닝에서, 해시 값을 범위 별로 나눴었음. 그런데, 왜 해시 값을 `mod` 연산하여 파티션을 지정하지 않았을까? 간단한 방법 아닌가? 노드의 갯수가 변경되면, 대부분의 데이터들이 리밸런싱 되기 때문. 필요한 만큼만 데이터를 이동시키는 것이 좋음.

##### Fixed Number of Partitions

"[Adding a new node to a databse cluster with multiple partitions per node](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0606.png)" 그림에서 보듯, 노드 별로 여러 개의 파티션을 미리 할당해 두고, 새로운 노드가 추가 되면, 각 노드에서 일부 파티션만을 새로운 노드로 재할당. 모든 데이터가 아닌, 노드 별로 일부 데이터만 이동하면 됨. 노드가 제거될 때는 반대로 진행.

이 방식에서는 파티션의 갯수도 변경되지 않고(그래서 제목이 Fixed Number of Patitions), 파티션에 대한 키 할당도 변경되지 않음. 오직, 노드에 할당되는 파티션만 바뀜. 파티션 분할이 없으면, 운영적인 측면에서 용이함.

파티션의 갯수를 최대한 크게 잡는 것이 좋아 보일지 모르지만, 각 파티션의 관리는 부담으로 다가옴. 리밸런싱과 노드 장애 복구 비용이 커진다고 함. 그렇다고 너무 작게 유지하는 것은 역시 또 부담(이유는 언급되지 않았지만, 고르게 분할될 가능성도 낮아지고, 파티션 분할도 때에 따라서는 해야 하기 때문이 아닐까 추측해 봄).

참고로, 이 방식은 Riak, Elasticsearch, Couchbase, Voldemort에서 사용된다고 함.

##### Dynamic Partitioning

고정된 파티션 수<sup>fixed number of partitions</sup> 방식에 고정된 경계를 부여하는 경우, 특정 파티션에만 데이터가 몰릴 수 있고, 이 경계를 일일이 조정하는 것은 성가신 일임. 이런 이유로, HBase나 RethinkDB 등의 키 범위로 파티션 된 데이터베이스들은 파티션을 동적으로 만듦. 만약 파티션의 크기가 일정 수준 이상으로 넘어가면 2개로 나누고, 반대로 데이터 삭제로 파티션 크기가 일정 임계치 아래로 내려가면 다른 파티션과 병합한다. B-트리의 최상위 레벨에서 일어나는 일과 유사.

이 방식의 장점은 파티션의 수가 전체 데이터 볼륨에 적응적이라는 것. 예컨대, 데이터가 작으면 파티션 수를 작게 유지하므로 불필요한 파티션 관리 오버헤드가 줄어듦. 주의할 점은 빈 데이터베이스는 단일 파티션으로 시작한다는 것. 경계를 나눌 만한 정보가 없기 때문. 이런 문제(이게 왜 문제일까. 오히려 초창기에는 데이터가 적으므로, 파티션이 필요 없고, 따라서 효율적인 것 아닌가)를 완화하기 위해, HBase와 MongoDB는 초기 파티션 수를 지정할 수 있게 해 둠. *pre-splitting*이라고 불리고, 이를 위해서는 키 분배가 어떻게 이뤄질지 미리 어느 정도 예측해야 함.

동적 파티셔닝은 키 범위 파티셔닝 데이터뿐만 아니라, 해시 파티셔닝 데이터에도 잘 들어맞음. MongoDB는 2.4 이후로, 키 범위와 해시 파티셔닝을 모두 지원하고, 두 방식에서 모두 파티션을 동적으로 분할한다고 함.

##### Partitioning Proportionally to Nodes

동적 파티셔닝에서는 파티션의 수가 데이터 셋의 크기에 비례했고, 고정된 파티션 수에서는 각 파티션의 크기가 데이터 셋의 크기에 비례했음. 두 가지 접근법 모두 파티션의 수는 노드 수와는 독립적.

세 번째 옵션은 파티션의 수를 노드의 수에 비례하도록 하는 것. 다시 말해, 노드 별로 파티션의 수를 고정하는 것. Cassandra와 Ketama에서 사용하는 방식. 이 방식에서는, 노드의 수가 변하지 않을 때는 파티션의 크기가 데이터 셋 크기에 비례하여 증가하고, 노드의 수가 늘어나면 파티션의 크기는 다시 작아짐.

새로운 노드가 클러스터에 합류하면, 일정 수의 파티션을 무작위로 고르고, 선택한 각 파티션의 절반만을 자신에게 이동시킴. 무작위로 선택하기는 하지만, 파티셔닝을 여러 번 하기 때문에 고르게 분배됨. 이는 consistent hashing의 원래(?) 정의에 거의 근접함.

#### Operations: Automatic or Manual Rebalancing

리밸런싱은 수동으로 하는 것이 좋을까, 자동으로 하는 것이 좋을까?

완전 자동화 된 방식은 물론 편함. 하지만, 예측하기 어려움. 리밸런싱은 비용이 큰 작업이다. 요청을 라우팅해야 하고, 많은 양의 데이터를 옮겨야 하기 때문. 이는 네트워크의 부담을 가중시키고, 노드가 다른 요청을 처리하는 성능을 저하시킬 수 있음. 게다가, 자동화된 장애 감지와 함께 사용되면, 장애 전파로 이어질 수도 있음. 예를 들어, 한 노드가 부하가 걸렸고, 다른 노드들이 이 노드를 죽은 것으로 간주하게 되고, 자동으로 리밸런싱이 시작되면, 부하 걸린 노드에 더 큰 부하를 주게 되는 것. 네트워크에게도 추가적인 부담이고. 이게 바로 장애 전파<sup>cascading failure</sup>.

이런 이유로, 리밸런싱에는 사람이 개입되는 것이 좋음. 자동화 된 방식에 비해 다소 느릴지 몰라도, 운영상의 리스크를 줄이는 데 도움이 됨.

### Request Routing

각 파티션들은 노드에 분산되어 있는데, 클라이언트는 요청을 보낼 노드를 어떻게 알 수 있을까? 이는 *service discovery*라고 불리는 일반화 된 문제의 한 사례. 데이터베이스에 국한된 문제도 아님.

여기에 대한 몇 가지 접근법이 있음. "[Three different ways of routing a request to the right node](https://www.safaribooksonline.com/library/view/designing-data-intensive-applications/9781491903063/assets/ddia_0607.png)" 그림 함께 참고.

1. 클라이언트는 임의의 노드로 요청을 보냄. 요청을 받은 노드는 자신에게 데이터가 있으면 바로 응답. 그렇지 않다면 요청을 적절한 노드로 포워딩. 포워딩한 노드로부터 응답을 받으면, 그 응답을 클라이언트에게 그대로 반환.
2. 라우팅 계층이 클라이언트의 요청을 받음. 이 계층에서 요청의 분배를 적절히 수행. partition-aware load balancer.
3. 클라이언트가 어느 노드에 파티션이 할당되어 있는지를 인지.

각 접근법의 공통된 주요 문제는, 파티션이 할당된 노드가 바뀐 것을 어떻게 인지할 수 있느냐임.

많은 분산 데이터 시스템들은 ZooKeeper와 같은 별도의 코디네이션 서비스를 사용. Cassandra와 Riak은 조금 다름. 노드 간에 *gossip protocol*이라는 프로토콜을 사용해서, 클러스터 상태의 변경을 감지함. 이 프로토콜은 위에서 언급한 접근법 중에 첫 번째 것과 유사한 방식임. ZooKeeper 등의 외부 서비스로부터 독립적이긴 하지만, 데이터베이스에 복잡성이 가중됨.












