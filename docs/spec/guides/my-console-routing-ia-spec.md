# my-console Routing & IA Spec

**Status**: [Baseline]

## 1. 목적
- `my-console` 현재 프로그램 범위의 메뉴 구조, 라우팅, breadcrumb, URL 파라미터 규칙을 고정한다.

## 2. 상위 정보 구조
| Section | Primary Route | Notes |
|---|---|---|
| Auth | `/login` | 비보호 |
| Projects | `/projects` | 기본 landing |
| Project Detail | `/projects/:projectId` | 하위 기능 진입점 |
| Runtime | `/projects/:projectId/runtime` | 읽기 전용 상태 화면 |
| Execute | `/projects/:projectId/runtime/flows/:flowId/execute` | 수동 실행 화면 |

## 3. 메뉴 구조
- Global Sidebar
  - `Projects`
- Project Context Navigation
  - `Flows`
  - `Data Definitions`
  - `Connections`
  - `Deployments`
  - `Runtime`

이번 프로그램 비범위:
- Dashboard 전용 landing
- Transaction Viewer
- API Designer / API Product

## 4. 라우트 매핑
| Route | Page | Required Permission | Params / Query |
|---|---|---|---|
| `/login` | LoginPage | 없음 | `redirect?` |
| `/projects` | ProjectListPage | `PROJECT_READ` | `page`, `size`, `sort` |
| `/projects/:projectId` | ProjectDetailPage | `PROJECT_READ` | `tab?` |
| `/projects/:projectId/flows` | FlowListPage | `PROJECT_READ` | `page`, `size`, `sort`, `q?` |
| `/projects/:projectId/flows/:flowId` | FlowEditorPage | `PROJECT_READ` | `panel?`, `nodeId?` |
| `/projects/:projectId/data-definitions` | DataDefinitionPage | `PROJECT_READ` | `selected?`, `mode?` |
| `/projects/:projectId/connections` | ConnectionPage | `PROJECT_READ` | `selected?`, `mode?` |
| `/projects/:projectId/flows/:flowId/data-bindings` | FlowBindingPage | `PROJECT_READ` | none |
| `/projects/:projectId/deployments` | DeploymentPage | `DEPLOYMENT_READ` | `flowId?`, `status?` |
| `/projects/:projectId/runtime` | RuntimeOpsPage | `RUNTIME_READ` | `executionId?`, `poll?` |
| `/projects/:projectId/runtime/flows/:flowId/execute` | ExecutePage | `RUNTIME_EXECUTE` | `deploymentId?`, `mode?` |

## 5. Breadcrumb 규칙
| Route Pattern | Breadcrumb |
|---|---|
| `/projects` | `Projects` |
| `/projects/:projectId` | `Projects / {projectName}` |
| `/projects/:projectId/flows` | `Projects / {projectName} / Flows` |
| `/projects/:projectId/flows/:flowId` | `Projects / {projectName} / Flows / {flowName}` |
| `/projects/:projectId/data-definitions` | `Projects / {projectName} / Data Definitions` |
| `/projects/:projectId/connections` | `Projects / {projectName} / Connections` |
| `/projects/:projectId/flows/:flowId/data-bindings` | `Projects / {projectName} / Flows / {flowName} / Data Bindings` |
| `/projects/:projectId/deployments` | `Projects / {projectName} / Deployments` |
| `/projects/:projectId/runtime` | `Projects / {projectName} / Runtime` |
| `/projects/:projectId/runtime/flows/:flowId/execute` | `Projects / {projectName} / Runtime / {flowName} / Execute` |

## 6. 파라미터 해석 규칙
- `projectId`, `flowId`는 path param이며 항상 숫자다. 파싱 실패 시 `not-found`로 처리한다.
- `page`, `size`, `sort`, `q`, `tab`, `mode`, `executionId`, `deploymentId`는 query param이다.
- invalid query param은 hard error 대신 기본값으로 정규화한다.
- `panel=node-inspector`와 `nodeId`가 같이 오면 editor 진입 후 해당 노드를 선택한다.
- `poll=false`면 RuntimeOpsPage 자동 polling을 시작하지 않는다.

## 7. 뒤로가기 / deep-link 규칙
- 목록에서 상세로 이동 후 뒤로가기는 직전 목록 query state를 복원한다.
- ProjectDetailPage의 `tab` query는 새로고침 후에도 유지한다.
- ExecutePage는 `flowId`가 없는 임의 실행 화면을 만들지 않는다.
- forbidden/not-found 상태에서도 breadcrumb의 상위 링크는 유지한다.

## 8. 참조
- `docs/spec/modules/my-console.md`
- `docs/spec/guides/my-console-page-entity-view-model-spec.md`
- `docs/spec/guides/my-console-page-state-transition-spec.md`
