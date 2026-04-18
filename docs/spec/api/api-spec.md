# NexioOne API Spec (External API Baseline: Deployment + Runtime)

## 1. 목적
- 본 문서는 이번 프로그램 외부 API 기준선 중 `deployment`, `dry-run`, `execute-stub`, `runtime 조회` 영역을 상세화한다.
- 본 문서는 `control-plane-api-baseline.md`와 함께 외부 API 단일 계약 묶음을 구성한다.
- 이번 프로그램의 인증과 Control Plane CRUD API는 후속 범위가 아니라 현재 범위이며, 기준 문서는 `control-plane-api-baseline.md`로 고정한다.
- 상세 DTO는 `api-dto-baseline.md`를 함께 기준으로 사용한다.
- 오류 응답은 `error-response-baseline.md`를 우선 기준으로 사용한다.

## 1.1 외부 API 계약 묶음
| 계약 영역 | 기준 문서 | 비고 |
|---|---|---|
| 인증 / Project / Flow / DataDefinition / Connection CRUD | `control-plane-api-baseline.md` | Control Plane CRUD 단일 기준 |
| Deployment / Dry-Run / Execute-Stub / Runtime 조회 | `api-spec.md` | 본 문서 |
| 공통 DTO | `api-dto-baseline.md` | 필드명/응답 shape 기준 |
| 공통 오류 응답 | `error-response-baseline.md` | `ApiErrorResponse` 단일 기준 |

## 1.2 해석 규칙
- 외부 API 전체 범위 판정은 `release-scope.md`를 따른다.
- Control Plane CRUD와 Deployment/Runtime API는 같은 이번 프로그램 범위에 속한다.
- 새 외부 API 부속 문서를 만들더라도, 위 표의 기준 문서를 대체하거나 충돌시키면 안 된다.

## 2. 공통 규칙

### 2.1 인증
- 모든 API는 Bearer JWT를 사용한다.
- 권한 코드는 `docs/spec/security/security-authority-matrix.md`를 기준으로 연결한다.

### 2.1.1 권한 매핑
- Deployment 조회 계열: `DEPLOYMENT_READ`
- Deployment 변경 계열: `DEPLOYMENT_WRITE`
- Runtime 조회 계열: `RUNTIME_READ`
- Dry-Run, Execute-Stub, 재실행 계열: `RUNTIME_EXECUTE`

### 2.2 Content-Type
- Request: `application/json`
- Response: `application/json`

### 2.3 시간/식별자 형식
- `id`: `number` 또는 `string` 식별자
- `occurredAt`, `createdAt`, `updatedAt`: ISO-8601 UTC 문자열

### 2.4 공통 에러 응답
```json
{
  "timestamp": "2026-04-18T09:00:00Z",
  "status": 422,
  "error": "Unprocessable Entity",
  "code": "SYS-1002",
  "message": "validation failed",
  "path": "/api/projects/1/runtime/flows/10/dry-run",
  "details": [
    {
      "field": "inputContext",
      "reason": "must be object"
    }
  ],
  "traceId": "trace-20260418-0001"
}
```

### 2.5 공통 에러 코드
- `401`: `SYS-1100`
- `403`: `SYS-1101`
- `404`: `SYS-1001`
- `409`: `SYS-1409`, `ADM-2001`, `ADM-2002`, `BE-2103`
- `422`: `SYS-1002`, `BE-2001`, `BE-2102`
- `500`: `SYS-0001`
- `503`: `SYS-1503`

## 3. 이번 프로그램 외부 API 범위
- 인증 API: `control-plane-api-baseline.md`
- Project / Flow / DataDefinition / Connection CRUD: `control-plane-api-baseline.md`
- Deployment API: 본 문서
- Runtime Read API: 본 문서
- Dry-Run API: 본 문서
- Execute-Stub API: 본 문서
- Execution Status API: 본 문서

## 4. Deployment API

### 3.1 배포 생성
- `POST /api/projects/{projectId}/deployments`
- 권한: `DEPLOYMENT_WRITE`

Request:
```json
{
  "flowId": 10,
  "version": "v1",
  "description": "initial deployment",
  "requestedBy": "owner@nexioone.local"
}
```

Response `201`:
```json
{
  "id": 101,
  "projectId": 1,
  "flowId": 10,
  "version": "v1",
  "status": "CREATED",
  "requestedBy": "owner@nexioone.local",
  "createdAt": "2026-04-18T09:00:00Z"
}
```

