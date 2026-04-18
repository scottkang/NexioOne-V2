# NexioOne API DTO Baseline

## 1. 목적
- 본 문서는 우선 구현 API의 DTO 구조를 타입 수준으로 고정한다.
- 대상 범위는 `deployment`, `dry-run`, `execute-stub`, `runtime 조회` API다.
- Control Plane CRUD/API는 `control-plane-api-baseline.md`를 우선 기준으로 사용한다.

## 2. 공통 DTO

### 2.1 ApiErrorResponse
```json
{
  "timestamp": "2026-04-18T09:00:00Z",
  "status": 422,
  "error": "Unprocessable Entity",
  "code": "BE-2001",
  "message": "invalid flow definition",
  "path": "/api/projects/1/runtime/flows/10/dry-run",
  "details": [
    {
      "field": "nodes[0].type",
      "reason": "unsupported node type"
    }
  ],
  "traceId": "trace-20260418-0001"
}
```

Field Rules:
- `timestamp`: ISO-8601 UTC 문자열
- `status`: HTTP status code
- `error`: HTTP reason phrase
- `code`: 전역 에러 코드
- `message`: 사용자/운영자가 읽을 수 있는 단문
- `path`: 요청 path
- `details`: optional array
- `traceId`: optional string

### 2.2 ReferenceStatusDto
```json
{
  "id": "101",
  "status": "CREATED",
  "createdAt": "2026-04-18T09:00:00Z",
  "updatedAt": "2026-04-18T09:05:00Z"
}
```

## 3. Deployment DTO

### 3.1 CreateDeploymentRequest
```json
{
  "flowId": 10,
  "version": "v1",
  "description": "initial deployment",
  "requestedBy": "owner@nexioone.local"
}
```

Field Rules:
- `flowId`: required, positive integer
- `version`: required, 1..64 chars
- `description`: optional, 0..500 chars
- `requestedBy`: required, 1..120 chars

### 3.2 DeploymentResponse
```json
{
  "id": 101,
  "projectId": 1,
  "flowId": 10,
  "version": "v1",
  "status": "CREATED",
  "description": "initial deployment",
  "requestedBy": "owner@nexioone.local",
  "createdAt": "2026-04-18T09:00:00Z",
  "updatedAt": "2026-04-18T09:00:00Z"
}
```

### 3.3 UpdateDeploymentStatusRequest
```json
{
  "status": "DEPLOYED",
  "reason": "approved for runtime sync",
  "requestedBy": "owner@nexioone.local"
}
```

Allowed `status`:
- `CREATED`
- `DEPLOYED`
- `ROLLED_BACK`
- `FAILED`

### 3.4 RollbackDeploymentRequest
```json
{
  "targetDeploymentId": 99,
  "reason": "rollback due to runtime issue",
  "requestedBy": "owner@nexioone.local"
}
```

### 3.5 RollbackDeploymentResponse
```json
{
  "rollbackDeploymentId": 102,
  "sourceDeploymentId": 101,
  "targetDeploymentId": 99,
  "status": "ROLLBACK_REQUESTED",
  "requestedAt": "2026-04-18T09:20:00Z"
}
```

## 4. Runtime Read DTO

### 4.1 RuntimeSkeletonResponse
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

### 4.2 SchedulerStatusResponse
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

### 4.3 TriggerStatusResponse
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

### 4.4 DispatchSummaryResponse
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

## 5. Dry-Run DTO

### 5.1 DryRunRequest
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

Field Rules:
- `deploymentId`: optional, positive integer
- `inputContext`: required object
- `options.trace`: optional boolean, default `false`
- `options.validateOnly`: optional boolean, default `false`

### 5.2 DryRunResponse
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
      "status": "SUCCEEDED"
    }
  ]
}
```

## 6. Execute-Stub DTO

### 6.1 ExecuteStubRequest
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

Field Rules:
- `deploymentId`: optional, positive integer
- `inputContext`: required object
- `options.async`: optional boolean, default `false`
- `options.trace`: optional boolean, default `false`

### 6.2 ExecuteStubSyncResponse
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
      "status": "SUCCEEDED"
    }
  ]
}
```

### 6.3 ExecuteStubAsyncAcceptedResponse
```json
{
  "requestId": "req-20260418-0001",
  "executionId": "stub-20260418-0002",
  "mode": "EXECUTE_STUB",
  "status": "ACCEPTED",
  "statusUrl": "/api/projects/1/runtime/executions/stub-20260418-0002"
}
```

## 7. Execution Status DTO

### 7.1 ExecutionStatusResponse
```json
{
  "executionId": "stub-20260418-0002",
  "mode": "EXECUTE_STUB",
  "status": "RUNNING",
  "requestedAt": "2026-04-18T09:30:00Z",
  "startedAt": "2026-04-18T09:30:01Z",
  "finishedAt": null
}
```

Allowed `status`:
- `ACCEPTED`
- `RUNNING`
- `SUCCEEDED`
- `FAILED`
- `CANCELED`
