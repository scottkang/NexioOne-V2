# Internal Execution Schema Baseline

## 1. 목적
- 본 문서는 `my-console-backend -> my-backend` 실행 payload를 구조 단위로 고정한다.
- 본 문서는 `internal-api-contract-design.md`의 필드 정의를 세부 schema 형태로 정리한 부속 문서다.

## 1.1 상태 Ownership 규칙
- `Async Accepted Response`와 `Execution Status Response`는 실행 제어용 상태를 표현하며 정본은 `my-backend`다.
- execution history/read model은 본 문서 범위가 아니며 `logging-service` consumer/read model 문서에서 다룬다.

## 2. ExecuteRequest
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
      },
      {
        "id": "end",
        "type": "END",
        "config": {}
      }
    ],
    "edges": [
      {
        "sourceNodeId": "start",
        "targetNodeId": "end"
      }
    ]
  },
  "connectionProfiles": [],
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

## 3. Top-Level Field Schema
| Field | Type | Required | Rules |
|---|---|---|---|
| `projectId` | integer | Y | positive |
| `deploymentId` | integer | Y | positive |
| `executionId` | string | Y | 1..120 chars, 요청 단위 고유값 |
| `requestedBy` | string | Y | 1..255 chars |
| `flowDefinition` | object | Y | 아래 `FlowDefinition` 규칙 준수 |
| `connectionProfiles` | array | N | 기본값 `[]` |
| `flowBindings` | array | N | 기본값 `[]`, deployment snapshot binding |
| `inputContext` | object | Y | key/value 컨텍스트 |
| `options` | object | Y | 아래 `ExecutionOptions` 규칙 준수 |

## 4. FlowDefinition Schema
| Field | Type | Required | Rules |
|---|---|---|---|
| `flowId` | integer | Y | positive |
| `flowKey` | string | Y | 1..120 chars |
| `version` | string | Y | 1..64 chars |
| `nodes` | array | Y | 최소 1개 |
| `edges` | array | Y | 없으면 `[]` |

### 4.1 FlowNode
| Field | Type | Required | Rules |
|---|---|---|---|
| `id` | string | Y | flow 내 유일 |
| `type` | string | Y | 이번 프로그램 기준 허용: `START`, `END`, `MAPPING`, `REST_CLIENT`, `SQL_EXECUTOR` |
| `config` | object | Y | 없으면 `{}` |
| `name` | string | N | 표시용 |
| `connectionRef` | string | N | `connectionProfiles[].id`와 일치해야 함 |

### 4.2 FlowEdge
| Field | Type | Required | Rules |
|---|---|---|---|
| `sourceNodeId` | string | Y | 존재하는 노드 id |
| `targetNodeId` | string | Y | 존재하는 노드 id |
| `condition` | string | N | 표현식 |

Validation Rules:
- `START` 노드는 정확히 1개
- `END` 노드는 최소 1개
- 모든 edge는 존재하는 node를 참조해야 함
- 순환 허용 여부는 차기 문서에서 확정 전까지 `불허`로 간주
- `DRY_RUN`에서는 `START`, `END`만 실행 의미론 대상으로 본다. 다른 지원 노드는 구조 검증 대상에는 포함될 수 있지만 실제 step 실행은 하지 않는다.
- `EXECUTE_STUB`에서는 `START`, `END`, `MAPPING`, `REST_CLIENT`, `SQL_EXECUTOR`를 허용한다.
- 각 노드 타입별 지원 수준은 `runtime-node-support-matrix.md`를 단일 기준으로 사용한다.

## 5. ConnectionProfile Schema
```json
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
```

| Field | Type | Required | Rules |
|---|---|---|---|
| `id` | string | Y | 요청 내 유일 |
| `type` | string | Y | `JDBC`, `REST`, `MQ`, `SFTP` |
| `properties` | object | N | 비민감 설정만 포함 |
| `secretsRef` | object | N | 민감 키 참조 |

Validation Rules:
- `properties`에 `password`, `token`, `secret`, `privateKey` 직접 포함 금지
- `FlowNode.connectionRef`가 있으면 동일 `id`의 profile이 존재해야 함
- 유형별 필수/옵션 필드와 secret key 규칙은 `connection-profile-contract.md`를 따른다.

