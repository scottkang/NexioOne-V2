# my-console Page Presentation Attributes Spec

**Status**: [Baseline]

## 1. 목적
- 각 페이지가 어떤 속성을 어디에서 어떻게 표현하는지, 어떤 입력 컴포넌트를 쓰는지, 저장 payload에 포함되는지, dirty 판정 대상인지 고정한다.

## 2. 해석 규칙
- `Surface`: 속성이 노출되는 위치
  - `list-column`
  - `header`
  - `detail-panel`
  - `form-section`
  - `inspector`
  - `banner`
  - `result-panel`
- `Widget`: 표현/입력 방식
  - `text`
  - `textarea`
  - `badge`
  - `date`
  - `number`
  - `select`
  - `switch`
  - `json-editor`
  - `code-editor`
  - `hidden`
- `Payload`: 저장/실행 요청에 포함 여부
- `Dirty`: 값 변경 시 dirty 판정 대상 여부

## 3. 공통 규칙
- 식별자, audit, 파생 badge는 기본적으로 `Payload=N`, `Dirty=N`이다.
- `Derived` 속성은 화면 표시 전용이므로 요청 payload에 포함하지 않는다.
- `UI State`는 렌더링 제어용이며 저장 payload에 포함하지 않는다.
- secret 계열 필드는 입력 시에만 payload에 포함하고, 미입력 시 기존 값 유지로 해석한다.

## 4. 페이지별 표현 속성

### 4.1 LoginPage
| Field | Surface | Widget | Source | Payload | Dirty | Notes |
|---|---|---|---|---|---|---|
| `username` | `form-section` | `text` | DTO | Y | Y | autofocus 대상 |
| `password` | `form-section` | `text` | DTO | Y | Y | password mask |
| `submitError.message` | `banner` | `text` | Derived | N | N | 실패 시만 표시 |

### 4.2 ProjectListPage
| Field | Surface | Widget | Source | Payload | Dirty | Notes |
|---|---|---|---|---|---|---|
| `id` | `list-column` | `text` | Entity | N | N | 행 key, 숨김 가능 |
| `name` | `list-column` | `text` | Entity | Y | Y | primary column |
| `description` | `list-column` | `text` | Entity | Y | Y | secondary column |
| `updatedAt` | `list-column` | `date` | Entity | N | N | default sort |
| `createdAt` | `list-column` | `date` | Entity | N | N | optional column |
| `createDialogOpen` | `hidden` | `hidden` | UI State | N | N | modal control |
| `createDraft.name` | `form-section` | `text` | UI State | Y | Y | create dialog |
| `createDraft.description` | `form-section` | `textarea` | UI State | Y | Y | create dialog |

### 4.3 ProjectDetailPage
| Field | Surface | Widget | Source | Payload | Dirty | Notes |
|---|---|---|---|---|---|---|
| `project.name` | `header` | `text` | Entity | Y | Y | title/edit field |
| `project.description` | `detail-panel` | `textarea` | Entity | Y | Y | summary/edit field |
| `project.id` | `detail-panel` | `text` | Entity | N | N | read-only meta |
| `project.createdAt` | `detail-panel` | `date` | Entity | N | N | read-only meta |
| `project.updatedAt` | `detail-panel` | `date` | Entity | N | N | read-only meta |
| `activeTab` | `header` | `select` | UI State | N | N | route-driven |

### 4.4 FlowListPage
| Field | Surface | Widget | Source | Payload | Dirty | Notes |
|---|---|---|---|---|---|---|
| `name` | `list-column` | `text` | Entity | Y | Y | primary column |
| `description` | `list-column` | `text` | Entity | Y | Y | secondary |
| `status` | `list-column` | `badge` | Entity | N | N | read-only |
| `version` | `list-column` | `badge` | Entity | N | N | read-only |
| `updatedAt` | `list-column` | `date` | Entity | N | N | default sort |
| `searchText` | `header` | `text` | UI State | N | N | query state |
| `createDraft.name` | `form-section` | `text` | UI State | Y | Y | create dialog |
| `createDraft.description` | `form-section` | `textarea` | UI State | Y | Y | create dialog |
| `createDraft.flowDefinition.nodes` | `hidden` | `hidden` | UI State | Y | Y | create payload |
| `createDraft.flowDefinition.edges` | `hidden` | `hidden` | UI State | Y | Y | create payload |

