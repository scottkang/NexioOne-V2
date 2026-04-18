# Test Fixture Baseline

**Status**: [Baseline]

## 1. 목적
- API, runtime, event contract 테스트와 증적에 사용할 fixture 종류와 최소 세트를 고정한다.

## 2. Fixture 분류

### 2.1 API Fixture
- HTTP request/response 샘플
- happy path, validation error, auth error, permission error, not found 포함

### 2.2 Runtime Fixture
- `dry-run`, `execute-stub`, `execution status` 샘플
- 지원 노드/미지원 노드 flowDefinition 샘플
- 내부 실행 payload의 `projectId`, `deploymentId`, `requestedBy`, `flowBindings` 포함 샘플
- node별 required config, stub output, validation error 샘플

### 2.3 Event Fixture
- `execution.started`
- `execution.step.completed`
- `execution.completed`
- `execution.failed`

### 2.4 Data Fixture
- project / flow / dataDefinition / connection seed 데이터

## 3. 권장 디렉터리 구조
```text
fixtures/
  api/
    auth/
    projects/
    flows/
    data-definitions/
    connections/
    deployments/
    runtime/
  runtime/
    flow-definitions/
    input-contexts/
    nodes/
  events/
    runtime/
  seeds/
```

## 4. 파일 naming 규칙
- `{feature}-{scenario}-{direction}.json`
- 예:
  - `project-create-happy-request.json`
  - `project-create-happy-response.json`
  - `execute-stub-idempotency-conflict-response.json`
  - `execution-completed-event.json`

## 5. 최소 Fixture 세트

### 5.1 Auth
- `auth-login-happy-request/response`
- `auth-login-invalid-credential-response`
- `auth-refresh-happy-request/response`

현재 baseline 경로:
- `fixtures/api/auth/auth-login-happy-request.json`
- `fixtures/api/auth/auth-login-happy-response.json`
- `fixtures/api/auth/auth-login-invalid-credential-response.json`
- `fixtures/api/auth/auth-refresh-happy-request.json`
- `fixtures/api/auth/auth-refresh-happy-response.json`

### 5.2 Project
- `project-list-happy-response`
- `project-create-happy-request/response`
- `project-detail-not-found-response`

현재 baseline 경로:
- `fixtures/api/projects/project-list-happy-response.json`
- `fixtures/api/projects/project-create-happy-request.json`
- `fixtures/api/projects/project-create-happy-response.json`
- `fixtures/api/projects/project-detail-not-found-response.json`

### 5.3 Flow
- `flow-create-happy-request/response`
- `flow-update-invalid-definition-response`
- `flow-binding-save-happy-request/response`
- `flow-binding-version-conflict-response`

현재 baseline 경로:
- `fixtures/api/flows/flow-create-happy-request.json`
- `fixtures/api/flows/flow-create-happy-response.json`
- `fixtures/api/flows/flow-update-invalid-definition-response.json`
- `fixtures/api/flows/flow-binding-save-happy-request.json`
- `fixtures/api/flows/flow-binding-save-happy-response.json`
- `fixtures/api/flows/flow-binding-version-conflict-response.json`

### 5.4 DataDefinition
- `data-definition-create-happy-request/response`
- `data-definition-create-validation-error-response`

현재 baseline 경로:
- `fixtures/api/data-definitions/data-definition-create-happy-request.json`
- `fixtures/api/data-definitions/data-definition-create-happy-response.json`
- `fixtures/api/data-definitions/data-definition-create-validation-error-response.json`

### 5.5 Connection
- `connection-create-jdbc-happy-request/response`
- `connection-create-rest-bearer-happy-request/response`
- `connection-create-mq-happy-request/response`
- `connection-create-sftp-happy-request/response`
- `connection-detail-masked-response`
- `connection-create-invalid-response`

현재 baseline 경로:
- `fixtures/api/connections/connection-create-jdbc-happy-request.json`
- `fixtures/api/connections/connection-create-jdbc-happy-response.json`
- `fixtures/api/connections/connection-create-rest-bearer-happy-request.json`
- `fixtures/api/connections/connection-create-rest-bearer-happy-response.json`
- `fixtures/api/connections/connection-create-mq-happy-request.json`
- `fixtures/api/connections/connection-create-mq-happy-response.json`
- `fixtures/api/connections/connection-create-sftp-happy-request.json`
- `fixtures/api/connections/connection-create-sftp-happy-response.json`
- `fixtures/api/connections/connection-detail-masked-response.json`
- `fixtures/api/connections/connection-create-invalid-response.json`
- `fixtures/api/connections/connection-create-mq-invalid-response.json`
- `fixtures/api/connections/connection-create-sftp-invalid-response.json`

