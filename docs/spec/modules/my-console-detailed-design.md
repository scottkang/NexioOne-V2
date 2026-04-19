# my-console Detailed Design (Frontend Architecture & Flow Designer)

...

## 2. 아키텍처 및 폴더 구조
...

## 3. 메뉴 구성 및 네비게이션 구조 (Sitemap)

관리 콘솔의 주요 기능을 논리적으로 그룹화하여 계층적인 네비게이션을 제공한다.

### 3.1 상단 네비게이션 (Global Navigation)
- **Organization Selector**: 현재 소속된 조직 또는 환경(Dev, Stage, Prod) 선택.
- **Global Search**: 프로젝트, API, 트랜잭션 ID 등을 통합 검색.
- **System Status**: 시스템 건전성 요약 및 알림 센터.
- **User Profile**: 사용자 정보, 설정, 로그아웃.

### 3.2 사이드바 네비게이션 (Local Navigation)

| 대메뉴 | 소메뉴 | 설명 |
| :--- | :--- | :--- |
| **Dashboard** | Overview | 전체 트래픽, 에러율, 주요 지표 요약 |
| **API Management** | API Designer | API 엔드포인트 설계 및 보안 설정 |
| | API Products | 상품 그룹화 및 Quota/Rate Limit 관리 |
| | API Docs (OAS) | Swagger 기반 인터랙티브 문서 및 테스트 |
| **Project** | Projects | 프로젝트 목록 및 Flow/Data Def 관리 |
| | Shared Connections | DB, API, MQ 등 공통 연결 정보 관리 |
| **Monitoring & Ops** | Transaction Viewer | 실행 이력 추적 및 상세 페이로드 분석 |
| | Deployments | 환경별 배포 이력 및 롤백 제어 |
| | Alert History | 알람 발생 내역 및 알림 정책 연동 |
| **Settings** | Access Control | RBAC (User, Role, Permission) 관리 |
| | Global Config | 시스템 전역 변수 및 환경 설정 |
| | Audit Logs | 관리자 작업 감사 로그 조회 |

## 4. 상태 관리 전략 (State Management)

데이터의 성격에 따라 계층화된 상태 관리 라이브러리를 사용하며, 복잡도를 낮추기 위해 'Prop Drilling'을 지양하고 상태의 소유권을 명확히 한다.

### 4.1 Server State (React Query)
- **역할**: API 서버로부터 가져온 데이터의 캐싱, 동기화, 로딩/에러 상태 관리.
- **표준**:
    - **Query Keys**: `src/constants/queryKeys.ts`에서 상수로 관리하여 휴먼 에러 방지.
    - **Custom Hooks**: 모든 API 호출은 `useQuery` 또는 `useMutation`을 래핑한 커스텀 훅(예: `useProjectList`, `useUpdateFlow`)으로 구현하여 도메인 로직을 컴포넌트와 분리.
    - **Caching Policy**: 기본적으로 `staleTime`을 5분으로 설정하되, 실시간성이 중요한 지표 데이터는 필요에 따라 조정.

### 4.2 Global UI State (Zustand)
- **역할**: 여러 화면에서 공유되는 UI 상태(사이드바 개폐, 테마, 현재 선택된 프로젝트 정보 등).
- **표준**:
    - **Store 분리**: `authStore`, `layoutStore`, `projectStore` 등 도메인별로 Store를 잘게 나누어 불필요한 리렌더링 방지.
    - **Actions**: 상태 변경 로직은 Store 내부의 `actions` 객체로 정의하여 컴포넌트에서는 호출만 수행.

### 4.3 Form & Local State
- **Form State**: 복잡한 폼은 `react-hook-form`을 사용하고, `zod`를 연동하여 런타임 및 타입 수준의 유효성 검증을 일원화.
- **Local State**: 특정 컴포넌트 내부에서만 닫힌 상태(모달 Open/Close 등)는 `useState`를 사용.
- **Canvas State**: Flow Designer의 노드/엣지 데이터는 `React Flow` 전용 상태 시스템을 사용하되, 최종 저장 시점에만 Global/Server State로 동기화.

## 5. 컴포넌트 설계 표준 (Component Design Standards)

재사용성과 유지보수성을 극대화하기 위해 컴포넌트 계층 구조와 작성 규칙을 엄격히 준수한다.

### 5.1 컴포넌트 계층 구조 (Hierarchy)
- **Common (Shared)**: 버튼, 입력창, 모달 등 프로젝트 전역에서 재사용되는 원자적(Atomic) UI 컴포넌트.
- **Layout**: 애플리케이션의 기본 뼈대를 형성하는 컴포넌트 (Sidebar, Header, MainLayout).
- **Features**: 특정 기능 단위(예: API Designer, Flow Editor)에 종속된 컴포넌트. `features/{domain}/components` 폴더에 위치.
- **Pages**: 라우터와 직접 연결되는 최상위 컴포넌트. 비즈니스 로직을 조합하고 데이터를 주입하는 역할.