### 4.5 FlowEditorPage
| Field | Surface | Widget | Source | Payload | Dirty | Notes |
|---|---|---|---|---|---|---|
| `name` | `header` | `text` | Entity | Y | Y | title/save payload |
| `description` | `header` | `textarea` | Entity | Y | Y | save payload |
| `status` | `header` | `badge` | Entity | N | N | read-only |
| `version` | `header` | `badge` | Entity | N | N | read-only |
| `nodes[].type` | `inspector` | `select` | DTO | Y | Y | read-only for existing node type change 권장 금지 |
| `nodes[].name` | `inspector` | `text` | DTO | Y | Y | optional label |
| `nodes[].config` | `inspector` | `json-editor` | DTO | Y | Y | type-specific form render 우선 |
| `nodes[].connectionRef` | `inspector` | `select` | DTO | Y | Y | connector node only |
| `nodes[].position` | `hidden` | `hidden` | UI State | Y | Y | persisted canvas state |
| `edges[]` | `hidden` | `hidden` | DTO | Y | Y | canvas graph payload |
| `selectedNodeId` | `inspector` | `hidden` | UI State | N | N | selection control |
| `validationErrors` | `banner` | `text` | Derived | N | N | summary panel |

### 4.6 DataDefinitionPage
| Field | Surface | Widget | Source | Payload | Dirty | Notes |
|---|---|---|---|---|---|---|
| `name` | `form-section` | `text` | Entity | Y | Y | meta form |
| `description` | `form-section` | `textarea` | Entity | Y | Y | meta form |
| `schemaJson` | `form-section` | `json-editor` | DTO | Y | Y | core payload |
| `metadata.dataFormat` | `form-section` | `select` | Derived | Y | Y | `schemaJson.x-nexio` sync |
| `metadata.structureType` | `form-section` | `select` | Derived | Y | Y | `schemaJson.x-nexio` sync |
| `deprecatedType` | `detail-panel` | `text` | Derived | N | N | reference only |
| `createdAt` | `detail-panel` | `date` | Entity | N | N | read-only |
| `updatedAt` | `detail-panel` | `date` | Entity | N | N | read-only |

### 4.7 ConnectionPage
| Field | Surface | Widget | Source | Payload | Dirty | Notes |
|---|---|---|---|---|---|---|
| `name` | `form-section` | `text` | Entity | Y | Y | common field |
| `type` | `form-section` | `select` | Entity | Y | Y | create 시 editable, edit 시 disabled 권장 |
| `status` | `detail-panel` | `badge` | Entity | N | N | read-only |
| `properties` | `form-section` | `json-editor` | DTO | Y | Y | type-specific form 우선 |
| `secrets` | `form-section` | `text` | DTO | conditional | Y | write-only |
| `secretPresence` | `detail-panel` | `badge` | Derived | N | N | configured label |
| `createdAt` | `detail-panel` | `date` | Entity | N | N | read-only |
| `updatedAt` | `detail-panel` | `date` | Entity | N | N | read-only |

#### Connection Type-Specific Presentation
| Field | Surface | Widget | Payload | Dirty | Notes |
|---|---|---|---|---|---|
| `JDBC.url` | `form-section` | `text` | Y | Y | required |
| `JDBC.username` | `form-section` | `text` | Y | Y | required |
| `JDBC.validationQuery` | `form-section` | `textarea` | Y | Y | optional |
| `REST.baseUrl` | `form-section` | `text` | Y | Y | required |
| `REST.defaultHeaders` | `form-section` | `json-editor` | Y | Y | optional |
| `MQ.destinationType` | `form-section` | `select` | Y | Y | required |
| `SFTP.port` | `form-section` | `number` | Y | Y | required |

