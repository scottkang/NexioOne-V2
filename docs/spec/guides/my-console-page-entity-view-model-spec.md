# my-console Page Entity & View-Model Spec

**Status**: [Baseline]

## 1. 목적
- `my-console` 구현에 필요한 페이지별 엔티티, API DTO, 프론트 view-model, 주요 컴포넌트 prop 계약을 고정한다.
- 본 문서는 "백엔드 계약 필드"와 "프론트 전용 조합 필드"를 구분해 재작업 없는 병렬 개발 기준을 제공한다.

## 2. 해석 규칙
- `Entity`:
  - API 계약 또는 저장 모델에 직접 대응되는 핵심 비즈니스 필드 묶음
- `DTO`:
  - 외부 API request/response shape
- `View-Model`:
  - 페이지 렌더링과 폼/테이블/패널 제어를 위해 프론트에서 조합하는 상태
- `Derived`:
  - API 원문이 아니라 프론트에서 계산하는 표시 전용 필드

프론트 구현 원칙:
- 가능한 한 API DTO를 그대로 받되, 페이지 렌더링 직전에 명시적 view-model로 변환한다.
- dirty, selectedRowIds, isDialogOpen 같은 UI 전용 상태는 DTO에 섞지 않는다.
- 한 페이지 안에서도 `list item view-model`, `detail view-model`, `form draft`를 분리한다.

## 3. 공통 페이지 계약

### 3.1 AppShellViewModel
| Field | Type | Source | Editable | Notes |
|---|---|---|---|---|
| `accessToken` | string | auth response | N | 메모리/secure storage |
| `refreshToken` | string | auth response | N | secure storage |
| `authorities` | string[] | auth response | N | RBAC gate |
| `projectScopes` | number[] | JWT claims | N | project access filter |
| `selectedProjectId` | number \| null | UI state | Y | last selected project |
| `sidebarCollapsed` | boolean | UI state | Y | layout only |
| `globalLoading` | boolean | UI state | N | route transition / bootstrap |

### 3.2 ApiErrorViewModel
| Field | Type | Source | Editable | Notes |
|---|---|---|---|---|
| `status` | number | `ApiErrorResponse` | N | HTTP status |
| `code` | string | `ApiErrorResponse` | N | domain/system code |
| `message` | string | `ApiErrorResponse` | N | banner/toast text |
| `fieldErrors` | `{ field: string; reason: string }[]` | `details` | N | form binding |
| `traceId` | string \| null | `traceId` | N | operator support |

## 4. 페이지 인벤토리
| Page | Route | Primary Entity | Primary APIs |
|---|---|---|---|
| LoginPage | `/login` | AuthSession | `POST /api/auth/login`, `POST /api/auth/refresh` |
| ProjectListPage | `/projects` | ProjectSummary | `GET /api/projects`, `POST /api/projects` |
| ProjectDetailPage | `/projects/:projectId` | ProjectDetail | `GET /api/projects/{projectId}`, `PUT`, `DELETE` |
| FlowListPage | `/projects/:projectId/flows` | FlowSummary | `GET /flows`, `POST /flows` |
| FlowEditorPage | `/projects/:projectId/flows/:flowId` | FlowDetail + FlowDefinitionDraft | `GET/PUT /flows/{flowId}` |
| DataDefinitionPage | `/projects/:projectId/data-definitions` | DataDefinitionSummary | `GET/POST/PUT/DELETE /data-definitions*` |
| ConnectionPage | `/projects/:projectId/connections` | ConnectionSummary + ConnectionDraft | `GET/POST/PUT/DELETE /connections*` |
| FlowBindingPage | `/projects/:projectId/flows/:flowId/data-bindings` | FlowBindingSet | `GET/PUT /data-bindings` |
| DeploymentPage | `/projects/:projectId/deployments` | DeploymentItem | `POST /deployments`, `PATCH /status`, `POST /rollback` |
| RuntimeOpsPage | `/projects/:projectId/runtime` | RuntimeOverview | skeleton/status APIs |
| ExecutePage | `/projects/:projectId/runtime/flows/:flowId/execute` | ExecuteRequestDraft | dry-run / execute-stub APIs |