## 6. InputContext Schema
- JSON object
- value 허용 타입:
  - string
  - number
  - boolean
  - null
  - object
  - array

Example:
```json
{
  "customerId": "C100",
  "retryCount": 0,
  "dryRun": true,
  "tags": [
    "vip",
    "sync"
  ]
}
```

## 7. FlowBinding Schema
```json
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
```

| Field | Type | Required | Rules |
|---|---|---|---|
| `role` | string | Y | `INPUT`, `OUTPUT`, `INTERNAL` |
| `bindingKey` | string | Y | flow 내 유일 logical key |
| `required` | boolean | Y | 필수 여부 |
| `dataDefinitionId` | integer | Y | positive |
| `dataDefinitionVersion` | integer | Y | positive |
| `schemaJson` | object | N | validation용 schema snapshot |

Validation Rules:
- `INPUT`, `OUTPUT`는 각 0..1개, `INTERNAL`은 복수 허용
- `schemaJson`이 있으면 runtime validation은 snapshot 기준으로 수행한다.
- 동일 요청 내 `role + bindingKey` 조합은 유일해야 한다.
- 이번 프로그램에서 `DRY_RUN`은 `INPUT` binding이 있으면 시작 payload validation에 사용한다.
- 이번 프로그램에서 `EXECUTE_STUB`은 `OUTPUT` binding이 있으면 종료 payload validation에 사용한다.

## 8. ExecutionOptions Schema
```json
{
  "mode": "EXECUTE_STUB",
  "async": false,
  "trace": true,
  "timeoutMs": 30000
}
```

| Field | Type | Required | Rules |
|---|---|---|---|
| `mode` | string | Y | `DRY_RUN`, `EXECUTE_STUB` |
| `async` | boolean | Y | 비동기 여부 |
| `trace` | boolean | Y | step trace 포함 여부 |
| `timeoutMs` | integer | Y | `1..600000` |

## 9. Async Accepted Response Schema
```json
{
  "requestId": "req-20260418-0001",
  "executionId": "exec-20260418-0002",
  "status": "ACCEPTED",
  "statusUrl": "/api/projects/1/runtime/executions/exec-20260418-0002",
  "statusOwner": "my-backend"
}
```

| Field | Type | Required | Rules |
|---|---|---|---|
| `requestId` | string | Y | 접수 요청 식별자 |
| `executionId` | string | Y | 실행 식별자 |
| `status` | string | Y | `ACCEPTED` 고정 |
| `statusUrl` | string | Y | 외부 execution status 조회 경로 |
| `statusOwner` | string | Y | `my-backend` 고정 |

## 10. Execution Status Response Schema
```json
{
  "executionId": "exec-20260418-0002",
  "mode": "EXECUTE_STUB",
  "status": "RUNNING",
  "requestedAt": "2026-04-18T09:30:00Z",
  "startedAt": "2026-04-18T09:30:01Z",
  "finishedAt": null,
  "statusOwner": "my-backend"
}
```

| Field | Type | Required | Rules |
|---|---|---|---|
| `executionId` | string | Y | 실행 식별자 |
| `mode` | string | Y | `DRY_RUN`, `EXECUTE_STUB` |
| `status` | string | Y | `ACCEPTED`, `RUNNING`, `SUCCEEDED`, `FAILED`, `CANCELED` |
| `requestedAt` | string | Y | ISO-8601 UTC |
| `startedAt` | string | N | 실행 시작 전이면 `null` 가능 |
| `finishedAt` | string | N | 종료 전이면 `null` |
| `statusOwner` | string | Y | `my-backend` 고정 |

## 11. Idempotency Validation Rules
- `Idempotency-Key`는 비동기 요청에서만 허용
- 동일 key + 동일 payload: 기존 응답 재사용
- 동일 key + 다른 payload: `409 Conflict`
- key 보관 기본값: `24h`
- 동일 key + 완료 상태(`SUCCEEDED`, `FAILED`, `CANCELED`)면 기존 최종 상태를 반환
- `statusUrl`과 `statusOwner`는 재시도 응답에서도 동일해야 한다
