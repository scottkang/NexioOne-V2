# my-console Page Layout Wire Spec

**Status**: [Baseline]

## 1. 목적
- `my-console` 현재 프로그램 범위 페이지의 섹션 배치와 화면 구조를 고정한다.

## 2. 공통 레이아웃 규칙
- Desktop:
  - 좌측 `AppShell Sidebar`
  - 상단 `Header / Breadcrumb / Global Actions`
  - 본문은 최대 3개 영역으로 분할한다: `primary content`, `secondary detail panel`, `right utility panel`
- Mobile:
  - sidebar는 drawer로 전환
  - breadcrumb는 한 줄 축약
  - detail panel은 bottom sheet 또는 별도 route로 전환
- 리스트 + 상세 조합 페이지는 기본적으로 `2-column` 구조를 사용한다.

## 3. 페이지별 배치

### 3.1 LoginPage
- 상단:
  - 로고/서비스명
- 본문 중앙 카드:
  - `username`
  - `password`
  - submit button
  - inline error
- 하단:
  - 도움말 문구

### 3.2 ProjectListPage
- 헤더:
  - `PageTitle`
  - `Create Project` button
  - optional refresh
- 본문:
  - table only
- dialog:
  - create/edit form

### 3.3 ProjectDetailPage
- 헤더:
  - project name
  - description summary
  - edit/delete actions
- 탭 바:
  - `Flows`, `Data Definitions`, `Connections`, `Deployments`, `Runtime`
- 본문:
  - 선택 탭 요약 카드 또는 deep link CTA

### 3.4 FlowListPage
- 헤더:
  - title
  - search box
  - create button
- 본문:
  - flow table
- secondary:
  - optional selected row summary

### 3.5 FlowEditorPage
- 헤더:
  - flow name
  - status badge
  - save / validate / back actions
- 본문 3분할:
  - left: node palette
  - center: canvas
  - right: node inspector
- 하단:
  - validation summary panel

### 3.6 DataDefinitionPage
- 헤더:
  - title
  - create button
- 본문 2분할:
  - left: definition table
  - right: detail/form panel
- detail panel 내부:
  - meta fields
  - schema editor
  - JSON preview

### 3.7 ConnectionPage
- 헤더:
  - title
  - create button
- 본문 2분할:
  - left: connection table
  - right: detail/form panel
- detail panel 내부:
  - common fields
  - type-specific section
  - secret presence section

### 3.8 FlowBindingPage
- 헤더:
  - flow name
  - binding version badge
  - save button
- 본문:
  - editable table
- 하단:
  - conflict / validation banner

### 3.9 DeploymentPage
- 헤더:
  - title
  - create deployment button
- 본문:
  - deployment table
- dialogs:
  - create deployment
  - change status
  - rollback confirm

### 3.10 RuntimeOpsPage
- 헤더:
  - title
  - polling toggle
  - refresh button
- 본문 grid:
  - runtime skeleton card
  - scheduler status card
  - trigger status card
  - dispatch summary card
- secondary panel:
  - execution status detail

### 3.11 ExecutePage
- 헤더:
  - flow name
  - mode toggle
  - submit button
- 본문 2분할:
  - left: request form / JSON input
  - right: result / trace / output preview
- 하단:
  - async status section

## 4. Responsive Rules
- `FlowEditorPage`는 모바일에서 `palette -> canvas -> inspector` 탭형 레이아웃 허용
- `DataDefinitionPage`, `ConnectionPage`는 모바일에서 목록과 detail을 한 화면에 동시에 두지 않는다.
- `RuntimeOpsPage` 카드 grid는 mobile에서 1열, tablet에서 2열, desktop에서 4열 허용

## 5. 참조
- `docs/spec/guides/my-console-routing-ia-spec.md`
- `docs/spec/guides/my-console-page-field-matrix.md`
- `docs/spec/guides/my-console-flow-editor-interaction-spec.md`
