# Internal API Contract & Communication Design

**Status**: [Baseline]
**Related Modules**: [my-console-backend](../modules/my-console-backend.md), [my-backend](../modules/my-backend.md), [logging-service](../modules/logging-service.md)

## 1. 목적
- 본 문서는 `my-console-backend -> my-backend` 실행 요청 계약과 `my-backend -> logging-service` 이벤트 계약의 최소 기준을 고정한다.
- 현재 구현은 없으므로, 본 문서는 우선 개발 순서를 맞추기 위한 목표 계약으로 사용한다.
- 상세 schema는 `internal-execution-schema.md`를 함께 기준으로 사용한다.

## 2. 모듈 간 통신 매트릭스
| Source | Target | Protocol | 목적 |
|---|---|---|---|
| `my-console-backend` | `my-backend` | REST | dry-run, execute-stub, 상태 조회 |
| `my-backend` | `logging-service` | RabbitMQ | 실행 상태 이벤트 전달 |
| `my-console-backend` | `api-gw` | gRPC/xDS 또는 동등 제어 채널 | 라우팅/정책 동기화 |

## 3. `my-console-backend -> my-backend` 실행 계약

### 3.1 Endpoint
- `POST /execute`
- `POST /execute/async`

### 3.2 Request Payload
```json
{
  "projectId": 1,
  "deploymentId": 101,
  "executionId": "exec-20260418-0001",
  "requestedBy": "owner@nexioone.local",
  "flowDefinition": {
    "flowId": 10,
    "flowKey": "customer-sync",
    "version": "v1",
    "nodes": [
      {
        "id": "start",
        "type": "START",
        "config": {}
      }
    ],
    "edges": []
  },
  "connectionProfiles": [
    {
      "id": "db-source",
      "type": "JDBC",
      "properties": {
        "url": "jdbc:postgresql://db:5432/app",
        "username": "app"
      },
      "secretsRef": {
        "passwordKey": "secret/db-source/password"
      }
    }
  ],
  "flowBindings": [
    {
      "role": "INPUT",
      "bindingKey": "requestBody",
      "required": true,
      "dataDefinitionId": 7,
      "dataDefinitionVersion": 3,
      "schemaJson": {
        "type": "object",
        "required": ["customerId"]
      }
    }
  ],
  "inputContext": {
    "customerId": "C100"
  },
  "options": {
    "mode": "EXECUTE_STUB",
    "async": false,
    "trace": true,
    "timeoutMs": 30000
  }
}
```

### 3.3 Payload Field Rules
#### Top-level identifiers
- `projectId`: 필수 integer
- `deploymentId`: 필수 integer
- `executionId`: 필수 string
- `requestedBy`: 필수 string
- `projectId`와 `deploymentId`는 status 조회 경로와 event payload의 정본 식별자로 사용한다.
- `requestedBy`는 감사/audit 및 `execution.started` 이벤트 payload에 그대로 반영한다.

#### `flowDefinition`
- 필수 object
- 최소 필드: `flowId`, `flowKey`, `version`, `nodes`, `edges`
- `nodes`의 각 항목은 최소 `id`, `type`, `config`를 포함
- 이번 프로그램 허용 노드 타입: `START`, `END`, `MAPPING`, `REST_CLIENT`, `SQL_EXECUTOR`
- `DRY_RUN`과 `EXECUTE_STUB`의 실제 지원 수준은 `runtime-node-support-matrix.md`를 따른다.

#### `connectionProfiles`
- optional array
- 각 항목 최소 필드: `id`, `type`
- `properties`에는 비민감 설정만 포함
- 비밀번호, 토큰, 키는 평문이 아니라 `secretsRef`로 전달

#### `flowBindings`
- optional array, 기본값 `[]`
- deployment snapshot에서 계산된 Flow binding 목록이다.
- runtime은 이번 프로그램에서 최소한 `INPUT`, `OUTPUT` binding을 사용해 시작/종료 payload 검증 기준을 맞춘다.
- 각 항목 최소 필드: `role`, `bindingKey`, `required`, `dataDefinitionId`, `dataDefinitionVersion`
- `schemaJson`이 포함되면 runtime validation의 직접 기준으로 사용한다.
- binding snapshot은 요청 시점의 배포 정본이며, runtime은 control plane DB를 재조회하지 않는다.

#### `inputContext`
- 필수 object
- 최상위 key는 문자열
- 배열, 객체, 문자열, 숫자, boolean, null 허용

#### `options`
- 필수 object
- `mode`: `DRY_RUN` 또는 `EXECUTE_STUB`
- `async`: boolean
- `trace`: boolean
- `timeoutMs`: positive integer

### 3.4 Sync Response
Response `200`:
```json
{
  "executionId": "exec-20260418-0001",
  "mode": "EXECUTE_STUB",
  "status": "SUCCEEDED",
  "outputContext": {
    "customerId": "C100"
  },
  "steps": [
    {
      "nodeId": "start",
      "status": "SUCCEEDED"
    }
  ]
}
```

### 3.5 Async Response
Response `202`:
```json
{
  "requestId": "req-20260418-0001",
  "executionId": "exec-20260418-0002",
  "status": "ACCEPTED",
  "statusUrl": "/api/projects/1/runtime/executions/exec-20260418-0002"
}
```

### 3.6 Async Status Values
- `ACCEPTED`
- `RUNNING`
- `SUCCEEDED`
- `FAILED`
- `CANCELED`

