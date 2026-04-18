# logging-service Detailed Design (Event Consumption & Hybrid Observability)

**Status**: [Draft/Detailed Design]
**Parent Module**: [logging-service](logging-service.md)
**Related Architecture**: [Runtime Execution Logging Architecture](../runtime/runtime-execution-logging-architecture.md)

## 1. 개요
본 문서는 `my-backend`에서 발행된 실행 이벤트를 RabbitMQ를 통해 비동기적으로 수신하고, 이를 OpenTelemetry(OTEL) 트레이싱 정보와 결합하여 고도화된 관측성(Observability)을 제공하기 위한 `logging-service`의 상세 구현 설계를 정의한다.

## 2. 하이브리드 관측성 모델 (Hybrid Observability)

비즈니스 로직과 시스템 인프라 정보를 분리하여 수집하되, 식별자를 통해 상호 참조한다.

| 구분 | 비즈니스 실행 로그 (RabbitMQ + RDB) | 시스템 트레이스/메트릭 (OTEL) |
| :--- | :--- | :--- |
| **담당** | `logging-service` | OTEL Collector + Jaeger/Grafana |
| **데이터** | 노드 입출력 페이로드, 실행 상태, 재처리 컨텍스트 | 서비스 간 지연 시간, 시스템 리소스, 호출 체인 |
| **연결고리** | `executionId`, `traceId`, `spanId` | `executionId` (Span Attribute) |

## 3. 데이터 모델 및 DB 스키마 상세

### 3.1 Entity 계층 구조 및 OTEL 식별자 추가
- **ExecutionEventEntity**: 원본 이벤트 Envelope 보존 (Audit용).
- **ExecutionRecordEntity**: 글로벌 `trace_id`를 보유하여 전체 서비스 트레이스와 연결.
- **ExecutionStepEntity**: 개별 노드의 `span_id`를 보유하여 특정 성능 병목 지점과 연결.

### 3.2 상세 스키마 (OTEL 필드 포함)

#### TB_EXECUTION_RECORD (Master)
| 컬럼명 | 타입 | 제약사항 | 설명 |
| :--- | :--- | :--- | :--- |
| `execution_id` | VARCHAR(64) | PK | 실행 ID |
| `trace_id` | VARCHAR(64) | Index | **OTEL Trace ID (표준 연결고리)** |
| `project_id` | BIGINT | Index | 프로젝트 ID |
| `flow_id` | BIGINT | Index | 플로우 ID |
| `status` | VARCHAR(20) | | `SUCCESS`, `FAILURE`, `CANCELED` |
| `started_at` | TIMESTAMP | | 실행 시작 시각 |
| `finished_at` | TIMESTAMP | | 실행 종료 시각 |
| `duration_ms` | BIGINT | | 비즈니스 관점의 총 소요 시간 |
| `requested_by` | VARCHAR(128) | | 실행 요청자 정보 |

#### TB_EXECUTION_STEP (Detail)
| 컬럼명 | 타입 | 제약사항 | 설명 |
| :--- | :--- | :--- | :--- |
| `id` | BIGINT | PK | |
| `execution_id` | VARCHAR(64) | FK | |
| `span_id` | VARCHAR(64) | | **OTEL Span ID (노드별 성능 연결)** |
| `step_id` | VARCHAR(128) | | 노드 ID (Flow Designer 상의 ID) |
| `step_order` | INT | | 실행 순서 |
| `status` | VARCHAR(20) | | 노드 실행 상태 |
| `duration_ms` | BIGINT | | 노드 소요 시간 |
| `message` | TEXT | | 오류 메시지 등 비고 |

## 4. OTEL 연계 아키텍처 및 흐름

### 4.1 Trace Context 전파
1.  **API Gateway**: 요청 진입 시 OTEL Trace Context 생성 및 `X-Nexio-Execution-Id` 주입.
2.  **my-backend**: 실행 엔진이 노드 실행 시마다 새로운 Span을 생성하고, 이벤트 발행 시 페이로드에 `traceId`와 `spanId`를 포함한다.
3.  **logging-service**: RabbitMQ 이벤트를 소비할 때 포함된 `traceId`를 DB에 함께 저장한다.

### 4.2 시각화 연동 (Deep Link)
- `logging-service` API는 조회 결과에 OTEL 시각화 도구(Jaeger/Grafana Tempo)로 연결되는 **Trace Link**를 동적으로 생성하여 반환한다.

## 5. RabbitMQ Consumer 로직 설계

### 5.1 멱등성(Idempotency) 보장
- **`event_id` 기반 중복 체크**: `TB_EXECUTION_EVENT`에 `event_id` Unique 제약을 두어 중복 수신 시 무시하거나(Ignore) Upsert 처리.
- **`execution_id` 기반 상태 업데이트**: `execution.started`와 `execution.completed` 이벤트가 순서가 바뀌어 오더라도, 최종 상태가 보존되도록 `started_at`과 `finished_at`을 조건부 업데이트.

### 5.2 에러 핸들링 및 재시도 (DLQ)
- **Retry Policy**: DB 일시 장애 등 복구 가능한 에러 시 3회 재시도 (Exponential Backoff).
- **Dead Letter Queue (DLQ)**: `runtime.execution.logging.dlq`로 이동.

## 6. Read API 설계 고도화

### 6.1 실행 상세 조회 (`/executions/{executionId}`)
- **Response 확장**: 마스터 정보와 상세 단계 정보를 포함하며, 각 단계별 `spanUrl`과 전체 `traceUrl`을 포함한다.

## 7. 구현 로드맵 (OTEL 포함)
1.  **Phase 1**: 기존 RabbitMQ 기반 영속화 구조 완성.
2.  **Phase 2**: `my-backend`에 OTEL SDK 연동 및 이벤트 페이로드에 Trace 정보 주입.
3.  **Phase 3**: `logging-service` DB 스키마 확장 및 Trace ID 연동 저장.
4.  **Phase 4**: `my-console` UI와 연계한 통합 관측성 대시보드 제공.

## 8. 보안 및 컴플라이언스
- **Data Masking**: 민감정보 마스킹 확인.
- **Retention Policy**: 운영 데이터 보관 기간(예: 30일) 이후의 로그 정리.
