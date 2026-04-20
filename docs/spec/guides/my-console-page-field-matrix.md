# my-console Page Field Matrix

**Status**: [Baseline]

## 1. 목적
- `my-console` 주요 페이지에서 어떤 필드를 표시하고, 어떤 필드를 편집하며, 어떤 필드를 읽기 전용으로 처리하는지 고정한다.
- 본 문서는 화면 구현 시 필드 단위 재해석을 줄이기 위한 UI 기준이다.

## 2. 해석 규칙
- `Display`: 기본 화면에서 사용자에게 노출한다.
- `Edit`: 현재 프로그램 범위에서 직접 수정 가능하다.
- `Read Only`: 노출하지만 수정 불가다.
- `Hidden`: API/엔티티에 있더라도 현재 페이지에서는 직접 노출하지 않는다.
- `Required`: 생성/저장 요청 시 필수다.
- `Source`: `Entity`, `DTO`, `Derived`, `UI State` 중 하나로 표기한다.

## 3. 공통 규칙
- 식별자(`id`, `projectId`, `flowId`, `bindingVersion`)는 모두 읽기 전용으로 취급한다.
- `createdAt`, `updatedAt`, `requestedBy` 같은 audit 필드는 상세 또는 보조 패널에서만 읽기 전용으로 노출한다.
- 쓰기 권한이 없어도 조회 페이지는 열 수 있으며, 쓰기 필드는 disabled 또는 hidden 처리한다.
- secret 값은 생성/수정 draft에서만 입력하고 상세 조회에서는 원문을 노출하지 않는다.

## 4. 페이지별 필드 매트릭스

### 4.1 LoginPage
| Field | Source | Display | Edit | Required | Default | Validation / Notes |
|---|---|---|---|---|---|---|
| `username` | DTO | Y | Y | Y | `''` | 공백 trim, 빈 값 불가 |
| `password` | DTO | Y | Y | Y | `''` | 빈 값 불가, 마스킹 입력 |
| `submitError.message` | Derived | Y | N | N | `null` | 로그인 실패 시 인라인 표시 |

### 4.2 ProjectListPage
| Field | Source | Display | Edit | Required | Default | Validation / Notes |
|---|---|---|---|---|---|---|
| `id` | Entity | Y | N | N | - | 테이블 행 key |
| `name` | Entity | Y | Y | Y | `''` | 1~100자, 목록/상세 공통 제목 |
| `description` | Entity | Y | Y | N | `''` | 0~500자 |
| `createdAt` | Entity | Y | N | N | - | 목록 보조 컬럼 |
| `updatedAt` | Entity | Y | N | N | - | 기본 정렬 기준 후보 |
| `page` | UI State | Y | Y | N | `0` | query param 반영 |
| `size` | UI State | Y | Y | N | `20` | 허용 값 `10/20/50` |
| `sort` | UI State | Y | Y | N | `updatedAt,desc` | query param 반영 |

### 4.3 ProjectDetailPage
| Field | Source | Display | Edit | Required | Default | Validation / Notes |
|---|---|---|---|---|---|---|
| `id` | Entity | Y | N | N | - | header/meta |
| `name` | Entity | Y | Y | Y | API 값 | 수정 dialog 또는 inline edit |
| `description` | Entity | Y | Y | N | API 값 | multiline 허용 |
| `createdAt` | Entity | Y | N | N | - | meta panel |
| `updatedAt` | Entity | Y | N | N | - | meta panel |
| `activeTab` | UI State | Y | Y | N | `flows` | route/tab state |

### 4.4 FlowListPage
| Field | Source | Display | Edit | Required | Default | Validation / Notes |
|---|---|---|---|---|---|---|
| `id` | Entity | Y | N | N | - | row key |
| `name` | Entity | Y | Y | Y | `''` | 1~100자 |
| `description` | Entity | Y | Y | N | `''` | 0~500자 |
| `version` | Entity | Y | N | N | - | list/detail 배지 |
| `status` | Entity | Y | N | N | - | badge |
| `createdAt` | Entity | Y | N | N | - | 보조 컬럼 |
| `updatedAt` | Entity | Y | N | N | - | 기본 정렬 후보 |
| `searchText` | UI State | Y | Y | N | `''` | 현재는 client-safe filter 또는 reserved |
| `flowDefinition.nodes` | DTO | N | Y | Y | `[]` | 생성 시 최소 빈 배열 허용 |
| `flowDefinition.edges` | DTO | N | Y | Y | `[]` | 생성 시 최소 빈 배열 허용 |

