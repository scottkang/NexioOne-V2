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
- `logging-service`는 envelope 원문 저장과 read model projection을 분리한다.
- 장기 이력/검색 projection은 이벤트 payload를 그대로 복제하지 않고 필요한 필드만 저장한다.

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

### 6.1 Projection Baseline
- `execution_event`
  - envelope 원문 저장
  - 필수 필드: `eventId`, `eventType`, `schemaVersion`, `occurredAt`, `source`, `payloadJson`
- `execution_record`
  - 단건 실행 요약 저장
  - 필수 필드: `executionId`, `projectId`, `deploymentId`, `flowId`, `flowKey`, `mode`, `status`, `requestedBy`, `requestedAt`, `startedAt`, `finishedAt`, `durationMs`
- `execution_step`
  - step trace 저장
  - 필수 필드: `executionId`, `projectId`, `flowId`, `nodeId`, `nodeType`, `status`, `startedAt`, `finishedAt`, `durationMs`, `message`

### 6.2 Projection Exclude Rules
- `connectionProfiles`, `flowBindings`, secret 원문, `secretsRef`, binding schema 전체는 projection에 저장하지 않는다.
- `inputContext`, `outputContext` 전체 원문은 이번 프로그램 projection 기본 필드에 포함하지 않는다.
- 대용량 결과 payload, archive reference, trace context 확장 필드는 차기 범위다.

## 7. Non-Goals For This Program
- result archive/object storage reference
- replay/reprocess 이벤트
- tenant-aware multi-region routing

## 8. DLQ Rule
- DLQ 이동 대상은 아래로 고정한다.
  - schema validation 실패
  - 역직렬화 불가
  - 필수 필드 누락
  - persistence 실패가 재시도 한도를 초과한 경우
- 일시적 DB 오류 등 복구 가능 오류는 즉시 DLQ로 보내지 않고 제한 횟수 재시도 후 판단한다.
- DLQ 메시지는 원본 envelope를 유지한다.
- DLQ 메시지는 운영자 수동 분석/재처리 대상이며, 이번 프로그램에는 자동 재주입을 포함하지 않는다.

## 9. 참조
- `docs/spec/api/internal-api-contract-design.md`
- `docs/spec/runtime/runtime-execution-logging-architecture.md`