## 5. 페이지별 엔티티 및 View-Model

### 5.1 LoginPage
#### DTO
- `LoginRequest`
  - `username`
  - `password`
- `LoginResponse`
  - `accessToken`
  - `refreshToken`
  - `tokenType`
  - `expiresIn`
  - `authorities`

#### View-Model
| Field | Type | Source | Editable | Notes |
|---|---|---|---|---|
| `username` | string | form draft | Y | input |
| `password` | string | form draft | Y | password input |
| `isSubmitting` | boolean | UI state | N | disable submit |
| `submitError` | ApiErrorViewModel \| null | error mapping | N | inline error |

### 5.2 ProjectListPage
#### Entity
- `ProjectSummary`
  - `id`
  - `name`
  - `description`
  - `createdAt`
  - `updatedAt`

#### View-Model
| Field | Type | Source | Editable | Notes |
|---|---|---|---|---|
| `items` | `ProjectSummary[]` | API response | N | table rows |
| `page` | number | API/query state | Y | list query |
| `size` | number | API/query state | Y | list query |
| `sort` | string | API/query state | Y | list query |
| `selectedProjectId` | number \| null | UI state | Y | row navigation |
| `createDialogOpen` | boolean | UI state | Y | modal control |
| `createDraft` | `ProjectFormDraft` | local state | Y | create form |

#### Form Draft
- `ProjectFormDraft`
  - `name`
  - `description`

### 5.3 ProjectDetailPage
#### Entity
- `ProjectDetail` = `ProjectSummary`

#### View-Model
| Field | Type | Source | Editable | Notes |
|---|---|---|---|---|
| `project` | `ProjectDetail` | API response | N | header card |
| `activeTab` | `'flows' \| 'data-definitions' \| 'connections' \| 'deployments' \| 'runtime'` | route/ui state | Y | tab state |
| `canEdit` | boolean | authorities + scope | N | action gate |
| `deleteDialogOpen` | boolean | UI state | Y | destructive confirm |

### 5.4 FlowListPage
#### Entity
- `FlowSummary`
  - `id`
  - `projectId`
  - `name`
  - `description`
  - `version`
  - `status`
  - `createdAt`
  - `updatedAt`

#### View-Model
| Field | Type | Source | Editable | Notes |
|---|---|---|---|---|
| `items` | `FlowSummary[]` | API response | N | list rows |
| `searchText` | string | UI/query state | Y | future filter-safe |
| `createDraft` | `FlowCreateDraft` | local state | Y | modal/page form |
| `selectedFlowId` | number \| null | UI state | Y | detail route |

#### Form Draft
- `FlowCreateDraft`
  - `name`
  - `description`
  - `flowDefinition.nodes`
  - `flowDefinition.edges`

### 5.5 FlowEditorPage
#### Entity
- `FlowDetail`
  - `id`
  - `projectId`
  - `name`
  - `description`
  - `version`
  - `status`
  - `flowDefinition`

#### FlowDefinitionDraft
| Field | Type | Source | Editable | Notes |
|---|---|---|---|---|
| `name` | string | API response / form | Y | metadata |
| `description` | string | API response / form | Y | metadata |
| `nodes` | `FlowNodeVM[]` | transformed `flowDefinition.nodes` | Y | canvas state |
| `edges` | `FlowEdgeVM[]` | transformed `flowDefinition.edges` | Y | canvas state |
| `selectedNodeId` | string \| null | UI state | Y | inspector |
| `hasUnsavedChanges` | boolean | derived | N | dirty guard |
| `validationErrors` | `FlowValidationIssue[]` | local validate + API | N | banner/panel |

#### FlowNodeVM
| Field | Type | Source | Editable | Notes |
|---|---|---|---|---|
| `id` | string | entity | Y | stable in canvas |
| `type` | string | entity | Y | node catalog type |
| `name` | string | entity/display | Y | optional label |
| `config` | object | entity | Y | inspector form |
| `connectionRef` | string \| null | entity | Y | connector node only |
| `position` | `{ x: number; y: number }` | UI state | Y | canvas-only |
| `statusBadge` | string \| null | derived | N | validation/runtime badge |

