# my-console Component Contracts

**Status**: [Baseline]

## 1. 목적
- `my-console` 구현에서 공통 컴포넌트의 prop, event, 상태 계약을 고정한다.
- 시각 스타일이 아니라 데이터 흐름과 인터랙션 계약을 우선 기준으로 정의한다.

## 2. 공통 규칙
- 서버 응답 원문은 page/container에서 view-model로 변환하고, 공통 컴포넌트는 가능한 한 도메인 중립 prop을 받는다.
- 비동기 처리 상태는 `loading`, `submitting`, `disabled`, `error`를 분리한다.
- 이벤트는 부수효과가 아니라 사용자 의도를 표현하는 이름으로 정의한다. 예: `onSubmit`, `onSelectRow`.

## 3. 레이아웃 및 권한 컴포넌트

### 3.1 AppShell
- `projectOptions: { id: number; name: string }[]`
- `selectedProjectId: number | null`
- `authorities: string[]`
- `sidebarCollapsed: boolean`
- `globalLoading?: boolean`
- `breadcrumbs: BreadcrumbItem[]`
- `onSelectProject: (projectId: number) => void`
- `onToggleSidebar: () => void`
- `onLogout: () => void`

### 3.2 PermissionGate
- `permission: string`
- `projectId?: number`
- `mode?: 'hide' | 'disable' | 'readonly'`
- `fallback?: ReactNode`
- `children: ReactNode`

동작 규칙:
- `hide`: 권한이 없으면 children을 렌더링하지 않는다.
- `disable`: children 상호작용을 막고 설명 tooltip/문구를 허용한다.
- `readonly`: 조회는 허용하되 편집 핸들러를 연결하지 않는다.

## 4. 데이터 표시 컴포넌트

### 4.1 EntityTableProps<T>
- `rows: T[]`
- `columns: ColumnDef<T>[]`
- `loading: boolean`
- `emptyTitle: string`
- `emptyDescription?: string`
- `selectedRowId?: string | number | null`
- `sort?: { field: string; direction: 'asc' | 'desc' } | null`
- `pagination?: { page: number; size: number; totalElements?: number }`
- `error?: ApiErrorViewModel | null`
- `onSelectRow?: (row: T) => void`
- `onSortChange?: (field: string, direction: 'asc' | 'desc') => void`
- `onPageChange?: (page: number) => void`
- `onCreate?: () => void`
- `onRefresh?: () => void`

### 4.2 DetailPanelProps<T>
- `title: string`
- `item: T | null`
- `loading?: boolean`
- `error?: ApiErrorViewModel | null`
- `actions?: ReactNode`
- `children: ReactNode`

### 4.3 StatusBadgeProps
- `status: string`
- `variant?: 'neutral' | 'primary' | 'success' | 'warning' | 'error'`
- `tooltip?: string`

## 5. 입력/편집 컴포넌트

### 5.1 CrudFormDialogProps<TDraft>
- `open: boolean`
- `mode: 'create' | 'edit'`
- `title: string`
- `draft: TDraft`
- `errors: ApiErrorViewModel | null`
- `submitting: boolean`
- `dirty?: boolean`
- `submitDisabled?: boolean`
- `onChange: (next: TDraft) => void`
- `onSubmit: () => void`
- `onClose: () => void`

### 5.2 JsonEditorProps
- `value: unknown`
- `readOnly?: boolean`
- `height?: number | string`
- `schemaHint?: string | null`
- `parseError?: string | null`
- `onChange?: (next: unknown) => void`

### 5.3 SchemaEditorFormProps
- `draft: DataDefinitionDraft`
- `errors: ApiErrorViewModel | null`
- `submitting?: boolean`
- `onChange: (next: DataDefinitionDraft) => void`
- `onSubmit: () => void`

### 5.4 ConnectionFormProps
- `draft: ConnectionDraft`
- `typeOptions: Array<'JDBC' | 'REST' | 'MQ' | 'SFTP'>`
- `secretPresence: Record<string, boolean>`
- `errors: ApiErrorViewModel | null`
- `submitting?: boolean`
- `onTypeChange: (type: ConnectionDraft['type']) => void`
- `onChange: (next: ConnectionDraft) => void`
- `onSubmit: () => void`

## 6. Flow Editor 컴포넌트

### 6.1 FlowCanvasProps
- `nodes: FlowNodeVM[]`
- `edges: FlowEdgeVM[]`
- `selectedNodeId: string | null`
- `readOnly?: boolean`
- `validationErrors?: FlowValidationIssue[]`
- `onNodesChange: (next: FlowNodeVM[]) => void`
- `onEdgesChange: (next: FlowEdgeVM[]) => void`
- `onSelectNode: (nodeId: string | null) => void`

### 6.2 NodeInspectorProps
- `node: FlowNodeVM | null`
- `availableConnections: ConnectionSummary[]`
- `validationErrors: FlowValidationIssue[]`
- `readOnly?: boolean`
- `onChange: (next: FlowNodeVM) => void`
- `onClose: () => void`

### 6.3 BindingEditorTableProps
- `items: FlowBindingDraft[]`
- `availableDefinitions: DataDefinitionSummary[]`
- `bindingVersion: number`
- `readOnly?: boolean`
- `conflictError?: ApiErrorViewModel | null`
- `onChange: (next: FlowBindingDraft[]) => void`
- `onSubmit: () => void`

## 7. 실행/운영 컴포넌트

### 7.1 RuntimeSummaryCardProps
- `title: string`
- `value: string | number | null`
- `status?: string | null`
- `loading?: boolean`
- `description?: string`

### 7.2 ExecuteRequestFormProps
- `mode: 'DRY_RUN' | 'EXECUTE_STUB'`
- `draft: DryRunDraft | ExecuteStubDraft`
- `submitting?: boolean`
- `errors?: ApiErrorViewModel | null`
- `onChange: (next: DryRunDraft | ExecuteStubDraft) => void`
- `onSubmit: () => void`

### 7.3 JsonPreviewPanelProps
- `title: string`
- `value: unknown`
- `emptyText?: string`
- `collapsed?: boolean`
- `onToggleCollapsed?: () => void`

## 8. 참조
- `docs/spec/guides/my-console-page-entity-view-model-spec.md`
- `docs/spec/guides/my-console-page-field-matrix.md`
- `docs/spec/guides/my-console-table-and-form-behavior-spec.md`
- `docs/spec/guides/my-console-flow-editor-node-contracts.md`
- `docs/spec/guides/ui-component-spec.md`
