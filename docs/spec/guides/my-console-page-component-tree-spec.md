# my-console Page Component Tree Spec

**Status**: [Baseline]

## 1. 목적
- 각 페이지를 어떤 container/presenter/common 컴포넌트로 분해할지 고정한다.

## 2. 규칙
- `Page`는 route-level composition만 담당한다.
- `Container`는 query/mutation orchestration과 mapper 호출을 담당한다.
- `View` 또는 presentational component는 렌더링과 event emit만 담당한다.

## 3. Page Trees

### 3.1 LoginPage
- `LoginPage`
  - `AuthLayout`
  - `LoginFormContainer`
    - `LoginFormView`
      - `TextField(username)`
      - `PasswordField(password)`
      - `InlineError`
      - `SubmitButton`

### 3.2 ProjectListPage
- `ProjectListPage`
  - `AppShell`
  - `ProjectListContainer`
    - `PageHeader`
    - `ProjectTable`
    - `ProjectFormDialog`
    - `DeleteConfirmDialog`

### 3.3 ProjectDetailPage
- `ProjectDetailPage`
  - `AppShell`
  - `ProjectDetailContainer`
    - `ProjectHeaderCard`
    - `ProjectTabNav`
    - `ProjectSummaryPanel`
    - `ProjectFormDialog`
    - `DeleteConfirmDialog`

### 3.4 FlowListPage
- `FlowListPage`
  - `AppShell`
  - `FlowListContainer`
    - `PageHeader`
    - `FlowSearchBar`
    - `FlowTable`
    - `FlowCreateDialog`

### 3.5 FlowEditorPage
- `FlowEditorPage`
  - `AppShell`
  - `FlowEditorContainer`
    - `FlowEditorHeader`
    - `FlowPalettePanel`
    - `FlowCanvas`
    - `NodeInspector`
    - `ValidationSummaryPanel`
    - `UnsavedConfirmDialog`

### 3.6 DataDefinitionPage
- `DataDefinitionPage`
  - `AppShell`
  - `DataDefinitionContainer`
    - `PageHeader`
    - `DataDefinitionTable`
    - `DataDefinitionDetailPanel`
      - `DataDefinitionMetaForm`
      - `SchemaEditorForm`
      - `JsonPreviewPanel`
    - `DeleteConfirmDialog`

### 3.7 ConnectionPage
- `ConnectionPage`
  - `AppShell`
  - `ConnectionContainer`
    - `PageHeader`
    - `ConnectionTable`
    - `ConnectionDetailPanel`
      - `ConnectionCommonFields`
      - `ConnectionTypeFields`
      - `SecretPresencePanel`
    - `DeleteConfirmDialog`

### 3.8 FlowBindingPage
- `FlowBindingPage`
  - `AppShell`
  - `FlowBindingContainer`
    - `PageHeader`
    - `BindingEditorTable`
    - `ConflictBanner`

### 3.9 DeploymentPage
- `DeploymentPage`
  - `AppShell`
  - `DeploymentContainer`
    - `PageHeader`
    - `DeploymentTable`
    - `CreateDeploymentDialog`
    - `UpdateDeploymentStatusDialog`
    - `RollbackDialog`

### 3.10 RuntimeOpsPage
- `RuntimeOpsPage`
  - `AppShell`
  - `RuntimeOpsContainer`
    - `PageHeader`
    - `RuntimeCardGrid`
      - `RuntimeSummaryCard`
      - `SchedulerStatusCard`
      - `TriggerStatusCard`
      - `DispatchSummaryCard`
    - `ExecutionStatusPanel`

### 3.11 ExecutePage
- `ExecutePage`
  - `AppShell`
  - `ExecuteContainer`
    - `ExecuteHeader`
    - `ExecuteRequestForm`
    - `ExecutionResultPanel`
      - `JsonPreviewPanel`
      - `TraceTimeline`
      - `ExecutionStatusPanel`

## 4. 참조
- `docs/spec/guides/my-console-page-layout-wire-spec.md`
- `docs/spec/guides/my-console-frontend-code-structure-spec.md`
