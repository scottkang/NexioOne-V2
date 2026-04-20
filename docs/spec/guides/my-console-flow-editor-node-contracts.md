# my-console Flow Editor Node Contracts

**Status**: [Baseline]

## 1. 목적
- 현재 프로그램 범위에서 Flow Editor가 지원해야 하는 node/edge/inspector 세부 계약을 고정한다.

## 2. 지원 노드
| Node Type | Canvas Category | Runtime Support | Required Config |
|---|---|---|---|
| `START` | control | `DRY_RUN`, `EXECUTE_STUB` | none |
| `END` | control | `DRY_RUN`, `EXECUTE_STUB` | none |
| `MAPPING` | transform | `EXECUTE_STUB` | `mappings` |
| `REST_CLIENT` | connector | `EXECUTE_STUB` | `method`, `path`, `connectionRef?` |
| `SQL_EXECUTOR` | connector | `EXECUTE_STUB` | `statementType`, `sql`, `connectionRef` |

## 3. 미지원 노드
- `SQL_BATCH_EXECUTOR`
- `FILE_INPUT`
- `FILE_OUTPUT`
- `FILE_VALIDATOR`
- `FORMAT_CONVERTER`
- `RECORD_EXTRACT`
- `WAIT`
- `SPLIT`
- `JOIN`
- `FLOW_TASK`

규칙:
- 미지원 노드는 카탈로그에 노출하지 않는다.
- 기존 flow에 포함된 미지원 노드를 읽은 경우 read-only badge + validation error로 표시한다.

## 4. Node View-Model
| Field | Rules |
|---|---|
| `id` | canvas/session 내 stable |
| `type` | 지원 node type만 허용 |
| `name` | optional, 비어 있으면 `displayName(type)` 사용 |
| `config` | type별 schema 준수 |
| `connectionRef` | connector node에서만 사용 |
| `position` | `{x,y}` persisted |
| `statusBadge` | validation/runtime derived |

## 5. Inspector Field Rules

### 5.1 START
- editable field 없음
- 설명 패널만 노출

### 5.2 END
- editable field 없음
- 현재 output binding / 종료 의미만 설명

### 5.3 MAPPING
| Field | Type | Validation | UI |
|---|---|---|---|
| `mappings` | array | 1개 이상 required | row repeater |
| `mappings[].source` | string | required | text input |
| `mappings[].target` | string | required | text input |
| `mappings[].expression` | string | optional | textarea/code editor |

### 5.4 REST_CLIENT
| Field | Type | Validation | UI |
|---|---|---|---|
| `method` | enum | required | select |
| `path` | string | required, 1~500자 | text input |
| `connectionRef` | string/null | optional | connection selector |
| `timeoutMs` | integer | optional, 1~120000 | number input |
| `headers` | object | optional | JSON editor |

### 5.5 SQL_EXECUTOR
| Field | Type | Validation | UI |
|---|---|---|---|
| `statementType` | enum | required | select |
| `sql` | string | required, 1~4000자 | code editor |
| `connectionRef` | string | required | JDBC connection selector |
| `parameters` | object | optional | JSON editor |

## 6. Edge Rules
- `START` incoming edge 금지
- `END` outgoing edge 금지
- cycle 금지
- self-loop 금지
- source/target node는 모두 존재해야 한다.

## 7. Local Validation Rules
- `START` 정확히 1개
- `END` 1개 이상
- 미지원 node 포함 금지
- connector node의 `connectionRef`는 선택 가능한 connection과 일치해야 함
- Flow save 전 local validation을 먼저 수행하고, server validation 결과와 병합 표시

## 8. Result/Trace Interpretation
- `DRY_RUN`: `START`, `END` 위주 trace
- `EXECUTE_STUB`: 지원 node 전체 trace
- node badge는 `SUCCEEDED`, `FAILED`, `SKIPPED`, `NOT_SUPPORTED`를 우선 사용

## 9. 참조
- `docs/spec/runtime/runtime-node-support-matrix.md`
- `docs/spec/runtime/node-catalog-schema.md`
- `docs/spec/api/internal-execution-schema.md`
