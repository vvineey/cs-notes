# 면접용 목차

## 01. 프로젝트 설명

- [ ] 프로젝트가 해결하려는 문제
- [ ] 전체 아키텍처와 데이터 흐름
- [ ] MQTT 상태 관제와 OCPP 거래 수신
- [ ] 서비스 분리와 데이터 소유권
- [ ] 구현 범위와 목표 설계
- [ ] 본인이 내린 설계 결정과 선택 근거

## 02. 프로젝트 적용 사례

- [ ] 최신 상태의 조건부 UPSERT
- [ ] OCPP 거래 식별자와 seqNo
- [ ] 중복·역순·누락·내용 충돌 처리
- [ ] 계량값 정규화와 거래 후보 계산
- [ ] PROVISIONAL·HOLD·후보 무효화
- [ ] DB와 Kafka의 Dual Write
- [ ] 직접 발행·afterCommit·Outbox 비교
- [ ] 합성 Relay의 행 잠금과 트랜잭션 경계
- [ ] WSS 시험용 Relay의 자동 커밋과 발행 완료 기록
- [ ] Kafka ACK 후 종료와 동일 eventId 재발행
- [ ] Consumer DB 커밋 후 종료와 offset 재처리
- [ ] 원천·Outbox·Kafka·업무 효과 대사
- [ ] Kafka ISR 부족과 쓰기 실패·복구
- [ ] 충전소 키 편중과 소비 병렬도
- [ ] 긴 거래의 Inbox 재조회 비용
- [ ] Kafka 로그를 이용한 조회 모델 재구축
- [ ] Polling Relay와 CDC 비교

## 03. 운영체제·동시성

- [ ] 프로세스와 스레드의 차이
- [ ] 문맥 교환과 스레드 풀
- [ ] 경쟁 상태와 임계 영역
- [ ] 원자성과 메모리 가시성
- [ ] 교착상태와 락 순서
- [ ] CPU 바운드·I/O 바운드·블로킹 I/O

## 04. 네트워크·보안

- [ ] TCP와 메시지 경계
- [ ] HTTP와 WebSocket의 동작
- [ ] TLS 서버 인증서 검증
- [ ] 인증과 인가·Basic 인증·mTLS
- [ ] 타임아웃과 결과 불확실성
- [ ] MQTT QoS와 브로커 간 전달 범위

## 05. 데이터베이스·트랜잭션

- [ ] ACID와 트랜잭션 경계
- [ ] PK·UNIQUE·복합 키
- [ ] 인덱스와 실행 계획
- [ ] 격리 수준과 MVCC
- [ ] SELECT FOR UPDATE·SKIP LOCKED
- [ ] 낙관적 락·비관적 락·조건부 갱신
- [ ] JDBC autoCommit과 커넥션 풀
- [ ] DB 트랜잭션과 외부 I/O

## 06. 메시징·분산시스템

- [ ] Kafka를 선택한 이유
- [ ] Kafka와 RabbitMQ의 역할
- [ ] 파티션 키와 순서 보장
- [ ] Consumer Group과 리밸런싱
- [ ] acks·복제·ISR·min.insync.replicas
- [ ] Producer 멱등성과 업무 멱등성
- [ ] DB COMMIT·Producer ACK·Offset COMMIT
- [ ] At-least-once·Exactly-once의 보장 범위
- [ ] Inbox·Outbox·CDC의 역할과 한계
- [ ] 재시도·백오프·DLQ·백프레셔

## 07. Java·Spring

- [ ] 객체 불변성과 equals/hashCode
- [ ] BigDecimal의 정밀도와 비교
- [ ] Future의 대기·타임아웃·예외
- [ ] JVM 메모리·GC·JIT
- [ ] Spring IoC·DI·Bean 생명주기
- [ ] 트랜잭션 프록시·전파·롤백
- [ ] self-invocation과 afterCommit
- [ ] JDBC와 Spring JDBC

## 08. 검증·성능·운영

- [ ] 단위·통합·E2E 검증 범위
- [ ] 장애 주입 시점과 독립 정답
- [ ] 재시작 후 불변조건과 대사
- [ ] 처리량·lag·p95·p99의 측정 경계
- [ ] 제공률·입력 지연·적체 해소
- [ ] 성능 비교의 동일 조건과 반복 측정
- [ ] 실험 환경·기능 플래그·결과 한계
- [ ] Docker 네트워크·볼륨·재시작
- [ ] Graceful Shutdown과 강제 종료

## 09. 직무 연결과 후속 개선

- [ ] 업무 시스템의 데이터 정합성
- [ ] 보험 시스템 간 연동·정산·대사
- [ ] 증권 주문·체결과 시세 처리
- [ ] 증권 계좌·잔고·원장 정합성
- [ ] 증권 결제·정산·대사
- [ ] 메시징·멱등성·장애 복구의 보험·증권 업무 적용
- [ ] 장애 대응·재처리·감사 이력
- [ ] 설계 대안과 트레이드오프
- [ ] 현재 남은 과제와 개선 우선순위