### 3.7 Async Status Ownership
- 비동기 실행의 표준 상태 조회 엔드포인트는 `my-backend`의 `GET /api/projects/{projectId}/runtime/executions/{executionId}`다.
- `my-backend`는 실행 중 상태(`ACCEPTED`, `RUNNING`)의 정본(source of truth)이다.
- 실행 완료 후 `logging-service`는 이벤트 기반 read model을 구축하며, `my-console-backend`는 사용자용 실행 이력/검색 API를 해당 read model에서 제공할 수 있다.
- 이번 프로그램에서는 "실행 제어용 상태 조회"와 "실행 이력 조회"를 분리한다.
- `statusUrl`의 `{projectId}`는 요청 payload의 `projectId`와 동일해야 한다.

## 4. Idempotency 규칙

### 4.1 Header
- 비동기 요청은 `Idempotency-Key` 헤더를 허용한다.
- 동기 요청에서는 사용하지 않는다.

### 4.2 의미
- 동일 `Idempotency-Key`와 동일 payload면 같은 실행 요청으로 간주한다.
- 동일 `Idempotency-Key`와 다른 payload면 `409`를 반환한다.

### 4.3 보관
- 키 보관 기간 기본값: `24h`
- 보관 만료 후 동일 키는 신규 요청으로 처리할 수 있다.

### 4.4 재시도 응답
- 이전 요청이 `ACCEPTED` 또는 `RUNNING`이면 기존 `requestId`, `executionId`, `statusUrl`을 반환한다.
- 이전 요청이 `SUCCEEDED` 또는 `FAILED`이면 기존 최종 상태를 반환한다.

### 4.5 에러 응답 예시
```json
{
  "timestamp": "2026-04-18T09:40:00Z",
  "status": 409,
  "error": "Conflict",
  "code": "BE-2103",
  "message": "idempotency key already used with different payload",
  "path": "/execute/async"
}
```

## 5. `my-backend -> logging-service` 이벤트 계약

### 5.1 Exchange / Routing Key
- 이벤트 transport와 schema의 단일 기준은 `runtime-event-schema.md`를 따른다.
- Exchange: `runtime.execution`
- Routing Keys:
  - `execution.started`
  - `execution.step.completed`
  - `execution.completed`
  - `execution.failed`

### 5.2 Common Event Fields
```json
{
  "eventId": "evt-20260418-0001",
  "eventType": "execution.completed",
  "schemaVersion": 1,
  "occurredAt": "2026-04-18T09:45:00Z",
  "source": "my-backend",
  "payload": {}
}
```

### 5.3 Started Event
```json
{
  "eventId": "evt-20260418-0001",
  "eventType": "execution.started",
  "schemaVersion": 1,
  "occurredAt": "2026-04-18T09:45:00Z",
  "source": "my-backend",
  "payload": {
    "executionId": "exec-20260418-0001",
    "projectId": 1,
    "flowId": 10,
    "flowKey": "customer-sync",
    "mode": "EXECUTE_STUB",
    "requestedBy": "owner@nexioone.local",
    "requestedAt": "2026-04-18T09:45:00Z"
  }
}
```

### 5.4 Step Completed Event
```json
{
  "eventId": "evt-20260418-0002",
  "eventType": "execution.step.completed",
  "schemaVersion": 1,
  "occurredAt": "2026-04-18T09:45:01Z",
  "source": "my-backend",
  "payload": {
    "executionId": "exec-20260418-0001",
    "projectId": 1,
    "flowId": 10,
    "nodeId": "start",
    "status": "SUCCEEDED",
    "durationMs": 5,
    "message": "step completed"
  }
}
```

### 5.5 Completed Event
```json
{
  "eventId": "evt-20260418-0003",
  "eventType": "execution.completed",
  "schemaVersion": 1,
  "occurredAt": "2026-04-18T09:45:02Z",
  "source": "my-backend",
  "payload": {
    "executionId": "exec-20260418-0001",
    "projectId": 1,
    "flowId": 10,
    "flowKey": "customer-sync",
    "mode": "EXECUTE_STUB",
    "status": "SUCCEEDED",
    "startedAt": "2026-04-18T09:45:00Z",
    "finishedAt": "2026-04-18T09:45:02Z",
    "durationMs": 10,
    "summary": {
      "totalSteps": 1,
      "succeededSteps": 1,
      "failedSteps": 0
    }
  },
}
```

### 5.6 Event Projection Rules
- 모든 이벤트 payload는 `projectId`, `flowId`, `executionId`를 포함해야 한다.
- `execution.started`는 `requestedBy`, `requestedAt`를 포함해야 한다.
- `execution.step.completed`는 최소 `nodeId`, `status`, `durationMs`를 포함해야 한다.
- `execution.completed`, `execution.failed`는 `startedAt`, `finishedAt`, `durationMs`를 포함해야 한다.
- step/event payload에는 secret 원문, `secretsRef` 전체 object, connection secret key를 포함하지 않는다.

## 6. 버전 및 호환 정책
- 새 필드는 optional로만 추가한다.
- 기존 필드의 이름 변경, 타입 변경, 삭제는 메이저 버전 변경 없이는 금지한다.
- `my-console-backend`는 worker가 지원하지 않는 필드를 보내면 안 된다.
- worker가 알 수 없는 optional field는 무시할 수 있다.
- stage 1 범위를 넘는 노드 또는 옵션 필드는 `422` + `BE-2102` 또는 동등 validation error로 거부한다.

## 7. 참조
- `docs/spec/api/internal-execution-schema.md`
- `docs/spec/api/global-error-code-map.md`
- `docs/spec/runtime/runtime-event-schema.md`
