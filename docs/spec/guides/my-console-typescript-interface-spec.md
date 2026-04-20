# my-console TypeScript Interface Spec

**Status**: [Baseline]

## 1. 목적
- DTO, ViewModel, FormDraft, 공통 컴포넌트 props를 코드 수준 shape로 고정한다.

## 2. Core Types
```ts
export interface ApiErrorViewModel {
  status: number;
  code: string;
  message: string;
  fieldErrors: Array<{ field: string; reason: string }>;
  traceId: string | null;
}

export interface BreadcrumbItem {
  label: string;
  href?: string;
}
```

## 3. Domain Types
```ts
export interface ProjectListItemVm {
  id: number;
  name: string;
  description: string;
  createdAt: string;
  updatedAt: string;
}

export interface ProjectFormDraft {
  name: string;
  description: string;
}

export interface FlowNodeVm {
  id: string;
  type: 'START' | 'END' | 'MAPPING' | 'REST_CLIENT' | 'SQL_EXECUTOR' | string;
  name: string;
  config: Record<string, unknown>;
  connectionRef: string | null;
  position: { x: number; y: number };
  statusBadge: string | null;
}

export interface FlowEdgeVm {
  id: string;
  source: string;
  target: string;
}

export interface FlowEditorVm {
  id: number;
  projectId: number;
  name: string;
  description: string;
  version: string;
  status: string;
  nodes: FlowNodeVm[];
  edges: FlowEdgeVm[];
  selectedNodeId: string | null;
  hasUnsavedChanges: boolean;
}

export interface DataDefinitionDraft {
  name: string;
  description: string;
  schemaJson: Record<string, unknown>;
  metadata: {
    dataFormat: string | null;
    structureType: string | null;
  };
}

export interface ConnectionDraft {
  name: string;
  type: 'JDBC' | 'REST' | 'MQ' | 'SFTP';
  properties: Record<string, unknown>;
  secrets: Record<string, string>;
}

export interface FlowBindingDraft {
  id?: number;
  role: string;
  bindingKey: string;
  displayOrder: number;
  required: boolean;
  dataDefinitionId: number;
}

export interface DryRunDraft {
  deploymentId?: number | null;
  inputContext: Record<string, unknown>;
  options: {
    trace: boolean;
    validateOnly: boolean;
  };
}

export interface ExecuteStubDraft {
  deploymentId?: number | null;
  inputContext: Record<string, unknown>;
  options: {
    async: boolean;
    trace: boolean;
  };
  idempotencyKey?: string;
}
```

## 4. Component Prop Types
```ts
export interface EntityTableProps<T> {
  rows: T[];
  columns: Array<unknown>;
  loading: boolean;
  emptyTitle: string;
  emptyDescription?: string;
  selectedRowId?: string | number | null;
  error?: ApiErrorViewModel | null;
  onSelectRow?: (row: T) => void;
  onRefresh?: () => void;
}

export interface CrudFormDialogProps<TDraft> {
  open: boolean;
  mode: 'create' | 'edit';
  title: string;
  draft: TDraft;
  errors: ApiErrorViewModel | null;
  submitting: boolean;
  dirty?: boolean;
  submitDisabled?: boolean;
  onChange: (next: TDraft) => void;
  onSubmit: () => void;
  onClose: () => void;
}
```

## 5. 참조
- `docs/spec/guides/my-console-page-entity-view-model-spec.md`
- `docs/spec/guides/my-console-component-contracts.md`
