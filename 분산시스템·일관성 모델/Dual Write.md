# Dual Write

## Dual Write란 무엇인가?

Dual Write는 하나의 업무를 처리하기 위해 **서로 다른 두 시스템에 쓰기 작업을 수행하는 것**이다. 예를 들어 거래 마감 정보를 PostgreSQL에 저장하고, 완료 이벤트를 Kafka에 발행하는 경우가 이에 해당한다.

핵심 문제는 두 시스템의 쓰기를 하나의 원자적 커밋으로 묶기 어렵다는 점이다. 두 작업 사이에 프로세스 종료, 네트워크 장애, 타임아웃이 발생하면 한쪽에만 결과가 반영될 수 있다.

이 문서의 직접 발행 예제는 Kafka 트랜잭션을 사용하지 않고, Producer가 `acks=all`로 브로커의 확인 응답을 기다린다고 가정한다. `acks=0`은 브로커의 응답을 기다리지 않으므로, 전송 완료를 같은 의미로 해석하면 안 된다. [Kafka Producer 설정 문서](https://kafka.apache.org/44/configuration/producer-configs/)

## DB를 먼저 저장하면 어떤 문제가 발생할까?

다음 예제에서 `chargingService.complete()`는 자체 DB 트랜잭션을 커밋한 뒤 반환하며, 호출자를 감싸는 외부 트랜잭션은 없다고 가정한다.

```java
public void complete(Long transactionId) {
    chargingService.complete(transactionId);
    kafkaTemplate.send("charging.completed", transactionId.toString());
}
```

1. DB 트랜잭션을 시작하고 거래 마감 정보를 UPDATE한 뒤 COMMIT에 성공한다.
2. Kafka 발행 요청 전에 프로세스가 종료되거나, 발행이 실패한다.
3. 업무 DB에는 마감 기록이 남지만 Kafka에는 완료 이벤트가 없다.
4. 후속 Consumer는 거래가 완료됐다는 사실을 전달받지 못한다.

서버를 재시작하더라도 메모리에 있던 발행 요청이 자동으로 복구되지는 않는다. 재처리하려면 **무엇을 발행해야 하는지 영속적으로 식별할 수 있어야 한다.**

또한 `kafkaTemplate.send()`는 비동기 API다. 호출이 정상 반환됐다는 것과 브로커 ACK를 받았다는 것은 서로 다르다. 반환된 Future나 콜백으로 전송 결과를 확인해야 한다. [Spring Kafka 전송 문서](https://docs.spring.io/spring-kafka/reference/kafka/sending-messages.html)

## Kafka를 먼저 발행하면 해결될까?

DB 작업의 트랜잭션 경계는 앞의 예제와 같고, 이번에는 Kafka 전송 결과를 먼저 확인한다.

```java
public void complete(Long transactionId) throws Exception {
    kafkaTemplate.send("charging.completed", transactionId.toString()).get();
    chargingService.complete(transactionId);
}
```

1. Kafka 발행에 성공하고 브로커 ACK를 확인한다.
2. 이후 DB UPDATE나 COMMIT이 실패한다.
3. Kafka에는 완료 이벤트가 있지만 업무 DB에는 마감 결과가 반영되지 않는다.
4. 후속 Consumer가 확정되지 않은 업무 결과를 바탕으로 후속 처리를 수행할 수 있다.

발행 순서를 바꾸더라도 두 시스템의 결과가 어긋날 수 있다. DB ROLLBACK은 이미 발행된 Kafka 메시지나 Consumer의 업무 결과를 자동으로 취소하지 않는다.

| 처리 순서 | 장애 시점 | 발생 가능한 불일치 |
| --- | --- | --- |
| DB → Kafka | DB COMMIT 후 발행 전 | 업무 데이터만 있고 완료 이벤트는 누락 |
| Kafka → DB | Kafka ACK 후 DB COMMIT 전 | 완료 이벤트만 있고 업무 변경은 미확정 |
| 순서만 변경 | 두 쓰기 사이 | 원자성 문제는 그대로 남음 |

예제의 `.get()`은 전송 결과를 기다리기 위한 표현이다. 실제 구현에서는 대기 시간과 예외 처리 정책을 정해야 하며, 타임아웃을 받았다고 해서 브로커에 기록되지 않았다고 단정해서는 안 된다.

## @Transactional을 사용하면 해결될까?

다음 예제는 DB 트랜잭션 매니저가 적용되고, Kafka 트랜잭션은 별도로 구성하지 않은 경우다.

```java
@Transactional
public void complete(Long transactionId) {
    chargingRepository.complete(transactionId);
    kafkaTemplate.send("charging.completed", transactionId.toString());
}
```

일반적인 Spring DB `@Transactional`은 해당 DB 트랜잭션이 관리하는 작업을 묶는다. **Kafka 쓰기까지 같은 원자적 트랜잭션으로 자동으로 묶어주지는 않는다.**

- DB 트랜잭션 안에서 Kafka 전송을 요청하더라도, Kafka에 먼저 기록된 뒤 DB COMMIT이 실패할 수 있다.
- `send().get()`으로 Producer ACK를 기다려도 DB와 Kafka를 원자적으로 커밋하는 것은 아니다.
- Kafka 트랜잭션을 사용한다면 개별 전송의 ACK와 Kafka 트랜잭션의 COMMIT도 구분해야 한다. `send().get()`의 성공만으로 Kafka 트랜잭션의 커밋까지 완료됐다고 볼 수 없다. [KafkaProducer API 문서](https://kafka.apache.org/44/javadoc/org/apache/kafka/clients/producer/KafkaProducer.html)

Spring Kafka의 DB·Kafka 트랜잭션 동기화를 구성할 수는 있다. 다만 두 트랜잭션은 순차적으로 커밋되므로, 첫 번째 커밋 이후 두 번째 커밋이 실패하는 구간이 남는다. 이를 두 자원의 원자적 커밋을 조정하는 분산 2PC와 구별해야 한다. [Spring Kafka 트랜잭션 문서](https://docs.spring.io/spring-kafka/reference/kafka/transactions.html)

## 그렇다면 afterCommit은?

`@TransactionalEventListener(phase = AFTER_COMMIT)`을 사용하면, 트랜잭션 안에서 발행한 애플리케이션 이벤트의 리스너를 DB COMMIT 이후에 실행할 수 있다. 이를 통해 DB 변경이 롤백됐는데 완료 이벤트가 미리 발행되는 상황을 피할 수 있다. [Spring 트랜잭션 이벤트 문서](https://docs.spring.io/spring-framework/reference/data-access/transaction/event.html)

하지만 다음과 같은 장애 구간은 남는다.

- DB COMMIT 직후부터 리스너 실행 전까지 프로세스가 종료되는 경우
- 리스너가 실행됐지만 Kafka 발행에 실패하는 경우

`afterCommit` 자체는 발행할 이벤트를 영속적으로 저장하지 않는다. 따라서 발행 시점을 조정하는 것만으로는 누락된 이벤트를 복구할 근거가 생기지 않는다.

## 해결 방법: Transactional Outbox

Transactional Outbox는 **업무 데이터와 발행 의도인 Outbox 레코드를 같은 DB 트랜잭션에 저장**하는 패턴이다. 이후 별도의 전달 과정에서 Kafka에 이벤트를 발행한다. [Transactional Outbox 패턴 문서](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html)

```sql
BEGIN;

UPDATE charging_transactions
SET status = 'CLOSED'
WHERE transaction_id = 501 AND status = 'ACTIVE';

-- 애플리케이션에서 변경 행 수가 1인지 확인한다.
-- 1이 아니면 ROLLBACK하고, 아래 INSERT를 실행하지 않는다.

INSERT INTO outbox (event_id, aggregate_id, event_type, payload, status)
VALUES ('EVT-1001', '501', 'CHARGING_COMPLETED', '{"transactionId":501,"usedWh":2500}', 'PENDING');

COMMIT;
```

위 SQL은 애플리케이션의 조건 분기를 포함한 학습용 개략 예시다. 변경 행 수 확인과 ROLLBACK 분기는 SQL 주석만으로 실행되지 않으므로 애플리케이션에서 구현해야 한다. 실제 구현에서는 다음 조건을 확인한다.

- `event_id`의 유일성을 보장한다. `'EVT-1001'`은 설명용 값이며, 서로 다른 이벤트에는 고유 ID를 부여한다.
- 조건부 UPDATE가 성공한 경우에만 Outbox 레코드를 생성한다.
- 중복 업무 요청으로 같은 상태 전이의 이벤트가 다시 생성되지 않도록 한다.
- 허용된 업무 상태 전이인지 검증한다.
- 업무 UPDATE와 Outbox INSERT가 같은 DB의 같은 트랜잭션에 참여하도록 한다.

장애 시 결과는 다음과 같다.

- COMMIT 전에 트랜잭션이 중단되면 업무 UPDATE와 Outbox INSERT가 함께 롤백된다.
- COMMIT에 성공하면 업무 데이터와 Outbox의 발행 의도가 함께 남는다.
- Kafka가 중단돼도 Outbox 레코드를 보존해 복구 후 발행할 수 있다.

Outbox가 원자적으로 저장하는 범위는 **동일 DB 안의 업무 변경과 발행 의도**다. Kafka에 즉시 도착하는 것, Kafka에 물리적으로 정확히 1건만 기록되는 것, Consumer의 업무 처리가 자동으로 완료되는 것까지 보장하지는 않는다.

### Relay는 무엇을 할까?

Polling Relay는 일반적으로 다음 순서로 동작한다.

1. Outbox에서 발행이 완료되지 않은 이벤트를 조회한다.
2. Kafka로 전송하고 Producer ACK를 확인한다.
3. Outbox에 완료 이력을 기록하거나 상태를 갱신한다.
4. 실패하면 미완료 상태를 보존해 재시도하고, 복구 후 다시 조회한다.

여러 Relay가 같은 작업을 동시에 가져가지 않도록 PostgreSQL의 `SELECT ... FOR UPDATE SKIP LOCKED` 등을 사용할 수 있다. 이때 선점에 필요한 행 잠금을 트랜잭션 안에서 유지하거나, 잠금을 해제하기 전에 선점 상태를 기록해야 한다. `SKIP LOCKED`는 다른 트랜잭션이 잠근 행을 건너뛰는 기능이다. [PostgreSQL SELECT 문서](https://www.postgresql.org/docs/current/sql-select.html)

하지만 **Kafka ACK 이후 DB에 완료 이력을 기록하지 못해 발생하는 재발행까지 제거하지는 않는다.**

| 전달 방식 | 특징 | 주의할 점 |
| --- | --- | --- |
| Polling Relay: 락 유지형 | DB 행 잠금을 유지한 채 Kafka ACK까지 기다림 | ACK 대기 동안 DB 커넥션과 락을 오래 점유 |
| Polling Relay: 선점 후 커밋형 | 짧은 DB 트랜잭션으로 PROCESSING 상태를 기록한 뒤 전송 | 중간 종료 시 만료 작업 회수, 소유권 토큰과 조건부 갱신 필요 |
| CDC | 커밋된 Outbox 변경을 DB 변경 로그와 커넥터로 전달 | PostgreSQL 기준 복제 슬롯·WAL 보존, 커넥터 장애와 오프셋 관리 필요 |

Outbox는 발행 의도를 영속적으로 기록하는 패턴이고, Polling과 CDC는 그 기록을 전달하는 방식이다. 따라서 `Outbox + Polling`과 `Outbox + CDC` 모두 가능하다. CDC에서는 위의 Polling Relay처럼 매번 Outbox 상태를 완료로 갱신하는 흐름을 그대로 전제하지 않는다. [Debezium Outbox Event Router 문서](https://debezium.io/documentation/reference/stable/transformations/outbox-event-router.html)

## Outbox를 적용해도 중복 발행은 발생할 수 있다

1. Relay가 `EVT-1001`을 Kafka에 발행한다.
2. Kafka가 이벤트를 기록하고, Relay가 ACK를 확인한다.
3. Outbox에 발행 완료를 기록하기 전에 Relay가 종료된다.
4. 재시작한 Relay는 아직 미완료 상태인 `EVT-1001`을 다시 발행한다.

| 구분 | 첫 발행 후 | 재발행 후 |
| --- | --- | --- |
| Kafka 물리 기록 | 1건 | 누적 2건 가능 |
| 고유 eventId | 1개 | 여전히 1개 |
| 기대하는 업무 효과 | 1회 | 여전히 1회 |

따라서 Consumer는 `eventId` 기준의 중복 처리 이력과 업무 DB 변경을 **같은 DB 트랜잭션에 기록**해야 한다.

예를 들어 `processed_events(event_id UNIQUE)`에 처음 기록한 경우에만 업무 변경을 수행하고, 두 작업을 함께 커밋한다. 이미 처리한 `eventId`가 다시 오면 업무를 중복 반영하지 않는다. 같은 ID로 다른 payload가 들어오면 단순히 무시하지 않고 내용 충돌로 분리한다.

이 방식이 원자적으로 묶는 범위는 해당 Consumer의 로컬 DB 변경이다. 외부 결제나 다른 서비스 호출이 포함된다면 그 경계에도 별도의 멱등성·복구 정책이 필요하다.

Kafka Producer의 `enable.idempotence`는 Producer 내부 재시도로 생기는 중복을 방지한다. Relay가 같은 이벤트를 애플리케이션에서 다시 `send()`하는 것까지 자동으로 중복 제거하지는 않으므로, Producer 멱등성과 업무 멱등성을 구별해야 한다. [KafkaProducer API 문서](https://kafka.apache.org/44/javadoc/org/apache/kafka/clients/producer/KafkaProducer.html)

## 장애 시점에 따라 무엇이 남고 어떻게 복구할까?

다음 표는 업무 데이터와 Outbox를 같은 DB 트랜잭션에 저장하고, Polling Relay가 비트랜잭션 Kafka 전송의 ACK를 확인한 뒤 완료 이력을 기록하는 구현을 가정한다. Kafka 트랜잭션을 사용한다면 개별 전송 ACK뿐 아니라 Kafka 트랜잭션의 커밋 결과도 확인해야 한다.

| 장애 시점 | 업무 DB | Outbox | Kafka | 복구 |
| --- | --- | --- | --- | --- |
| 업무·Outbox가 커밋되지 않은 상태에서 트랜잭션 중단 | 롤백 | 롤백 | 없음 | 원래 업무 요청을 멱등하게 재시도 |
| COMMIT 성공 직후, Relay 발행 전 | 저장 | 미발행 상태 보존 | 없음 | Relay 재실행 |
| Relay 전송 중, ACK 미확인 | 저장 | 미완료 | 기록 여부 불확실 | 동일 eventId로 재전송하고 Consumer에서 멱등 처리 |
| ACK 성공 후, 완료 기록 전 | 저장 | 미완료 | 기록됨 | 재발행 가능성에 대비해 Consumer에서 멱등 처리 |
| 완료 기록 COMMIT 후 | 저장 | 완료 | 기록됨 | Relay 발행 절차 완료, Consumer 처리 결과는 별도 확인 |

ACK를 확인하지 못했다고 해서 브로커에 기록되지 않은 것은 아니다. 응답이 유실되면 실제 발행 여부가 불확실하므로, 중복 가능성을 전제로 복구해야 한다.

DB COMMIT의 응답을 받지 못한 경우에도 실제 커밋 여부가 불확실할 수 있다. 이를 곧바로 롤백으로 간주하지 않고, 요청 ID와 업무 상태·Outbox 기록을 조회해 결과를 확인한다.

## Outbox 외에는 방법이 없을까?

| 방식 | 장점 | 한계 |
| --- | --- | --- |
| 직접 발행 | 구조가 간단함 | DB 커밋과 Kafka 발행 사이에 장애 구간이 남음 |
| afterCommit | DB COMMIT 전에 완료 이벤트가 발행되는 상황을 피함 | 자체적으로는 커밋 후 발행 누락·실패를 복구할 영속 기록이 없음 |
| Outbox + Polling Relay | 발행 의도를 보존하고 재시도 경로를 확보 | DB 폴링·완료 기록 비용, 물리 중복 가능성 |
| Outbox + CDC | DB 변경 로그를 기반으로 이벤트 전달 | 커넥터·WAL·복제 슬롯 등의 운영 부담 |
| 2PC 등 분산 트랜잭션 | 프로토콜을 지원하는 참여자 사이에서 원자적 커밋 조정 | 참여자 지원 제약, 장애 시 대기·가용성 문제, 운영 복잡도 |

2PC는 참여 시스템과 트랜잭션 매니저가 해당 프로토콜을 지원해야 적용할 수 있다. 위의 DB·Kafka 예제에 `@Transactional`을 붙이는 것만으로 2PC가 구성되는 것은 아니다.

장애 복구가 중요한 업무 이벤트라면 발행 의도나 이를 복원할 수 있는 근거를 영속적으로 남겨야 한다. 구현 방식은 DB 비용, 실시간성, 처리량, 팀의 운영 역량을 기준으로 선택한다.

## 발행 누락과 지연은 어떻게 탐지할까?

- **Outbox pending count:** 발행 완료가 확인되지 않은 이벤트 수
- **Oldest pending age:** 가장 오래된 미완료 이벤트의 대기시간
- **Kafka publish retry/error:** Kafka 발행 오류와 재시도 횟수
- **전달 지연 p95/p99:** DB COMMIT 이후 Kafka 최초 관찰까지의 시간 등 측정 경계를 정의한 지연 지표
- **대사:** 업무 마감 결과, 고유 `eventId`, 최종 Consumer 업무 효과 사이에 기대한 대응 관계가 성립하는지 비교

미완료 이벤트에는 `PENDING`뿐 아니라 장기간 남은 `PROCESSING`이나 재시도 대상도 포함해 확인한다. CDC 방식은 Polling 상태값을 그대로 쓰기보다 커넥터의 처리 위치와 지연 등 전달 방식에 맞는 지표를 함께 본다.

Kafka Consumer Lag이 0이더라도 **애초에 Kafka에 발행되지 않은 이벤트**는 찾아낼 수 없다. 업무 DB와 Outbox, 최종 소비 결과를 함께 확인해야 한다.

Pending 이벤트가 늘었다는 사실만으로 영구 누락을 단정해서도 안 된다. 입력량, Relay 처리율, Kafka 상태를 함께 분석해 일시적 적체인지 복구가 필요한 장애인지 구분한다.

원문: [05.02 Dual Write](https://app.notion.com/p/05-02-Dual-Write-41664b7e118d818ca452ec7df20be488)
