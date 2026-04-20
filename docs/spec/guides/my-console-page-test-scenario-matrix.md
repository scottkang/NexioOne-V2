# my-console Page Test Scenario Matrix

**Status**: [Baseline]

## 1. 목적
- `my-console` 페이지별로 구현 직후 최소 검증해야 하는 시나리오를 고정한다.

## 2. 공통 시나리오 집합
- happy path
- validation error
- forbidden
- not-found
- conflict
- loading / empty

## 3. 페이지별 시나리오
| Page | Required Scenarios |
|---|---|
| LoginPage | login success, invalid credential, expired token redirect |
| ProjectListPage | list success, empty state, create success, create validation error, delete confirm |
| ProjectDetailPage | detail success, `403`, `404`, edit success |
| FlowListPage | list success, empty state, create success, create validation error |
| FlowEditorPage | load existing flow, local validation fail, server validation fail, dirty confirm, unsupported node read |
| DataDefinitionPage | list/select success, schema parse error, create validation error, delete confirm |
| ConnectionPage | type switch rendering, required field validation, secret preserved on edit, forbidden write |
| FlowBindingPage | load/save success, duplicate key local validation, `409` bindingVersion conflict |
| DeploymentPage | create success, invalid transition, rollback success, conflict handling |
| RuntimeOpsPage | aggregate load success, section refresh, execution not found |
| ExecutePage | dry-run success, execute-stub sync success, execute-stub async accepted, idempotency conflict, invalid inputContext |

## 4. 우선 자동화 후보
| Priority | Area | Notes |
|---|---|---|
| P0 | Login, Project CRUD, FlowEditor local validation, Execute happy/conflict | 구현 초기 회귀 방지 |
| P1 | DataDefinition JSON parse, Connection type forms, Binding conflict | 계약 리스크 높은 영역 |
| P2 | Runtime read-only polling, Deployment transitions | 운영성 확인 |

## 5. Fixture / Mock Mapping
| Scenario | Suggested Source |
|---|---|
| Dry-run success | `fixtures/api/runtime/dry-run-happy-response.json` |
| Unsupported node | `fixtures/api/runtime/dry-run-unsupported-node-response.json` |
| Execute async accepted | `fixtures/api/runtime/execute-stub-async-accepted-response.json` |
| Idempotency conflict | `fixtures/api/runtime/execute-stub-idempotency-conflict-response.json` |
| Binding conflict | `fixtures/api/flows/flow-binding-version-conflict-response.json` |
| DataDefinition validation | `fixtures/api/data-definitions/data-definition-create-validation-error-response.json` |
| REST node validation | `fixtures/runtime/nodes/rest-client-node-validation-error.json` |
| SQL node validation | `fixtures/runtime/nodes/sql-executor-node-validation-error.json` |

## 6. Definition of Done for Page Implementation
- 지정된 happy path와 주요 오류 시나리오가 수동 또는 자동 테스트로 검증된다.
- 권한이 없는 경우 숨김/disable/read-only 규칙이 문서대로 동작한다.
- loading/empty/error 상태가 명시적으로 렌더링된다.

## 7. 참조
- `docs/spec/guides/my-console-page-state-transition-spec.md`
- `docs/spec/quality/test-fixture-baseline.md`
- `docs/spec/foundation/functional-dod-matrix.md`
