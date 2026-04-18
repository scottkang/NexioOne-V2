# logging-service Specification

**Status**: [Not Implemented] - 제품 범위에 포함되며, 본 문서는 목표 로깅 서비스 기능을 정의한다.

## 개요
NexioOne의 런타임 실행 결과(Execution Log)를 비동기적으로 수집하여 영속화하고, 이를 조회할 수 있는 API를 제공할 통합 로깅 서비스이다.

## 기술 스택
- **Framework**: Spring Boot
- **Messaging**: RabbitMQ (Consumer)
- **Persistence**: Spring Data JPA (RDB)

## 주요 기능
...
### 1. Execution Event Consumption
- `my-backend`가 발행한 실행 상태 변경 이벤트를 RabbitMQ 큐에서 수신.

### 2. Lifecycle Logging (영속화)
- 수신된 이벤트를 기반으로 전체 실행 흐름 및 개별 Step의 실행 결과 저장.

### 3. Execution Data Query
- `my-console` 트랜잭션 뷰어용 REST 엔드포인트 제공.

## 연관 문서
- [Runtime Execution Logging Architecture](../runtime/runtime-execution-logging-architecture.md)
- [API Spec](../api/api-spec.md)
