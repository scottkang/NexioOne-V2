# NexioOne Frontend Development Guide

**Status**: [Baseline]

## 1. 개요
본 문서는 NexioOne의 관리 콘솔(`my-console`) 개발을 위한 표준 가이드라인을 정의한다. 일관된 코드 스타일과 효율적인 상태 관리를 통해 유지보수성을 높이는 것을 목적으로 한다.

## 2. 기술 스택 (Standard Stack)
- **Framework**: React 19 (TypeScript)
- **Build Tool**: Vite
- **Styling**: Tailwind CSS 4
- **State Management**:
  - **Server State**: React Query (@tanstack/react-query)
  - **Global/UI State**: React Context API
  - **Local State**: `useState`, `useReducer`
- **Iconography**: Lucide React
- **Flow Engine**: @xyflow/react

## 3. 상태 관리 전략

### 3.1 Server State (React Query)
- 모든 API 호출 결과(Data Fetching, Caching, Syncing)는 React Query로 관리한다.
- `src/hooks/queries` 디렉토리에 도메인별 커스텀 훅을 생성하여 재사용한다.
- **예시**:
  ```typescript
  export const useProjectList = () => {
    return useQuery({
      queryKey: ['projects'],
      queryFn: fetchProjects,
      staleTime: 5 * 60 * 1000,
    });
  };
  ```

### 3.2 Global/UI State (Context API)
- 인증 정보, 테마 설정, 현재 선택된 프로젝트 ID 등 전역적으로 공유가 필요한 UI 상태는 Context API를 사용한다.
- 불필요한 리렌더링 방지를 위해 Context를 관심사별로 작게 분리한다.

## 4. UI 및 스타일링 가이드
상세한 컴포넌트 규격 및 디자인 시스템 가이드는 [UI 컴포넌트 상세 명세](./ui-component-spec.md)를 참조한다.

### 4.1 Tailwind CSS 사용 원칙
- 모든 스타일은 Tailwind Utility Class를 우선적으로 사용한다.
- 반복되는 복잡한 클래스 조합은 `@apply` 대신 React 컴포넌트로 추상화한다.
- 정적인 수치(Spacing, Color)는 `tailwind.config.js`에 정의된 테마 값을 따른다.

### 4.2 컴포넌트 구조
- `src/components/common`: 버튼, 입력창, 모달 등 아토믹한 공용 컴포넌트
- `src/components/layout`: 헤더, 사이드바, 레이아웃 컨테이너
- `src/features/{domain}`: 특정 비즈니스 로직에 종속된 복합 컴포넌트

## 5. 권한 기반 렌더링 (RBAC Control)

- 사용자의 JWT 내 권한(`authorities`) 목록에 따라 UI 요소를 제어한다.
- `HasPermission` 래퍼 컴포넌트나 `usePermission` 훅을 사용한다.
- 현재 프로그램에서 화면 노출에 사용하는 권한은 `PROJECT_READ`, `PROJECT_WRITE`, `DEPLOYMENT_READ`, `DEPLOYMENT_WRITE`, `RUNTIME_READ`, `RUNTIME_EXECUTE`로 제한한다.
- `DEPLOYMENT_EXECUTE`는 reserved 권한이므로 이번 프로그램 UI 노출 조건에 사용하지 않는다.
- **예시**:
  ```tsx
  <HasPermission permission="RUNTIME_EXECUTE">
    <Button onClick={handleExecuteStub}>실행</Button>
  </HasPermission>
  ```

### 5.1 화면별 API / 권한 / 상태 모델 매핑
| Screen | Primary APIs | Required Permission | State Model |
|---|---|---|---|
| 로그인 | `POST /api/auth/login`, `POST /api/auth/refresh` | 없음 | 인증 토큰, `authorities`, `projectScopes` |
| 프로젝트 목록/상세 | `GET /api/projects`, `GET /api/projects/{projectId}` | `PROJECT_READ` | 현재 선택 프로젝트, 목록 paging/sort/filter |
| 프로젝트 생성/수정/삭제 | `POST /api/projects`, `PUT /api/projects/{projectId}`, `DELETE /api/projects/{projectId}` | `PROJECT_WRITE` | 폼 draft, 저장 중 상태, API validation error |
| Flow 목록/상세/편집 | `GET/POST/PUT/DELETE /api/projects/{projectId}/flows*` | 조회 `PROJECT_READ`, 편집 `PROJECT_WRITE` | `flowDefinition`, validation error, dirty flag |
| DataDefinition 관리 | `GET/POST/PUT/DELETE /api/projects/{projectId}/data-definitions*` | 조회 `PROJECT_READ`, 편집 `PROJECT_WRITE` | schema editor draft, validation error |
| Connection 관리 | `GET/POST/PUT/DELETE /api/projects/{projectId}/connections*` | 조회 `PROJECT_READ`, 편집 `PROJECT_WRITE` | connection form, masked response, write-only secret field |
| Flow Data Binding 관리 | `GET/PUT /api/projects/{projectId}/flows/{flowId}/data-bindings` | 조회 `PROJECT_READ`, 편집 `PROJECT_WRITE` | binding list, role uniqueness validation |
| Deployment 관리 | `POST /api/projects/{projectId}/deployments`, `PATCH .../status`, `POST .../rollback` | 조회 `DEPLOYMENT_READ`, 변경 `DEPLOYMENT_WRITE` | deployment status, transition availability, conflict error |
| Runtime 상태 조회 | `GET /api/projects/{projectId}/runtime/skeleton`, `.../scheduler/status`, `.../triggers/status`, `.../dispatch/summary`, `.../executions/{executionId}` | `RUNTIME_READ` | runtime summary cards, async execution status poll |
| Dry-Run / Execute-Stub 실행 | `POST /api/projects/{projectId}/runtime/flows/{flowId}/dry-run`, `POST /api/projects/{projectId}/runtime/flows/{flowId}/execute-stub` | `RUNTIME_EXECUTE` | execution request form, sync result, async accepted, idempotency conflict |

### 5.2 프론트엔드 해석 규칙
- 하나의 화면에서 조회와 편집 기능이 같이 있으면 읽기 권한으로 기본 화면은 열고, 쓰기 액션만 개별적으로 비활성화한다.
- Deployment 화면은 `DEPLOYMENT_WRITE`로 배포 생성/상태 변경/롤백만 제어하고, 실행 버튼은 두지 않는다.
- runtime 수동 실행 버튼은 `RUNTIME_EXECUTE`로만 제어한다.
- async execution status는 `RUNTIME_READ` 권한으로 조회 가능해야 하므로, 실행 권한이 없는 사용자도 상태 화면 자체는 읽기 전용으로 볼 수 있다.
- 프로젝트 scope 밖의 화면 진입 시 프론트는 버튼 숨김만으로 처리하지 않고 `403` 응답을 기준으로 접근 차단 상태를 표시한다.

## 6. 에러 핸들링 및 로딩 상태
- API 호출 시 `ApiErrorResponse` 표준 포맷에 따라 사용자에게 Toast 알림 또는 인라인 에러 메시지를 표시한다.
- 주요 페이지 진입 시 `Suspense` 또는 스켈레톤 UI를 사용하여 로딩 상태를 명시한다.

## 7. 참조
- `docs/spec/api/control-plane-api-baseline.md`
- `docs/spec/api/api-spec.md`
- `docs/spec/security/security-authority-matrix.md`
- `docs/spec/modules/my-console.md`