Validation:
- `flowId` 필수
- `version` 필수, 프로젝트 내 유일
- `requestedBy` 필수

### 3.2 배포 상태 변경
- `PATCH /api/projects/{projectId}/deployments/{deploymentId}/status`
- 권한: `DEPLOYMENT_WRITE`

Request:
```json
{
  "status": "DEPLOYED",
  "reason": "approved for runtime sync",
  "requestedBy": "owner@nexioone.local"
}
```

Response `200`:
```json
{
  "id": 101,
  "status": "DEPLOYED",
  "reason": "approved for runtime sync",
  "updatedAt": "2026-04-18T09:10:00Z"
}
```

Status Rules:
- 허용 상태: `CREATED`, `DEPLOYED`, `ROLLED_BACK`, `FAILED`
- 불가능한 전이 시 `409`

### 3.3 배포 롤백
- `POST /api/projects/{projectId}/deployments/{deploymentId}/rollback`
- 권한: `DEPLOYMENT_WRITE`

Request:
```json
{
  "targetDeploymentId": 99,
  "reason": "rollback due to runtime issue",
  "requestedBy": "owner@nexioone.local"
}
```

Response `202`:
```json
{
  "rollbackDeploymentId": 102,
  "sourceDeploymentId": 101,
  "targetDeploymentId": 99,
  "status": "ROLLBACK_REQUESTED",
  "requestedAt": "2026-04-18T09:20:00Z"
}
```

## 5. Runtime Read API

### 4.1 Runtime Skeleton 조회
- `GET /api/projects/{projectId}/runtime/skeleton`
- 권한: `RUNTIME_READ`

Response `200`:
```json
{
  "projectId": 1,
  "runtimeMode": "STUB",
  "workerConfigured": false,
  "availableApis": [
    "dry-run",
    "execute-stub",
    "status"
  ]
}
```

### 4.2 Scheduler 상태 조회
- `GET /api/projects/{projectId}/runtime/scheduler/status`
- 권한: `RUNTIME_READ`

Response `200`:
```json
{
  "projectId": 1,
  "enabled": false,
  "lastSyncAt": null,
  "policies": [],
  "summary": {
    "totalPolicies": 0,
    "activePolicies": 0,
    "failedPolicies": 0
  }
}
```

### 4.3 Trigger 상태 조회
- `GET /api/projects/{projectId}/runtime/triggers/status`
- 권한: `RUNTIME_READ`

Response `200`:
```json
{
  "projectId": 1,
  "enabled": false,
  "channels": [
    {
      "type": "WEBHOOK",
      "status": "NOT_CONFIGURED",
      "activePolicies": 0
    }
  ]
}
```

### 4.4 Dispatch Summary 조회
- `GET /api/projects/{projectId}/runtime/dispatch/summary`
- 권한: `RUNTIME_READ`

Response `200`:
```json
{
  "projectId": 1,
  "window": "PT24H",
  "counts": {
    "requested": 0,
    "succeeded": 0,
    "failed": 0
  },
  "lastDispatchAt": null
}
```

## 6. Dry-Run API

### 5.1 Flow Dry-Run 실행
- `POST /api/projects/{projectId}/runtime/flows/{flowId}/dry-run`
- 권한: `RUNTIME_EXECUTE`

Request:
```json
{
  "deploymentId": 101,
  "inputContext": {
    "customerId": "C100"
  },
  "options": {
    "trace": true,
    "validateOnly": false
  }
}
```

Response `200`:
```json
{
  "executionId": "dryrun-20260418-0001",
  "flowId": 10,
  "mode": "DRY_RUN",
  "status": "SUCCEEDED",
  "validationErrors": [],
  "outputContext": {
    "customerId": "C100"
  },
  "trace": [
    {
      "nodeId": "start",
      "nodeType": "START",
      "status": "SUCCEEDED",
      "startedAt": "2026-04-18T09:30:00Z",
      "finishedAt": "2026-04-18T09:30:00Z"
    },
    {
      "nodeId": "end",
      "nodeType": "END",
      "status": "SUCCEEDED",
      "startedAt": "2026-04-18T09:30:00Z",
      "finishedAt": "2026-04-18T09:30:00Z"
    }
  ]
}
```

Fixture:
- `fixtures/api/runtime/dry-run-happy-request.json`
- `fixtures/api/runtime/dry-run-happy-response.json`
- `fixtures/api/runtime/dry-run-unsupported-node-response.json`