### 4.5 FlowEditorPage
| Field | Source | Display | Edit | Required | Default | Validation / Notes |
|---|---|---|---|---|---|---|
| `id` | Entity | Y | N | N | - | header/meta |
| `name` | Entity | Y | Y | Y | API 값 | 저장 대상 |
| `description` | Entity | Y | Y | N | API 값 | 저장 대상 |
| `version` | Entity | Y | N | N | - | snapshot/version 표시 |
| `status` | Entity | Y | N | N | - | draft/published badge |
| `nodes[].id` | DTO | Y | Y | Y | generated | canvas 안정 식별자 |
| `nodes[].type` | DTO | Y | Y | Y | none | 지원 node type만 허용 |
| `nodes[].name` | DTO | Y | Y | N | `''` | 표시명 |
| `nodes[].config` | DTO | Y | Y | type-specific | `{}` | inspector 편집 |
| `nodes[].connectionRef` | DTO | Y | Y | conditional | `null` | connector node에서만 사용 |
| `nodes[].position` | UI State | Y | Y | Y | generated | canvas 저장 포함 |
| `edges[].id` | DTO | N | Y | Y | generated | 내부 안정 식별자 |
| `edges[].source` | DTO | Y | Y | Y | none | node id 참조 |
| `edges[].target` | DTO | Y | Y | Y | none | node id 참조 |
| `selectedNodeId` | UI State | Y | Y | N | `null` | inspector 제어 |
| `validationErrors` | Derived | Y | N | N | `[]` | banner/panel 표시 |

### 4.6 DataDefinitionPage
| Field | Source | Display | Edit | Required | Default | Validation / Notes |
|---|---|---|---|---|---|---|
| `id` | Entity | Y | N | N | - | list/detail key |
| `name` | Entity | Y | Y | Y | `''` | 1~100자 |
| `description` | Entity | Y | Y | N | `''` | 0~500자 |
| `schemaJson` | DTO | Y | Y | Y | `{}` | valid JSON object required |
| `metadata.dataFormat` | Derived | Y | Y | N | `null` | `schemaJson.x-nexio` 반영 |
| `metadata.structureType` | Derived | Y | Y | N | `null` | `schemaJson.x-nexio` 반영 |
| `deprecatedType` | Derived | Y | N | N | `null` | 레거시 해석용 참고 필드 |
| `createdAt` | Entity | Y | N | N | - | detail only |
| `updatedAt` | Entity | Y | N | N | - | detail only |

### 4.7 ConnectionPage
| Field | Source | Display | Edit | Required | Default | Validation / Notes |
|---|---|---|---|---|---|---|
| `id` | Entity | Y | N | N | - | row/detail key |
| `name` | Entity | Y | Y | Y | `''` | 1~100자 |
| `type` | Entity | Y | Y | Y | none | 생성 후 변경 비권장 |
| `status` | Entity | Y | N | N | - | health badge |
| `properties` | DTO | Y | Y | Y | `{}` | type-specific 필수 키 검증 |
| `secrets` | DTO | N | Y | conditional | `{}` | write-only, 다시 조회 금지 |
| `secretPresence` | Derived | Y | N | N | `{}` | 키 존재 여부만 표시 |
| `createdAt` | Entity | Y | N | N | - | detail only |
| `updatedAt` | Entity | Y | N | N | - | detail only |

#### Type-Specific Required Fields
| Type | Required Fields | Optional Fields |
|---|---|---|
| `JDBC` | `url`, `username` | `driverClassName`, `schema`, `catalog`, `validationQuery`, `connectionTimeoutMs`, `readOnly` |
| `REST` | `baseUrl` | `authenticationType`, `connectTimeoutMs`, `readTimeoutMs`, `defaultHeaders`, `apiKeyHeaderName` |
| `MQ` | `brokerUrl`, `destinationType`, `destinationName` | `clientId`, `ackMode`, `consumerGroup`, `virtualHost`, `sslEnabled` |
| `SFTP` | `host`, `port`, `username` | `basePath`, `knownHostsPolicy`, `connectTimeoutMs`, `charset` |