#### Core Components
- `FlowCanvasProps`
  - `nodes`
  - `edges`
  - `selectedNodeId`
  - `onNodesChange`
  - `onEdgesChange`
  - `onSelectNode`
- `NodeInspectorProps`
  - `node`
  - `availableConnections`
  - `validationErrors`
  - `onChange`
  - `onClose`

### 5.6 DataDefinitionPage
#### Entity
- `DataDefinitionSummary`
  - `id`
  - `projectId`
  - `name`
  - `description`
  - `createdAt`
  - `updatedAt`

- `DataDefinitionDetail`
  - `id`
  - `projectId`
  - `name`
  - `description`
  - `schemaJson`
  - `createdAt`
  - `updatedAt`

#### Form Draft
| Field | Type | Source | Editable | Notes |
|---|---|---|---|---|
| `name` | string | form | Y | required |
| `description` | string | form | Y | optional |
| `schemaJson` | object | form/editor | Y | required |
| `metadata.dataFormat` | string \| null | `schemaJson.x-nexio` | Y | derived editor field |
| `metadata.structureType` | string \| null | `schemaJson.x-nexio` | Y | derived editor field |
| `deprecatedType` | string \| null | legacy field interpretation | N | reference only, not editable |

#### Key Components
- `DataDefinitionTableProps`
  - `items`
  - `selectedId`
  - `onSelect`
  - `onCreate`
  - `onDelete`
- `SchemaEditorFormProps`
  - `draft`
  - `errors`
  - `onChange`
  - `onSubmit`

### 5.7 ConnectionPage
#### Entity
- `ConnectionSummary`
  - `id`
  - `projectId`
  - `name`
  - `type`
  - `status`
  - `createdAt`
  - `updatedAt`

- `ConnectionDetail`
  - `id`
  - `projectId`
  - `name`
  - `type`
  - `properties`
  - `status`
  - `createdAt`
  - `updatedAt`

#### ConnectionDraft
| Field | Type | Source | Editable | Notes |
|---|---|---|---|---|
| `name` | string | form | Y | required |
| `type` | `'JDBC' \| 'REST' \| 'MQ' \| 'SFTP'` | form | Y | creation-time fixed preferred |
| `properties` | object | form | Y | type-specific |
| `secrets` | object | form | Y | write-only |
| `status` | string \| null | API response | N | detail only |
| `secretPresence` | `{ [key: string]: boolean }` | derived | N | UI placeholder |

#### Type-Specific Form Fields
- `JDBC`: `url`, `username`, `driverClassName?`, `schema?`, `catalog?`, `validationQuery?`, `connectionTimeoutMs?`, `readOnly?`
- `REST`: `baseUrl`, `authenticationType?`, `connectTimeoutMs?`, `readTimeoutMs?`, `defaultHeaders?`, `apiKeyHeaderName?`
- `MQ`: `brokerUrl`, `destinationType`, `destinationName`, `clientId?`, `ackMode?`, `consumerGroup?`, `virtualHost?`, `sslEnabled?`
- `SFTP`: `host`, `port`, `username`, `basePath?`, `knownHostsPolicy?`, `connectTimeoutMs?`, `charset?`

### 5.8 FlowBindingPage
#### Entity
- `FlowBindingItem`
  - `id`
  - `role`
  - `bindingKey`
  - `displayOrder`
  - `required`
  - `dataDefinitionId`
  - `dataDefinitionName`

- `FlowBindingSet`
  - `bindingVersion`
  - `bindings`
  - `updatedAt`

#### View-Model
| Field | Type | Source | Editable | Notes |
|---|---|---|---|---|
| `bindingVersion` | number | API response | N | optimistic concurrency |
| `bindings` | `FlowBindingDraft[]` | API response / local form | Y | full-set replace |
| `availableDefinitions` | `DataDefinitionSummary[]` | API response | N | select options |
| `conflictError` | ApiErrorViewModel \| null | error mapping | N | version conflict |

#### Binding Draft
- `id?`
- `role`
- `bindingKey`
- `displayOrder`
- `required`
- `dataDefinitionId`

### 5.9 DeploymentPage
#### Entity
- `DeploymentResponse`
  - `id`
  - `projectId`
  - `flowId`
  - `version`
  - `status`
  - `description?`
  - `requestedBy`
  - `createdAt`
  - `updatedAt`

