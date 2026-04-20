# my-console Specification

**Status**: [Not Implemented] - 제품 범위에 포함되며, 본 문서는 목표 기능 기준만 정의한다.

## 개요
NexioOne의 프론트엔드 웹 애플리케이션으로, 사용자가 프로젝트를 설계하고 API를 관리하며 시스템 상태를 모니터링할 수 있는 직관적인 UI를 제공한다.

이번 프로그램 포함 범위는 `docs/spec/foundation/release-scope.md`를 우선 기준으로 해석한다. 본 문서의 상세 화면 아이디어는 차기 범위를 함께 포함할 수 있다.

## 기술 스택
- **Framework**: React (TypeScript)
- **Styling**: Tailwind CSS
- **Build Tool**: Vite
- **Routing**: React Router

## 주요 기능
...
### 1. Dashboard
- 차기 범위 중심.

### 2. API Management
- **API Designer**: 차기 범위
- **API Product**: 차기 범위

### 3. Project Management
- **Flow Designer**: 드래그 앤 드롭 방식의 워크플로우 설계 인터페이스.
- **Data Definitions**: 프로젝트 공통 데이터 구조 및 스키마 정의.

### 4. Monitoring & Ops
- **Transaction Viewer**: 차기 범위 또는 logging read model 연동 이후 확장
- **Deployment**: 환경별 배포 제어 및 롤백 기능.

## 이번 프로그램 포함
- 로그인 이후 기본 navigation
- 프로젝트/Flow/DataDefinition/Connection/Deployment 관리 화면
- dry-run / execute-stub 호출 화면
- runtime 상태 조회 화면

### 이번 프로그램 화면 기준
| Screen Group | Primary APIs | Required Permission | Notes |
|---|---|---|---|
| 로그인/세션 | `POST /api/auth/login`, `POST /api/auth/refresh` | 없음 | JWT `authorities`, `projectScopes` 저장 |
| 프로젝트/Flow/DataDefinition/Connection | `control-plane-api-baseline.md` 범위 CRUD | 조회 `PROJECT_READ`, 변경 `PROJECT_WRITE` | 기본 설계/편집 화면 |
| Deployment | `POST/PATCH/POST /api/projects/{projectId}/deployments*` | 조회 `DEPLOYMENT_READ`, 변경 `DEPLOYMENT_WRITE` | 실행 버튼과 분리 |
| Dry-Run / Execute-Stub | `POST /runtime/flows/{flowId}/dry-run`, `POST /runtime/flows/{flowId}/execute-stub` | `RUNTIME_EXECUTE` | reserved `DEPLOYMENT_EXECUTE` 미사용 |
| Runtime Status | `GET /runtime/skeleton`, `.../scheduler/status`, `.../triggers/status`, `.../dispatch/summary`, `.../executions/{executionId}` | `RUNTIME_READ` | 읽기 전용 모니터링/상태 확인 |

### 비범위
- `DEPLOYMENT_EXECUTE` 전용 UI
- transaction viewer / 장기 이력 read model 화면
- API designer / API product 화면
- 실 connector 실행 UI

## 연관 문서
- [Product Spec](../foundation/product-spec.md)
- [API Spec](../api/api-spec.md)
- [Frontend Development Guide](../guides/frontend-development-guide.md)
- [my-console Page Field Matrix](../guides/my-console-page-field-matrix.md)
- [my-console Page Entity & View-Model Spec](../guides/my-console-page-entity-view-model-spec.md)
- [my-console Component Contracts](../guides/my-console-component-contracts.md)
- [my-console Page State Transition Spec](../guides/my-console-page-state-transition-spec.md)
- [my-console Routing & IA Spec](../guides/my-console-routing-ia-spec.md)
- [Security Authority Matrix](../security/security-authority-matrix.md)
