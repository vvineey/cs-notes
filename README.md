# CS

## [01. 운영체제·자료구조·동시 실행](01-operating-systems-and-data-structures/)

- [ ] 큐·덱·해시 자료구조
- [ ] 시간·공간 복잡도와 병목
- [ ] 프로세스와 스레드
- [ ] 스레드 풀과 작업 큐
- [ ] 경쟁 상태와 임계 영역
- [ ] 교착상태·기아·라이브락
- [ ] 메모리 가시성·ThreadLocal
- [ ] CPU 바운드와 I/O 바운드
- [ ] CPU 스케줄링·문맥 교환
- [ ] 블로킹·논블로킹 I/O
- [ ] 가상 메모리·Heap·Stack
- [ ] 메모리 가시성과 원자성

## [02. 네트워크·HTTP·API](02-network-and-api/)

- [ ] TCP 연결과 네트워크 장애
- [ ] 타임아웃 종류와 예산
- [ ] HTTP 메서드와 멱등성
- [ ] API 오류와 재시도 계약
- [ ] 동기 호출과 비동기 처리
- [ ] 연결 풀과 커넥션 생명주기
- [ ] TLS·인증·기본 보안
- [ ] DNS·IP·Port와 요청 경로
- [ ] TCP 바이트 스트림과 메시지 경계
- [ ] TLS 서버 인증서 검증
- [ ] 인증과 인가·Basic 인증·mTLS
- [ ] WebSocket과 WSS
- [ ] MQTT 발행·구독과 세션

## [03. 데이터베이스·SQL·저장 구조](03-database-and-sql/)

- [ ] 관계형 모델과 정규화
- [ ] PK·UNIQUE·FK
- [ ] 인덱스와 실행 계획
- [ ] INSERT·UPDATE와 조건부 갱신
- [ ] DB 커넥션 풀과 트랜잭션 점유
- [ ] WAL·Redo Log·내구성
- [ ] 조회와 상태 변경의 DB 비용
- [ ] 스키마 마이그레이션과 호환성
- [ ] JOIN·정렬·LIMIT·집계 SQL
- [ ] 복합 키와 조건부 UPSERT
- [ ] 숫자·시간·JSON 데이터 타입
- [ ] SQL 논리 처리와 누적 재조회 비용

## [04. 트랜잭션·락·데이터 정합성](04-transactions-and-consistency/)

- [ ] ACID 네 가지 속성
- [ ] BEGIN·COMMIT·ROLLBACK
- [ ] 트랜잭션 경계와 외부 I/O
- [ ] READ COMMITTED·REPEATABLE READ
- [ ] MVCC·스냅샷·현재 읽기
- [ ] SELECT FOR UPDATE
- [ ] SELECT FOR UPDATE SKIP LOCKED
- [ ] 낙관적 락과 비관적 락
- [ ] Spring @Transactional
- [ ] afterCommit·TransactionSynchronization
- [ ] 중복 요청과 업무 정합성
- [ ] JDBC autoCommit과 커넥션
- [ ] 격리 이상과 동시 갱신

## [05. 분산시스템·일관성 모델](분산시스템·일관성%20모델/)

- [x] [부분 장애](분산시스템·일관성%20모델/부분%20장애.md)
- [x] [Dual Write 문제](분산시스템·일관성%20모델/Dual%20Write.md)
- [ ] 데이터 정합성과 대사
- [ ] 최종적 일관성과 상태 전이
- [ ] 이벤트 중복·역순·지연·누락
- [ ] 전달 보장과 업무 처리 보장
- [ ] 분산 트랜잭션과 2PC
- [ ] Saga와 보상 트랜잭션
- [ ] CAP과 일관성/가용성 절충
- [ ] 이벤트 식별자·순번·버전
- [ ] 업무 상태 머신과 불변조건
- [ ] 발생 시각·수신 시각·논리 순번
- [ ] 원천 중복 키와 내용 충돌