### 4.8 FlowBindingPage
| Field | Source | Display | Edit | Required | Default | Validation / Notes |
|---|---|---|---|---|---|---|
| `bindingVersion` | DTO | Y | N | N | - | optimistic concurrency |
| `bindings[].id` | DTO | N | N | N | - | update 추적용 |
| `bindings[].role` | DTO | Y | Y | Y | none | `INPUT`/`OUTPUT`/확장 role |
| `bindings[].bindingKey` | DTO | Y | Y | Y | `''` | role 내 unique |
| `bindings[].displayOrder` | DTO | Y | Y | Y | `0` | 숫자 정렬 |
| `bindings[].required` | DTO | Y | Y | Y | `false` | boolean |
| `bindings[].dataDefinitionId` | DTO | Y | Y | Y | none | select required |
| `bindings[].dataDefinitionName` | Derived | Y | N | N | - | option label |
| `conflictError` | Derived | Y | N | N | `null` | 버전 충돌 시 표시 |

### 4.9 DeploymentPage
| Field | Source | Display | Edit | Required | Default | Validation / Notes |
|---|---|---|---|---|---|---|
| `id` | DTO | Y | N | N | - | deployment key |
| `flowId` | DTO | Y | Y | Y | none | 생성 시 선택 |
| `version` | DTO | Y | N | N | - | 응답 기반 표시 |
| `status` | DTO | Y | Y | Y | API 값 | 상태 전이 규칙 준수 |
| `description` | DTO | Y | Y | N | `''` | 생성/상태 변경 메모 |
| `requestedBy` | DTO | Y | N | N | - | audit |
| `createdAt` | DTO | Y | N | N | - | audit |
| `updatedAt` | DTO | Y | N | N | - | audit |

### 4.10 RuntimeOpsPage
| Field | Source | Display | Edit | Required | Default | Validation / Notes |
|---|---|---|---|---|---|---|
| `skeleton` | DTO | Y | N | N | `null` | summary cards |
| `schedulerStatus` | DTO | Y | N | N | `null` | pollable |
| `triggerStatus` | DTO | Y | N | N | `null` | pollable |
| `dispatchSummary` | DTO | Y | N | N | `null` | pollable |
| `executionStatus` | DTO | Y | N | N | `null` | selected execution only |
| `polling` | UI State | Y | Y | N | `true` | auto refresh toggle |

### 4.11 ExecutePage
| Field | Source | Display | Edit | Required | Default | Validation / Notes |
|---|---|---|---|---|---|---|
| `deploymentId` | DTO | Y | Y | N | `null` | 없으면 latest or manual mode |
| `inputContext` | DTO | Y | Y | Y | `{}` | valid JSON object |
| `options.trace` | DTO | Y | Y | N | `true` | 결과 trace 표시 여부 |
| `options.validateOnly` | DTO | Y | Y | N | `false` | dry-run only |
| `options.async` | DTO | Y | Y | N | `true` | execute-stub only |
| `idempotencyKey` | DTO | Y | Y | N | `''` | execute-stub only |
| `lastRequestMode` | UI State | Y | N | N | `null` | `DRY_RUN`/`EXECUTE_STUB` |
| `syncResult` | DTO | Y | N | N | `null` | dry-run immediate result |
| `asyncAccepted` | DTO | Y | N | N | `null` | executionId/requestId |
| `executionStatus` | DTO | Y | N | N | `null` | async poll result |
| `traceItems` | Derived | Y | N | N | `[]` | 단계별 render model |
| `outputPreview` | Derived | Y | N | N | `null` | JSON preview |

## 5. 참조
- `docs/spec/guides/my-console-page-entity-view-model-spec.md`
- `docs/spec/guides/my-console-page-state-transition-spec.md`
- `docs/spec/guides/my-console-routing-ia-spec.md`
- `docs/spec/api/control-plane-api-baseline.md`
- `docs/spec/api/api-spec.md`
