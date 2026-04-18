# Functional DoD Matrix

**Status**: [Baseline]

## 1. 목적
- 이번 프로그램 범위 기능에 대해 구현 완료 판정 기준을 기능 단위로 고정한다.

## 2. 공통 DoD
- 코드가 메인 브랜치 기준으로 빌드 가능하다.
- API/이벤트/DTO 문서가 구현과 일치한다.
- 오류 응답이 `error-response-baseline.md`를 따른다.
- 권한 체크가 `security-authority-matrix.md`와 연결된다.
- 테스트가 최소 기준을 충족한다.

## 3. 기능별 DoD
| Feature | Code | Test | Docs | Evidence |
|---|---|---|---|---|
| Auth Login/Refresh | 로그인/토큰 갱신 API 구현 | auth success/failure API test | `control-plane-api-baseline.md` 반영 | `fixtures/api/auth/auth-login-happy-response.json`, `fixtures/api/auth/auth-login-invalid-credential-response.json`, `fixtures/api/auth/auth-refresh-happy-response.json` |
| Project CRUD | 목록/생성/조회/수정/삭제 구현 | API + service test | API 계약 반영 | `fixtures/api/projects/project-list-happy-response.json`, `fixtures/api/projects/project-create-happy-response.json`, `fixtures/api/projects/project-detail-not-found-response.json` |
| Flow CRUD | flowDefinition 저장/조회/수정/삭제 구현 | API + validation test | API 계약 반영 | `fixtures/api/flows/flow-create-happy-response.json`, `fixtures/api/flows/flow-update-invalid-definition-response.json` |
| DataDefinition CRUD | schema 저장/조회/수정/삭제 구현 | API + repository test | API 계약 반영 | `fixtures/api/data-definitions/data-definition-create-happy-response.json`, `fixtures/api/data-definitions/data-definition-create-validation-error-response.json` |
| Connection CRUD | secret write-only 처리 포함 CRUD 구현 | API + masking test | API 계약 반영 | `fixtures/api/connections/connection-detail-masked-response.json`, `fixtures/api/connections/connection-create-invalid-response.json` |
| Flow Data Binding | binding 목록/저장 구현 | API + relation validation test | 역할 모델 문서 반영 | `fixtures/api/flows/flow-binding-save-happy-response.json` |
| Deployment API | 생성/상태변경/롤백 구현 | API + service test | `api-spec.md` 반영 | `fixtures/api/deployments/deployment-create-happy-response.json`, `fixtures/api/deployments/deployment-status-conflict-response.json`, `fixtures/api/deployments/deployment-rollback-happy-response.json` |
| Dry-Run | runtime 호출 orchestration + 결과 반환 | API + integration test | `api-spec.md` 반영 | `fixtures/api/runtime/dry-run-happy-response.json`, `fixtures/api/runtime/dry-run-unsupported-node-response.json` |
| Execute-Stub Sync | 동기 실행 요청/응답 구현 | API + integration test | `api-spec.md` 반영 | `fixtures/api/runtime/execute-stub-sync-happy-response.json`, `fixtures/runtime/nodes/rest-client-node-validation-error.json` |
| Execute-Stub Async | 비동기 접수/상태 조회/idempotency 구현 | API + integration + contract test | 내부 실행 계약 반영 | `fixtures/api/runtime/execute-stub-async-accepted-response.json`, `fixtures/api/runtime/execute-stub-idempotency-conflict-response.json`, `fixtures/api/runtime/execution-status-running-response.json` |
| Runtime Status Read | skeleton/scheduler/trigger/dispatch/execution 조회 구현 | API test | `api-spec.md`, `runtime-operations-spec.md` 반영 | `fixtures/api/runtime/execution-status-running-response.json` |
| Runtime Event Publish | started/step/completed/failed 이벤트 발행 | producer contract test | `runtime-event-schema.md` 반영 | `fixtures/events/runtime/execution-started-event.json`, `fixtures/events/runtime/execution-step-completed-event.json`, `fixtures/events/runtime/execution-completed-event.json`, `fixtures/events/runtime/execution-failed-event.json` |
| Logging Consume/Persist | event consumer + read model 저장 구현 | consumer contract + persistence test | logging 아키텍처 반영 | `fixtures/events/runtime/execution-completed-event.json`, `fixtures/events/runtime/execution-failed-event.json` |

## 4. 최소 테스트 기준
### 4.1 API
- happy path
- validation error
- auth error
- permission error
- not found

### 4.2 Runtime
- dry-run success
- unsupported node type
- execute-stub sync success
- execute-stub async accepted/status
- idempotency key conflict

### 4.3 Event
- producer schema validation
- consumer idempotency
- persistence failure handling

## 5. 증적 형식
- API fixture JSON
- contract test 결과
- integration test 결과
- 필요 시 운영 화면 캡처 또는 로그 스니펫

### 5.1 공통 Seed 기준
- fixture 간 공통 식별자는 `fixtures/seeds/control-plane-minimal-seed.json`을 기준으로 맞춘다.
- control-plane, runtime, event fixture는 동일한 `projectId=1`, `flowId=10`, `deploymentId=101` 세트를 기본값으로 사용한다.

## 6. 참조
- `docs/spec/foundation/release-scope.md`
- `docs/spec/api/control-plane-api-baseline.md`
- `docs/spec/quality/test-fixture-baseline.md`
- `docs/spec/api/api-spec.md`
- `docs/spec/runtime/runtime-event-schema.md`
- `docs/spec/foundation/traceability-spec.md`