### 4.8 FlowBindingPage
| Field | Surface | Widget | Source | Payload | Dirty | Notes |
|---|---|---|---|---|---|---|
| `bindingVersion` | `header` | `badge` | DTO | Y | N | request 포함, editable 아님 |
| `bindings[].role` | `form-section` | `select` | DTO | Y | Y | row editor |
| `bindings[].bindingKey` | `form-section` | `text` | DTO | Y | Y | row editor |
| `bindings[].displayOrder` | `form-section` | `number` | DTO | Y | Y | row editor |
| `bindings[].required` | `form-section` | `switch` | DTO | Y | Y | row editor |
| `bindings[].dataDefinitionId` | `form-section` | `select` | DTO | Y | Y | row editor |
| `bindings[].dataDefinitionName` | `form-section` | `text` | Derived | N | N | option label |
| `conflictError` | `banner` | `text` | Derived | N | N | conflict banner |

### 4.9 DeploymentPage
| Field | Surface | Widget | Source | Payload | Dirty | Notes |
|---|---|---|---|---|---|---|
| `flowId` | `list-column` | `text` | DTO | Y | Y | create dialog select |
| `version` | `list-column` | `badge` | DTO | N | N | read-only after create |
| `status` | `list-column` | `badge` | DTO | Y | Y | status dialog payload |
| `description` | `detail-panel` | `textarea` | DTO | Y | Y | create/status change reason |
| `requestedBy` | `list-column` | `text` | DTO | N | N | audit |
| `createdAt` | `list-column` | `date` | DTO | N | N | audit |
| `updatedAt` | `list-column` | `date` | DTO | N | N | audit |

### 4.10 RuntimeOpsPage
| Field | Surface | Widget | Source | Payload | Dirty | Notes |
|---|---|---|---|---|---|---|
| `skeleton` | `detail-panel` | `text` | DTO | N | N | summary cards |
| `schedulerStatus` | `detail-panel` | `badge` | DTO | N | N | status card |
| `triggerStatus` | `detail-panel` | `badge` | DTO | N | N | status card |
| `dispatchSummary` | `detail-panel` | `text` | DTO | N | N | counts card |
| `executionStatus` | `detail-panel` | `text` | DTO | N | N | selected execution |
| `polling` | `header` | `switch` | UI State | N | N | client-only toggle |

### 4.11 ExecutePage
| Field | Surface | Widget | Source | Payload | Dirty | Notes |
|---|---|---|---|---|---|---|
| `deploymentId` | `form-section` | `select` | DTO | Y | Y | optional |
| `inputContext` | `form-section` | `json-editor` | DTO | Y | Y | required |
| `options.trace` | `form-section` | `switch` | DTO | Y | Y | mode common |
| `options.validateOnly` | `form-section` | `switch` | DTO | Y | Y | dry-run only |
| `options.async` | `form-section` | `switch` | DTO | Y | Y | execute-stub only |
| `idempotencyKey` | `form-section` | `text` | DTO | Y | Y | execute-stub only |
| `lastRequestMode` | `header` | `badge` | UI State | N | N | view control |
| `syncResult` | `result-panel` | `json-editor` | DTO | N | N | read-only preview |
| `asyncAccepted` | `result-panel` | `json-editor` | DTO | N | N | read-only preview |
| `executionStatus` | `result-panel` | `json-editor` | DTO | N | N | read-only preview |
| `traceItems` | `result-panel` | `text` | Derived | N | N | timeline render |
| `outputPreview` | `result-panel` | `json-editor` | Derived | N | N | read-only preview |

## 5. 참조
- `docs/spec/guides/my-console-page-field-matrix.md`
- `docs/spec/guides/my-console-page-entity-view-model-spec.md`
- `docs/spec/guides/my-console-page-layout-wire-spec.md`
- `docs/spec/guides/my-console-field-validation-rules.md`
- `docs/spec/guides/my-console-table-and-form-behavior-spec.md`