## [06. 메시징·이벤트 기반 아키텍처](06-messaging/)

- [ ] 메시지 브로커의 역할
- [ ] 이벤트 기반 아키텍처
- [ ] Kafka 기본 구조
- [ ] Producer ACK와 acks
- [ ] Producer 재시도와 중복 발행
- [ ] Kafka Producer 멱등성
- [ ] Consumer 처리와 Offset Commit
- [ ] DB COMMIT·Producer ACK·Offset COMMIT
- [ ] Consumer 멱등 처리
- [ ] 메시지 순서와 파티션 키
- [ ] Consumer Group·리밸런싱
- [ ] 배치 소비와 ACK 단위
- [ ] 메시지 스키마·호환성
- [ ] Kafka 이외의 브로커 선택
- [ ] Kafka 복제·Leader·ISR
- [ ] Kafka Controller와 KRaft
- [ ] 파티션 편중과 소비 병렬도
- [ ] Kafka 보존·재생·조회 모델 복구
- [ ] Kafka 전송 설정과 배치
- [ ] Consumer 처리시간과 오류 정책
- [ ] RabbitMQ 작업 큐와 전달 확인

## [07. 신뢰성 패턴·Outbox·Inbox·CDC](07-outbox-inbox-and-cdc/)

- [ ] Transactional Outbox
- [ ] Outbox 테이블 구조
- [ ] Outbox 상태 전이
- [ ] Outbox 원자성과 한계
- [ ] Polling Relay 처리 흐름
- [ ] Relay 트랜잭션 유지형과 선점 후 커밋형
- [ ] Polling 주기와 배치 크기
- [ ] Lease와 만료 작업 회수
- [ ] Claim Token·Fencing Token
- [ ] Relay 다중 실행과 선점 경쟁
- [ ] 재시도·영구 실패 관리
- [ ] Transactional Inbox / Processed Events
- [ ] Polling Relay vs CDC
- [ ] Debezium·Kafka Connect·WAL
- [ ] 직접 발행·afterCommit·Outbox 비교
- [ ] Outbox가 필요하지 않은 경우
- [ ] Outbox·Inbox의 대안
- [ ] CDC 운영 비용
- [ ] Outbox·Inbox 정리와 재처리 기간
- [ ] CDC 관찰 지연과 poll 설정

## [08. 장애 모델·복구·운영 안정성](08-failure-recovery/)

- [ ] DB COMMIT 전 종료
- [ ] DB COMMIT 직후 종료
- [ ] Relay 선점 직후 종료
- [ ] Kafka 전송 실패·통신 장애
- [ ] Kafka ACK 후 Relay 종료
- [ ] Consumer DB COMMIT 후 Offset 미커밋
- [ ] 여러 Relay 선점 경합
- [ ] 장애 후 최종 불변조건
- [ ] 재시도·지수 백오프·지터
- [ ] 서킷 브레이커·벌크헤드
- [ ] Backpressure·Load Shedding
- [ ] DLQ·재처리·복구 절차
- [ ] 장애 런북과 변경 이력

## [09. 성능·모니터링·병목 분석](09-performance-and-observability/)

- [ ] 지연·처리량·대기시간·적체
- [ ] p50·p95·p99 해석
- [ ] Polling·배치·동시 전송
- [ ] DB 부하와 이벤트 파이프라인 비용
- [ ] 트랜잭션 점유·락 대기·풀 대기
- [ ] 대기열 이론·포화 현상
- [ ] 로그·메트릭·분산 추적
- [ ] 복구 운영 지표
- [ ] SLO·알림 기준
- [ ] 성능과 정합성 트레이드오프
- [ ] 부하 생성 방식과 입력 지연
- [ ] 측정 경계와 자원 범위
- [ ] 긴 거래와 편중 입력의 성능

## [10. 테스트·장애 주입·데이터 대사](10-testing-and-reconciliation/)

