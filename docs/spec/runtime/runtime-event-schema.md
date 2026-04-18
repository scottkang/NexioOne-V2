# Runtime Event Schema Baseline

**Status**: [Baseline]

## 1. 목적
- `my-backend -> logging-service` 실행 이벤트 계약을 단일 schema로 고정한다.

## 2. Transport
- exchange: `runtime.execution`
- exchange type: `topic`
- queue: `runtime.execution.logging`
- DLQ queue: `runtime.execution.logging.dlq`
- routing keys:
  - `execution.started`
  - `execution.step.completed`
  - `execution.completed`
  - `execution.failed`

## 3. Envelope
```json
{
  "eventId": "evt-20260418-0001",
  "eventType": "execution.completed",
  "schemaVersion": 1,
  "occurredAt": "2026-04-18T09:45:02Z",
  "source": "my-backend",
  "payload": {}
}
```

## 4. Common Rules
- 모든 이벤트는 envelope를 사용한다.
- `schemaVersion`의 기본값은 `1`이다.
- 민감정보는 payload에서 마스킹 또는 제외한다.
- 전달 보장 수준은 `at-least-once`다.
- 소비자는 `eventId` 기준 멱등 처리한다.
- 모든 이벤트 payload는 최소 `projectId`, `flowId`, `executionId`를 포함해야 한다.
- 실행 요청 payload의 `requestedBy`, `projectId`는 event projection 시 손실되면 안 된다.
- `deploymentId`, `flowBindings`, `connectionProfiles` 전체 payload는 이벤트에 직접 싣지 않는다.

## 5. Payload Schemas

### 5.1 execution.started
```json
{
  "executionId": "exec-20260418-0001",
  "projectId": 1,
  "deploymentId": 101,
  "flowId": 10,
  "flowKey": "customer-sync",
  "mode": "EXECUTE_STUB",
  "requestedBy": "owner@nexioone.local",
  "requestedAt": "2026-04-18T09:45:00Z"
}
```

### 5.2 execution.step.completed
```json
{
  "executionId": "exec-20260418-0001",
  "projectId": 1,
  "flowId": 10,
  "nodeType": "START",
  "nodeId": "start",
  "status": "SUCCEEDED",
  "startedAt": "2026-04-18T09:45:00Z",
  "finishedAt": "2026-04-18T09:45:01Z",
  "durationMs": 5,
  "message": "step completed"
}
```

### 5.3 execution.completed
```json
{
  "executionId": "exec-20260418-0001",
  "projectId": 1,
  "deploymentId": 101,
  "flowId": 10,
  "flowKey": "customer-sync",
  "mode": "EXECUTE_STUB",
  "status": "SUCCEEDED",
  "startedAt": "2026-04-18T09:45:00Z",
  "finishedAt": "2026-04-18T09:45:02Z",
  "durationMs": 2000,
  "summary": {
    "totalSteps": 1,
    "succeededSteps": 1,
    "failedSteps": 0
  }
}
```

### 5.4 execution.failed
```json
{
  "executionId": "exec-20260418-0001",
  "projectId": 1,
  "deploymentId": 101,
  "flowId": 10,
  "flowKey": "customer-sync",
  "mode": "EXECUTE_STUB",
  "status": "FAILED",
  "startedAt": "2026-04-18T09:45:00Z",
  "finishedAt": "2026-04-18T09:45:02Z",
  "durationMs": 2000,
  "error": {
    "code": "BE-2301",
    "message": "stub execution failed"
  }
}
```

## 6. Event Projection Rules
- `execution.started`
  - 실행 요청 식별자와 요청자 문맥을 남긴다.
- `execution.step.completed`
  - node trace 최소 필드 `nodeId`, `nodeType`, `status`, `startedAt`, `finishedAt`, `durationMs`를 남긴다.
- `execution.completed`, `execution.failed`
  - 최종 상태 판단에 필요한 `deploymentId`, `startedAt`, `finishedAt`, `durationMs`를 남긴다.
- 이벤트 payload에는 secret 원문, `secretsRef`, binding schema 전체를 포함하지 않는다.

## 7. Non-Goals For This Program
- result archive/object storage reference
- replay/reprocess 이벤트
- tenant-aware multi-region routing

## 8. DLQ Rule
- 복구 불가능한 schema validation 실패 또는 persistence 실패는 DLQ로 이동할 수 있다.
- DLQ 메시지는 원본 envelope를 유지한다.

## 9. 참조
- `docs/spec/api/internal-api-contract-design.md`
- `docs/spec/runtime/runtime-execution-logging-architecture.md`
