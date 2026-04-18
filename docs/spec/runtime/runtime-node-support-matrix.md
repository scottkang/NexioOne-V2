# Runtime Node Support Matrix

**Status**: [Baseline]

## 1. 목적
- 이번 프로그램에서 runtime이 실제로 지원하는 노드와 차기 범위 노드를 구분한다.

## 2. 이번 프로그램 지원 노드
| Node Type | Support Level | Notes |
|---|---|---|
| `START` | `DRY_RUN`, `EXECUTE_STUB` | 필수 시작 노드 |
| `END` | `DRY_RUN`, `EXECUTE_STUB` | 최소 1개 필요 |
| `MAPPING` | `EXECUTE_STUB` only | 실제 매핑 엔진 대신 config 검증 + stub result |
| `REST_CLIENT` | `EXECUTE_STUB` only | 실제 HTTP 호출 없음 |
| `SQL_EXECUTOR` | `EXECUTE_STUB` only | 실제 DB 호출 없음 |

## 3. 차기 범위 노드
| Node Type | Current State | Notes |
|---|---|---|
| `SQL_BATCH_EXECUTOR` | Not Supported | 차기 범위 |
| `FILE_INPUT` | Not Supported | 차기 범위 |
| `FILE_OUTPUT` | Not Supported | 차기 범위 |
| `FILE_VALIDATOR` | Not Supported | 차기 범위 |
| `FORMAT_CONVERTER` | Not Supported | 차기 범위 |
| `RECORD_EXTRACT` | Not Supported | 차기 범위 |
| `WAIT` | Not Supported | 차기 범위 |
| `SPLIT` | Not Supported | 차기 범위 |
| `JOIN` | Not Supported | 차기 범위 |
| `FLOW_TASK` | Not Supported | 차기 범위 |

## 4. Semantics
- `DRY_RUN`
  - flow 구조와 입력 payload를 검증한다.
  - node 실행 결과는 trace 중심으로 반환한다.
- `EXECUTE_STUB`
  - 지원 노드에 대해 stub step 결과를 생성한다.
  - 외부 시스템 연결은 수행하지 않는다.
- `REAL_CONNECTOR`
  - 이번 프로그램 제외

## 5. Node Contract Baseline
| Node Type | Required Config | Stub Output Shape | Failure Rules | Retry |
|---|---|---|---|---|
| `START` | 없음 | 현재 `inputContext`를 그대로 다음 step 입력으로 전달 | 시작 노드 중복/누락 시 `422` + `BE-2102` | N |
| `END` | 없음 | 현재 컨텍스트를 최종 `outputContext`로 확정 | 종료 노드 누락 시 `422` + `BE-2102` | N |
| `MAPPING` | `mappings` array | `{ "mapped": true, "appliedRules": <count> }`를 step result metadata에 기록, 컨텍스트는 merge 결과 반환 | `mappings` 누락/빈 배열이면 `422` + `BE-2001` | N |
| `REST_CLIENT` | `method`, `path` | `{ "statusCode": 200, "stubbed": true }`를 `response` key에 기록 | `method`, `path` 누락이면 `422` + `BE-2001` | Y |
| `SQL_EXECUTOR` | `statementType`, `sql` | `{ "rowsAffected": 1, "stubbed": true }`를 `sqlResult` key에 기록 | `statementType`, `sql` 누락이면 `422` + `BE-2001` | Y |

### 5.1 Additional Rules
- `MAPPING`의 컨텍스트 merge 충돌은 마지막 rule 우선으로 처리한다.
- `REST_CLIENT`와 `SQL_EXECUTOR`의 retry는 이번 프로그램에서 자동 재시도가 아니라 "재시도 가능 오류 분류" 의미만 가진다.
- step trace가 활성화되면 각 node는 최소 `nodeId`, `nodeType`, `status`, `startedAt`, `finishedAt`를 남긴다.
- `DRY_RUN`은 `START`, `END`만 trace 대상이며 `outputContext`는 `inputContext`를 그대로 반환한다.
- `EXECUTE_STUB`은 지원 노드 전체를 step으로 기록하며 `MAPPING` merge 결과와 connector stub result를 `outputContext`에 반영한다.

## 6. Fixture Mapping
| Node Type | Happy Path Fixture | Validation/Error Fixture |
|---|---|---|
| `START`, `END` | `fixtures/runtime/nodes/start-end-dry-run-success.json` | 구조 오류는 `BE-2102`로 처리 |
| `MAPPING` | `fixtures/runtime/nodes/mapping-node-stub-success.json` | `mappings` 누락/빈 배열 시 `BE-2001` |
| `REST_CLIENT` | `fixtures/runtime/nodes/rest-client-node-stub-success.json` | `fixtures/runtime/nodes/rest-client-node-validation-error.json` |
| `SQL_EXECUTOR` | `fixtures/runtime/nodes/sql-executor-node-stub-success.json` | `fixtures/runtime/nodes/sql-executor-node-validation-error.json` |
| Unsupported Node | `fixtures/runtime/flow-definitions/unsupported-node-flow-definition.json` | `fixtures/api/runtime/dry-run-unsupported-node-response.json` |

## 7. Validation Rules
- `START`는 정확히 1개
- `END`는 최소 1개
- 미지원 노드 포함 시 `422` + `BE-2102`
- 순환 구조는 이번 프로그램에서 불허

## 8. 참조
- `docs/spec/api/internal-execution-schema.md`
- `docs/spec/modules/my-backend.md`