- [ ] 정상 경로와 실패 경로
- [ ] Fault Injection
- [ ] 장애별 기대 상태 매트릭스
- [ ] 재시작과 복구 검증
- [ ] 물리 중복과 논리적 중복
- [ ] 멱등성·원자성 테스트
- [ ] 원천 대비 데이터 대사
- [ ] 성능·복구 지표 계측
- [ ] 실험 범위와 한계
- [ ] 테스트 피라미드·통합 테스트
- [ ] 동시성·레이스 테스트
- [ ] 부하 테스트·워밍업·재현성
- [ ] Testcontainers·환경 격리
- [ ] 독립 정답과 재생 동등성
- [ ] 실험 환경과 결과 출처

## [11. Java·JVM·Spring·백엔드 아키텍처](11-java-jvm-and-spring/)

- [ ] Spring 트랜잭션 프록시와 전파
- [ ] Spring 트랜잭션 롤백 규칙
- [ ] 이벤트 리스너와 afterCommit
- [ ] 계층 분리와 의존성 방향
- [ ] 동기 API와 비동기 작업 상태
- [ ] 배포·무중단 변경·호환성
- [ ] 설계 대안 평가 기준
- [ ] Java 객체·불변성·equals/hashCode
- [ ] Java 예외와 Future
- [ ] JVM 메모리와 GC
- [ ] Spring IoC·DI·Bean 생명주기
- [ ] JDBC와 Spring JDBC
- [ ] 멀티모듈·서비스·데이터 소유권

## [12. 보험·증권 IT·SI/SM·업무 시스템 이해](12-business-systems/)

- [ ] 사업 구조·SI/SM
- [ ] 보험계열사 SW엔지니어 업무
- [ ] 보험 계약·청구·심사·지급
- [ ] 보험 시스템 간 데이터 연동
- [ ] 보험 데이터 정합성·감사 가능성
- [ ] 증권사 SW엔지니어 업무
- [ ] 증권 계좌·예수금·잔고·매매 가능 수량
- [ ] 주문·정정·취소·체결 처리
- [ ] 시세 수신·호가·실시간 데이터 처리
- [ ] 주문 중복·순서·유실·재처리
- [ ] 증권 원장 정합성·감사 추적
- [ ] 증권 결제·정산·대사
- [ ] 거래일·영업일·결제일·배치 마감
- [ ] 안정적 운영·모니터링·재처리
- [ ] 기존 시스템 개선 원칙
- [ ] 개인정보·권한·감사 로그
- [ ] 배치·정산·대사
- [ ] 직무 연결에서 사실과 가정 구분

## [13. OCPP·충전 거래·계량 도메인](13-ocpp-and-metering/)

- [ ] Charging Station·EVSE·Connector
- [ ] OCPP 요청·응답과 수신 책임
- [ ] TransactionEvent와 거래 순번
- [ ] 상태 관측과 거래 원천
- [ ] 전력과 전력량·누적 계량
- [ ] 계량 문맥과 단위 정규화
- [ ] 정밀도와 시간 표현
- [ ] 거래 후보·보류·마감
- [ ] 독립 대사와 감사 이력
- [ ] 요금·청구·결제 정합성
- [ ] AI 스케줄·장비 명령의 안전성

## [14. 실행 환경·배포·운영 기초](14-deployment-and-operations/)

- [ ] 컨테이너와 프로세스 격리
- [ ] Docker Compose와 서비스 연결
- [ ] 영속 볼륨과 장애 범위
- [ ] Graceful Shutdown과 강제 종료
- [ ] 버전·설정·마이그레이션
- [ ] 백업·복구·RPO/RTO

## [90. 프로젝트 적용 사례 및 면접 준비](90-project-and-interview/)

- [프로젝트 적용 사례](90-project-and-interview/project-cases/)
- [면접용 목차](90-project-and-interview/interview-preparation/)