#### View-Model
| Field | Type | Source | Editable | Notes |
|---|---|---|---|---|
| `items` | `DeploymentResponse[]` | API response / future list | N | deployment rows |
| `createDraft` | `CreateDeploymentDraft` | local form | Y | create modal |
| `statusPatchDraft` | `UpdateDeploymentStatusDraft` | local form | Y | inline dialog |
| `rollbackDraft` | `RollbackDeploymentDraft` | local form | Y | rollback dialog |
| `statusTransitionMap` | `{ [status: string]: string[] }` | derived from spec | N | enable buttons |

### 5.10 RuntimeOpsPage
#### Entity / DTO
- `RuntimeSkeletonResponse`
- `SchedulerStatusResponse`
- `TriggerStatusResponse`
- `DispatchSummaryResponse`
- `ExecutionStatusResponse`

#### View-Model
| Field | Type | Source | Editable | Notes |
|---|---|---|---|---|
| `skeleton` | object | API response | N | summary cards |
| `schedulerStatus` | object | API response | N | scheduler card |
| `triggerStatus` | object | API response | N | trigger card |
| `dispatchSummary` | object | API response | N | dispatch card |
| `executionStatus` | object \| null | API response | N | poll target |
| `polling` | boolean | UI state | Y | status refresh loop |

### 5.11 ExecutePage
#### Drafts
- `DryRunDraft`
  - `deploymentId?`
  - `inputContext`
  - `options.trace`
  - `options.validateOnly`

- `ExecuteStubDraft`
  - `deploymentId?`
  - `inputContext`
  - `options.async`
  - `options.trace`
  - `idempotencyKey?`

#### Result View-Model
| Field | Type | Source | Editable | Notes |
|---|---|---|---|---|
| `lastRequestMode` | `'DRY_RUN' \| 'EXECUTE_STUB' \| null` | UI state | N | result routing |
| `syncResult` | object \| null | API response | N | immediate result |
| `asyncAccepted` | object \| null | API response | N | requestId/executionId |
| `executionStatus` | object \| null | API response | N | polled result |
| `traceItems` | object[] | derived from response | N | step/trace renderer |
| `outputPreview` | object \| null | derived | N | JSON viewer |

## 6. 공통 컴포넌트 Prop 계약

### 6.1 EntityTableProps<T>
- `rows: T[]`
- `loading: boolean`
- `emptyTitle: string`
- `emptyDescription?: string`
- `selectedRowId?: string | number | null`
- `onSelectRow?: (row: T) => void`
- `onCreate?: () => void`
- `onRefresh?: () => void`

### 6.2 CrudFormDialogProps<TDraft>
- `open: boolean`
- `mode: 'create' | 'edit'`
- `draft: TDraft`
- `errors: ApiErrorViewModel | null`
- `submitting: boolean`
- `onChange: (next: TDraft) => void`
- `onSubmit: () => void`
- `onClose: () => void`

### 6.3 JsonPreviewPanelProps
- `title: string`
- `value: unknown`
- `emptyText?: string`
- `collapsed?: boolean`
- `onToggleCollapsed?: () => void`

### 6.4 PermissionGateProps
- `permission: string`
- `projectId?: number`
- `fallback?: ReactNode`
- `children: ReactNode`

## 7. 구현 시작 순서
1. `AppShellViewModel`, `ApiErrorViewModel`, `PermissionGateProps`
2. `ProjectListPage`, `ProjectDetailPage`
3. `FlowListPage`, `FlowEditorPage`
4. `DataDefinitionPage`, `ConnectionPage`, `FlowBindingPage`
5. `DeploymentPage`
6. `RuntimeOpsPage`, `ExecutePage`

## 8. 참조
- `docs/spec/modules/my-console.md`
- `docs/spec/modules/my-console-detailed-design.md`
- `docs/spec/guides/frontend-development-guide.md`
- `docs/spec/guides/ui-component-spec.md`
- `docs/spec/api/control-plane-api-baseline.md`
- `docs/spec/api/api-spec.md`
- `docs/spec/api/api-dto-baseline.md`
