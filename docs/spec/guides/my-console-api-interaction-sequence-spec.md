# my-console API Interaction Sequence Spec

**Status**: [Baseline]

## 1. 목적
- 페이지 진입, create/update/delete, polling 시 API 호출 순서와 query invalidation 규칙을 고정한다.

## 2. 공통 규칙
- 최초 진입 시 page-level skeleton을 띄운 뒤 primary query를 실행한다.
- 성공 후 invalidate는 동일 domain key만 우선 무효화한다.
- optimistic update는 이번 프로그램에서 기본 전략으로 사용하지 않는다.

## 3. Query Key Baseline
- `auth.session`
- `projects.list`
- `projects.detail`
- `flows.list`
- `flows.detail`
- `dataDefinitions.list`
- `connections.list`
- `flowBindings.detail`
- `deployments.list`
- `runtime.skeleton`
- `runtime.scheduler`
- `runtime.trigger`
- `runtime.dispatch`
- `runtime.execution`

## 4. Page Sequences

### 4.1 LoginPage
1. submit -> `POST /api/auth/login`
2. success -> token 저장
3. optional redirect path 확인
4. `projects.list` 진입

### 4.2 ProjectListPage
1. route enter -> `GET /api/projects?page&size&sort`
2. create submit -> `POST /api/projects`
3. success -> `projects.list` invalidate
4. optional newly created row select

### 4.3 ProjectDetailPage
1. route enter -> `GET /api/projects/{projectId}`
2. edit submit -> `PUT /api/projects/{projectId}`
3. success -> `projects.detail(projectId)` invalidate
4. delete submit -> `DELETE /api/projects/{projectId}`
5. success -> `projects.list` invalidate -> `/projects`

### 4.4 FlowListPage / FlowEditorPage
1. list enter -> `GET /flows`
2. create submit -> `POST /flows`
3. success -> `flows.list(projectId)` invalidate
4. editor enter -> `GET /flows/{flowId}`
5. save submit -> `PUT /flows/{flowId}`
6. success -> `flows.detail(flowId)` + `flows.list(projectId)` invalidate

### 4.5 DataDefinitionPage
1. route enter -> `GET /data-definitions`
2. select row -> optional detail query or list data reuse
3. create/update/delete -> corresponding CRUD call
4. success -> `dataDefinitions.list(projectId)` invalidate

### 4.6 ConnectionPage
1. route enter -> `GET /connections`
2. create/update/delete -> corresponding CRUD call
3. success -> `connections.list(projectId)` invalidate

### 4.7 FlowBindingPage
1. route enter -> `GET /flows/{flowId}/data-bindings`
2. save submit -> `PUT /flows/{flowId}/data-bindings`
3. success -> `flowBindings.detail(flowId)` invalidate
4. `409` -> latest bindingVersion reload CTA only

### 4.8 DeploymentPage
1. route enter -> deployment list query if available, else create/status actions 중심 lazy load 허용
2. create submit -> `POST /deployments`
3. status change -> `PATCH /status`
4. rollback -> `POST /rollback`
5. each success -> `deployments.list(projectId)` invalidate

### 4.9 RuntimeOpsPage
1. route enter -> parallel fetch
   - `GET /runtime/skeleton`
   - `GET /runtime/scheduler/status`
   - `GET /runtime/triggers/status`
   - `GET /runtime/dispatch/summary`
2. `executionId` 있으면 `GET /runtime/executions/{executionId}`
3. polling enabled면 interval refresh

### 4.10 ExecutePage
1. route enter -> optional prerequisite load
   - flow metadata
   - deployment options
2. dry-run submit -> `POST /dry-run`
3. success -> result panel update
4. execute-stub submit -> `POST /execute-stub`
5. sync success -> result panel update
6. async accepted -> execution status polling start
7. polling -> `GET /runtime/executions/{executionId}`

## 5. Polling Rules
- `RuntimeOpsPage`: default 15s
- `ExecutePage` async status: default 5s, terminal state 도달 시 중지
- tab hidden 상태에서는 polling 중지 허용

## 6. 참조
- `docs/spec/guides/my-console-page-state-transition-spec.md`
- `docs/spec/guides/my-console-routing-ia-spec.md`
