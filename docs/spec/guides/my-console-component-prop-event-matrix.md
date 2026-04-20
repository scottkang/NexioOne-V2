# my-console Component Prop Event Matrix

**Status**: [Baseline]

## 1. 목적
- 공통 및 핵심 도메인 컴포넌트의 prop, default, event 발생 시점, 부모/자식 책임을 표로 고정한다.

## 2. 공통 규칙
- 부모는 서버 상태와 side effect를 소유한다.
- 자식은 렌더링과 사용자 의도 이벤트만 발행한다.
- `onChange`는 partial patch가 아니라 next state 전체 전달을 우선 기준으로 한다.

## 3. Matrix
| Component | Key Props | Defaults | Events | Parent Responsibility | Child Responsibility |
|---|---|---|---|---|---|
| `AppShell` | `projectOptions`, `selectedProjectId`, `breadcrumbs`, `authorities` | `sidebarCollapsed=false` | `onSelectProject`, `onToggleSidebar`, `onLogout` | auth/session, route nav | shell layout render |
| `EntityTable<T>` | `rows`, `columns`, `sort`, `pagination`, `loading`, `error` | `loading=false` | `onSelectRow`, `onSortChange`, `onPageChange`, `onRefresh`, `onCreate` | query state, row action side effect | table render, empty/loading UI |
| `CrudFormDialog<T>` | `open`, `mode`, `draft`, `errors`, `dirty`, `submitting` | `dirty=false` | `onChange`, `onSubmit`, `onClose` | submit/cancel policy | form render, local input binding |
| `JsonEditor` | `value`, `readOnly`, `schemaHint`, `parseError` | `readOnly=false` | `onChange` | JSON parse result ownership | editor render and parse feedback |
| `PermissionGate` | `permission`, `projectId`, `mode` | `mode='hide'` | none | authority lookup | conditional render |
| `FlowCanvas` | `nodes`, `edges`, `selectedNodeId`, `readOnly` | `readOnly=false` | `onNodesChange`, `onEdgesChange`, `onSelectNode` | persist/validate graph | graph manipulation UI |
| `NodeInspector` | `node`, `availableConnections`, `validationErrors`, `readOnly` | `node=null` | `onChange`, `onClose` | selected node ownership | node form render |
| `BindingEditorTable` | `items`, `availableDefinitions`, `bindingVersion`, `readOnly`, `conflictError` | `readOnly=false` | `onChange`, `onSubmit` | save orchestration | row editing UI |
| `ExecuteRequestForm` | `mode`, `draft`, `submitting`, `errors` | `mode='DRY_RUN'` | `onChange`, `onSubmit` | request orchestration | mode-specific form render |
| `JsonPreviewPanel` | `title`, `value`, `collapsed` | `collapsed=false` | `onToggleCollapsed` | preview source | viewer render |

## 4. Event Timing Rules
- `onChange`: input commit 즉시
- `onSubmit`: 사용자가 명시적으로 submit action 수행할 때만
- `onClose`: dismiss, cancel, overlay close 시
- `onSelectRow`: row click 또는 keyboard selection 시
- `onNodesChange`/`onEdgesChange`: drag/connect/delete 완료 시

## 5. 참조
- `docs/spec/guides/my-console-component-contracts.md`
- `docs/spec/guides/my-console-table-and-form-behavior-spec.md`