Validation:
- `inputContext`는 object
- `options.trace`, `options.validateOnly`는 boolean
- Flow 미존재 시 `404`
- Flow 구조 오류 시 `422`
- `DRY_RUN`은 `START`, `END`만 실행 의미론 대상으로 간주하며 `outputContext`는 `inputContext`를 그대로 반환한다.

## 7. Execute-Stub API

### 6.1 Stub 실행
- `POST /api/projects/{projectId}/runtime/flows/{flowId}/execute-stub`
- 권한: `RUNTIME_EXECUTE`

Request:
```json
{
  "deploymentId": 101,
  "inputContext": {
    "customerId": "C100"
  },
  "options": {
    "async": false,
    "trace": true
  }
}
```

Sync Response `200`:
```json
{
  "executionId": "stub-20260418-0001",
  "flowId": 10,
  "mode": "EXECUTE_STUB",
  "status": "SUCCEEDED",
  "outputContext": {
    "customerId": "C100"
  },
  "steps": [
    {
      "nodeId": "start",
      "nodeType": "START",
      "status": "SUCCEEDED",
      "startedAt": "2026-04-18T09:32:00Z",
      "finishedAt": "2026-04-18T09:32:00Z"
    }
  ]
}
```

Async Response `202`:
```json
{
  "requestId": "req-20260418-0001",
  "executionId": "stub-20260418-0002",
  "mode": "EXECUTE_STUB",
  "status": "ACCEPTED",
  "statusUrl": "/api/projects/1/runtime/executions/stub-20260418-0002",
  "statusOwner": "my-backend"
}
```

Fixture:
- `fixtures/api/runtime/execute-stub-sync-happy-request.json`
- `fixtures/api/runtime/execute-stub-sync-happy-response.json`
- `fixtures/api/runtime/execute-stub-async-accepted-request.json`
- `fixtures/api/runtime/execute-stub-async-accepted-response.json`
- `fixtures/api/runtime/execute-stub-idempotency-conflict-response.json`
- `fixtures/api/runtime/execution-status-running-response.json`

Validation:
- `options.async` 미지정 시 `false`
- 비동기 접수 후 중복 실행 요청 충돌 시 `409`
- `EXECUTE_STUB` step은 최소 `nodeId`, `nodeType`, `status`, `startedAt`, `finishedAt`를 포함한다.
- `MAPPING`은 merge 결과를 `outputContext`에 반영하고, `REST_CLIENT`, `SQL_EXECUTOR`는 stub result를 `outputContext` 하위 key에 기록한다.

## 8. 우선 고정 대상 조회 API

### 7.1 실행 상태 조회
- `GET /api/projects/{projectId}/runtime/executions/{executionId}`
- 권한: `RUNTIME_READ`
- 정본: `my-backend`
- 목적: 비동기 실행 제어 상태 조회
- 해석 규칙:
  - 이 API는 실행 제어용 단건 상태 조회다.
  - 사용자용 장기 이력/검색 API는 본 문서 범위가 아니며, 차후 `my-console-backend -> logging-service` read model 경로로 별도 고정한다.
  - `my-console-backend`는 이 경로에서 `my-backend` 정본 상태를 중계하거나 동등 의미로 반환해야 한다.

Response `200`:
```json
{
  "executionId": "stub-20260418-0002",
  "mode": "EXECUTE_STUB",
  "status": "RUNNING",
  "requestedAt": "2026-04-18T09:30:00Z",
  "startedAt": "2026-04-18T09:30:01Z",
  "finishedAt": null,
  "statusOwner": "my-backend"
}
```

Fixture:
- `fixtures/api/runtime/execution-status-running-response.json`

상태 규칙:
- `ACCEPTED`, `RUNNING`은 runtime이 직접 반환하는 진행 상태다.
- `SUCCEEDED`, `FAILED`, `CANCELED`는 최종 상태다.
- 이 엔드포인트는 목록/검색/장기 이력 API를 대체하지 않는다.
- 존재하지 않는 `executionId`는 `404`
- scope 밖 프로젝트 접근은 `403`

## 9. 구현 순서 권고
1. Deployment API
2. Runtime 조회 API
3. Dry-Run API
4. Execute-Stub API
5. 실행 상태 조회 API

## 10. 참조
- `docs/spec/api/control-plane-api-baseline.md`
- `docs/spec/api/api-dto-baseline.md`
- `docs/spec/security/security-authority-matrix.md`
- `docs/spec/api/global-error-code-map.md`
- `docs/spec/api/error-response-baseline.md`
- `docs/spec/foundation/release-scope.md`