### 5.2 구현 규칙
- **Naming**: 컴포넌트 파일 및 이름은 `PascalCase`를 사용 (예: `ProjectList.tsx`).
- **Props Interface**: 모든 Props는 `interface` 또는 `type`을 통해 명시적으로 정의하고, `children`을 활용한 합성을 적극 권장.
- **Style**: **Tailwind CSS**를 기본으로 하며, 컴포넌트의 상태에 따른 스타일 변경은 `class-variance-authority (CVA)` 또는 `tailwind-merge`를 사용하여 선언적으로 관리.
- **Logic Separation**: UI 렌더링 외의 복잡한 로직은 별도의 `hooks`로 분리하여 컴포넌트의 가독성을 확보.

### 5.3 현재 프로그램 구현 기준
- 이번 프로그램 범위 페이지의 entity/DTO/view-model/prop 계약은 `guides/my-console-page-entity-view-model-spec.md`를 구현 기준으로 사용한다.
- 본 문서는 아키텍처/상세 설계 방향을 설명하고, 실제 페이지 필드 계약은 위 문서를 우선 기준으로 해석한다.

## 6. 핵심 컴포넌트 상세 설계

### 6.1 Flow Designer (Workflow Editor)
`React Flow`를 기반으로 한 시각적 워크플로우 설계 도구이다.

- **Canvas**: 노드와 엣지가 배치되는 메인 영역. 무한 캔버스 및 줌/팬 지원.
- **Node Inspector (Panel)**: 선택된 노드의 설정을 변경하는 우측 패널. 노드 타입별 동적 폼 생성.
- **Palette (Component Library)**:
    - **동적 로딩**: 하드코딩된 목록 대신 백엔드의 `/api/runtime/node-catalog` API를 호출하여 실시간으로 사용 가능한 컴포넌트 목록을 구성한다.
    - **메타데이터 활용**: 백엔드에서 내려주는 컴포넌트 타입, 아이콘 정보, 기본 설정 JSON 스키마를 기반으로 팔레트 아이템을 동적으로 렌더링한다.
- **Validation**: 순환 참조 방지 및 필수 연결 속성 검증 로직 포함.

### 6.2 Mapping Editor (Data Transformation)
소스 노드와 타겟 노드 간의 데이터 매핑을 위한 특화된 에디터이다.

- **Tree View**: 좌/우측에 각각 소스/타겟 데이터 구조를 트리 형태로 표시.
- **Anchor & Line**: 트리 노드 간 드래그로 연결(Edge) 생성.
- **Expression Support**: 매핑 라인 클릭 시 간단한 함수나 표현식을 주입할 수 있는 팝업 제공.

### 6.3 Transaction Viewer (Observability)
`logging-service`와 연동하여 트랜잭션의 상세 흐름을 시각화한다.

- **Timeline View**: 트랜잭션 전체 수행 시간을 타임라인 형태로 표시하여 병목 구간 식별 지원.
- **Sequence Diagram**: 노드 간 호출 순서 및 상태를 시각화.
- **Payload Inspector**: 각 노드 실행 시점의 Input/Output JSON 데이터 조회.
- **OTEL Deep Link**: 특정 로그 클릭 시 Grafana/Jaeger 트레이스 화면으로 점프하는 링크 제공.

## 7. UI/UX 디자인 원칙

- **Consistency**: Tailwind CSS 기반의 디자인 시스템을 구축하여 모든 화면에서 일관된 간격, 색상, 타이포그래피 유지.
- **Responsiveness**: 관리 도구의 특성상 데스크탑 환경에 최적화하되, 주요 모니터링 대시보드는 모바일 대응을 고려.
- **Feedback**: 모든 API 요청에 대해 Toast 메시지 또는 인라인 로더를 통해 즉각적인 피드백 제공.
- **Accessibility**: ARIA 속성 준수 및 키보드 네비게이션 지원 고려.

## 8. API 연동 및 데이터 동기화 규격

- **Base Instance**: Axios 인터페이스를 사용하여 요청/응답 인터셉터를 구현.
- **Component Catalog Sync**:
    - **Source of Truth**: `my-backend`에 등록된 `FlowNodeExecutor` Bean 목록이 카탈로그의 원천이 된다.
    - **Proxy & Cache**: `my-console-backend`는 이 정보를 프록시하여 제공하며, 빈번한 호출을 방지하기 위해 프론트엔드 앱 진입 시 1회 로드 후 `React Query`를 통해 메모리에 유지한다.
- **Auth Interceptor**: 401 Unauthorized 에러 시 세션 만료 알림 및 로그인 페이지 리다이렉션 처리.
- **Error Handling**: 서버의 공통 에러 응답 구조를 파싱하여 사용자 친화적인 메시지로 변환.

## 9. 최적화 및 안정성

- **Code Splitting**: 라우트 단위로 `React.lazy`를 적용하여 초기 번들 크기 최소화.
- **Virtual Scrolling**: 대량의 트랜잭션 목록 조회 시 가상 스크롤을 적용하여 렌더링 성능 확보.
- **Test Strategy**:
  - **Unit Test**: Vitest + React Testing Library를 통한 로직 및 UI 검증.
  - **E2E Test**: Playwright를 이용한 핵심 사용자 시나리오(Flow 설계 및 저장) 테스트.

## 10. 배포 및 환경 설정

- **Vite Configuration**: 환경별(`.env.development`, `.env.production`) API 엔드포인트 및 상수 설정.
- **Containerization**: Nginx를 베이스 이미지로 사용하여 빌드된 정적 자원을 서빙하는 Docker 이미지 생성.
