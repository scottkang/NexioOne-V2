# NexioOne Traceability Spec

## 1. 목적
- 구현 예정 기능이 어떤 문서, API 계약, 테스트, 증적과 연결되는지 추적 가능하게 정의한다.
- 현재 구현은 없으므로 본 문서는 완료 판정이 아니라 준비 상태 추적 기준으로 사용한다.

## 2. 추적 단위
- Requirement
- API Contract
- Module
- Test
- Evidence

## 3. 현재 판정
- 상태: `Planning Baseline`
- 근거:
  - 제품 범위, Control Plane API, runtime/event 계약, 기능별 DoD를 기준선으로 정리한 단계
  - 구현 코드, 테스트, 운영 증적은 아직 존재하지 않는 것으로 가정

## 4. 판정 규칙
- 개발 착수 가능:
  - 범위 문서
  - 외부 API 계약
  - 내부 실행 계약
  - 테스트 전략
  - 운영 목표 규격
  - 위 항목이 서로 연결되어 있어야 함
- 구현 완료:
  - 기능 코드 + 테스트 + 문서 + 증적이 모두 존재해야 함

## 5. 우선 추적 대상
| Requirement | Contract | Module | Test | Evidence |
|---|---|---|---|---|
| Auth Login/Refresh | `docs/spec/api/control-plane-api-baseline.md` | `my-console-backend` | API/Auth Test | fixture JSON |
| Project CRUD | `docs/spec/api/control-plane-api-baseline.md` | `my-console-backend` | API/Service Test | fixture JSON |
| Flow CRUD | `docs/spec/api/control-plane-api-baseline.md` | `my-console-backend` | API/Validation Test | invalid flow fixture |
| DataDefinition CRUD | `docs/spec/api/control-plane-api-baseline.md` | `my-console-backend` | API/Repository Test | schema fixture |
| Connection CRUD | `docs/spec/api/control-plane-api-baseline.md` | `my-console-backend` | API/Masking Test | masked response fixture |
| Flow Data Binding | `docs/spec/api/control-plane-api-baseline.md`, `docs/spec/data/flow-owned-data-definition-roles.md` | `my-console-backend` | API/Relation Validation Test | binding fixture |
| Connection Profile Contract | `docs/spec/data/connection-profile-contract.md` | `my-console-backend`, `my-backend` | API/Contract Test | jdbc/rest fixture |
| Internal ExecuteRequest Contract | `docs/spec/api/internal-api-contract-design.md`, `docs/spec/api/internal-execution-schema.md` | `my-console-backend`, `my-backend` | Contract Test | `execute-request-with-flow-bindings.json` |
| Deployment 생성/상태변경/롤백 | `docs/spec/api/api-spec.md` | `my-console-backend` | API/Service/Contract Test | `deployment-create.json`, `deployment-status-transition.json`, `deployment-rollback.json` |
| Flow Dry-Run | `docs/spec/api/api-spec.md` | `my-console-backend`, `my-backend` | API/Integration Test | `dry-run-success.json`, `dry-run-invalid-flow.json` |
| Flow Execute-Stub | `docs/spec/api/api-spec.md`, `docs/spec/api/internal-api-contract-design.md`, `docs/spec/runtime/runtime-node-support-matrix.md` | `my-console-backend`, `my-backend` | API/Integration/Contract Test | `execute-stub-sync-success.json`, `execute-stub-async-accepted.json`, `execute-stub-idempotency-conflict.json`, `mapping-node-stub-success.json` |
| Runtime Status 조회 | `docs/spec/api/api-spec.md`, `docs/spec/runtime/runtime-operations-spec.md` | `my-console-backend`, `my-backend` | API Test | `runtime-skeleton.json`, `runtime-execution-running.json` |
| Execution Event 전달 | `docs/spec/api/internal-api-contract-design.md`, `docs/spec/runtime/runtime-event-schema.md` | `my-backend`, `logging-service` | Producer Contract/Consumer Test | `event-started.json`, `event-step-completed.json`, `event-completed.json`, `event-failed.json` |

## 6. 참조
- `docs/spec/foundation/product-spec.md`
- `docs/spec/foundation/release-scope.md`
- `docs/spec/foundation/development-readiness-checklist.md`
- `docs/spec/api/control-plane-api-baseline.md`
- `docs/spec/data/connection-profile-contract.md`
- `docs/spec/api/api-spec.md`
- `docs/spec/api/internal-api-contract-design.md`
- `docs/spec/foundation/functional-dod-matrix.md`
- `docs/spec/quality/test-fixture-baseline.md`
- `docs/spec/quality/qa-testing-strategy.md`
