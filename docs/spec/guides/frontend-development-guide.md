# NexioOne Frontend Development Guide

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
- **예시**:
  ```tsx
  <HasPermission permission="DEPLOYMENT_EXECUTE">
    <Button onClick={handleDeploy}>배포 실행</Button>
  </HasPermission>
  ```

## 6. 에러 핸들링 및 로딩 상태
- API 호출 시 `ApiErrorResponse` 표준 포맷에 따라 사용자에게 Toast 알림 또는 인라인 에러 메시지를 표시한다.
- 주요 페이지 진입 시 `Suspense` 또는 스켈레톤 UI를 사용하여 로딩 상태를 명시한다.