### 5.6 Deployment / Runtime
- `deployment-create-happy-request/response`
- `deployment-status-conflict-response`
- `deployment-rollback-happy-request/response`
- `dry-run-happy-request/response`
- `dry-run-unsupported-node-response`
- `execute-stub-sync-happy-request/response`
- `execute-stub-async-accepted-request/response`
- `execute-stub-idempotency-conflict-response`
- `execution-status-running-response`
- `execute-request-with-flow-bindings.json`
- `mapping-node-stub-success.json`
- `rest-client-node-stub-success.json`
- `rest-client-node-validation-error.json`
- `sql-executor-node-stub-success.json`
- `sql-executor-node-validation-error.json`

현재 baseline 경로:
- API deployment/runtime:
  - `fixtures/api/deployments/deployment-create-happy-request.json`
  - `fixtures/api/deployments/deployment-create-happy-response.json`
  - `fixtures/api/deployments/deployment-status-conflict-response.json`
  - `fixtures/api/deployments/deployment-rollback-happy-request.json`
  - `fixtures/api/deployments/deployment-rollback-happy-response.json`
  - `fixtures/api/runtime/dry-run-happy-request.json`
  - `fixtures/api/runtime/dry-run-happy-response.json`
  - `fixtures/api/runtime/dry-run-unsupported-node-response.json`
  - `fixtures/api/runtime/execute-stub-sync-happy-request.json`
  - `fixtures/api/runtime/execute-stub-sync-happy-response.json`
  - `fixtures/api/runtime/execute-stub-async-accepted-request.json`
  - `fixtures/api/runtime/execute-stub-async-accepted-response.json`
  - `fixtures/api/runtime/execute-stub-idempotency-conflict-response.json`
  - `fixtures/api/runtime/execution-status-running-response.json`
- Runtime contract:
  - `fixtures/runtime/connection-profiles-mixed.json`
  - `fixtures/runtime/execute-request-with-flow-bindings.json`
  - `fixtures/runtime/flow-binding-snapshot-versioned.json`
  - `fixtures/runtime/flow-definitions/unsupported-node-flow-definition.json`
  - `fixtures/runtime/nodes/start-end-dry-run-success.json`
  - `fixtures/runtime/nodes/mapping-node-stub-success.json`
  - `fixtures/runtime/nodes/rest-client-node-stub-success.json`
  - `fixtures/runtime/nodes/rest-client-node-validation-error.json`
  - `fixtures/runtime/nodes/sql-executor-node-stub-success.json`
  - `fixtures/runtime/nodes/sql-executor-node-validation-error.json`

### 5.7 Events
- `execution-started-event`
- `execution-step-completed-event`
- `execution-completed-event`
- `execution-failed-event`

현재 baseline 경로:
- `fixtures/events/runtime/execution-started-event.json`
- `fixtures/events/runtime/execution-step-completed-event.json`
- `fixtures/events/runtime/execution-completed-event.json`
- `fixtures/events/runtime/execution-failed-event.json`

### 5.8 Seeds
- `control-plane-minimal-seed`

현재 baseline 경로:
- `fixtures/seeds/control-plane-minimal-seed.json`

## 6. 사용 규칙
- 문서 예시, contract test, integration test에서 같은 fixture를 재사용한다.
- fixture의 `code`, `status`, `eventType`, `schemaVersion`은 기준 문서와 일치해야 한다.
- secret fixture는 실제 비밀값 대신 dummy 값을 사용한다.
- masked response fixture에는 secret 원문이 없어야 한다.

## 7. 검증 포인트
- API fixture:
  - request/response field 존재 여부
  - error code / HTTP status 일치
- runtime fixture:
  - 지원 노드와 미지원 노드 구분
  - `DRY_RUN`은 `START`, `END` trace만 남기고 `inputContext`를 그대로 `outputContext`로 반환
  - `EXECUTE_STUB`은 지원 노드 step 결과와 `outputContext` 조립 규칙을 남김
  - idempotency conflict 응답 일치
  - `flowBindings` snapshot 포함 여부
  - `connectionProfiles` 유형별 필수 field / `secretsRef` key 일치
  - node별 stub output shape 일치
- event fixture:
  - envelope 구조
  - routing key 대응 eventType
  - 필수 payload 필드 존재
  - `deploymentId`, `nodeType`, `startedAt/finishedAt` 존재 여부
- seed fixture:
  - 기능별 fixture가 참조하는 최소 프로젝트/flow/dataDefinition/connection/deployment 식별자 일치

## 8. 증적 기준
- fixture는 테스트 입력/출력과 동일한 구조여야 한다.
- CI contract test 결과가 해당 fixture 이름과 매핑 가능해야 한다.
- 문서 예시는 fixture에서 파생하는 것을 원칙으로 한다.
- 내부 실행 fixture는 외부 API request body가 아니라 `my-console-backend -> my-backend` 전달 payload 기준으로도 보관한다.
- control-plane fixture와 runtime/event fixture는 공통 seed 식별자 세트를 재사용한다.

## 9. 참조
- `docs/spec/api/control-plane-api-baseline.md`
- `docs/spec/api/api-spec.md`
- `docs/spec/api/error-response-baseline.md`
- `docs/spec/runtime/runtime-event-schema.md`
- `docs/spec/foundation/functional-dod-matrix.md`
