# UI Component Specification

본 문서는 NexioOne 관리 콘솔(`my-console`)에서 공통으로 사용되는 UI 컴포넌트의 상세 명세와 스타일 가이드를 정의한다. 모든 컴포넌트는 **Tailwind CSS 4**와 **Lucide React** 아이콘 라이브러리를 기반으로 하며, 필요 시 **Radix UI** primitives를 사용하여 접근성(a11y)을 보장한다.

## 1. 디자인 시스템 기초 (Foundations)

### 1.1 색상 팔레트 (Color Palette)
Tailwind의 기본 색상 체계를 사용하되, 시맨틱 컬러를 CSS 변수로 관리하여 테마 확장을 고려한다.

| 구분 | CSS 변수 | 클래스 예시 | 용도 |
| :--- | :--- | :--- | :--- |
| **Primary** | `--color-primary` | `bg-primary` | 주요 액션, 브랜드 컬러 (Indigo-600) |
| **Secondary** | `--color-secondary` | `bg-secondary` | 보조 액션, 강조 (Slate-600) |
| **Success** | `--color-success` | `text-success` | 완료, 긍정적 상태 (Emerald-500) |
| **Warning** | `--color-warning` | `text-warning` | 주의, 대기 상태 (Amber-500) |
| **Error** | `--color-error` | `text-error` | 오류, 취소, 위험 상태 (Rose-500) |
| **Surface** | `--color-surface` | `bg-surface` | 배경, 카드, 모달 (White / Slate-50) |
| **Border** | `--color-border` | `border-border` | 구분선, 입력창 테두리 (Slate-200) |

### 1.2 타이포그래피 (Typography)
- **Font Family**: Inter, system-ui, sans-serif
- **Scales**:
  - `text-xs`: 0.75rem (12px) - 캡션, 보조 텍스트
  - `text-sm`: 0.875rem (14px) - 기본 본문, 입력창
  - `text-base`: 1rem (16px) - 강조 본문
  - `text-lg`: 1.125rem (18px) - 소제목
  - `text-xl`: 1.25rem (20px) - 페이지 제목

---

## 2. 공통 UI 컴포넌트 (Atomic Components)

### 2.1 Button
사용자의 액션을 유도하는 핵심 컴포넌트. `class-variance-authority (CVA)`를 사용하여 관리한다.

- **Variants**:
  - `solid`: 배경색이 채워진 스타일 (Primary 등)
  - `outline`: 테두리만 있는 스타일
  - `ghost`: 배경과 테두리가 없는 스타일 (호버 시 배경 생성)
  - `link`: 밑줄이 있는 텍스트 스타일
- **Sizes**: `sm` (height 32px), `md` (40px), `lg` (48px)
- **States**: `default`, `hover`, `active`, `disabled`, `loading` (Spinner 아이콘 노출)

### 2.2 Input & Forms
- **Input Field**:
  - `Base`: 둥근 테두리(`rounded-md`), 연한 회색 테두리(`border-slate-200`).
  - `Focus`: Primary 색상 아웃라인(`ring-2 ring-primary/20`).
  - `Error`: 빨간색 테두리와 하단 에러 메시지 노출.
- **Select**: Radix UI Select를 기반으로 스타일링. 네이티브 선택창보다 일관된 UX 제공.
- **Switch/Checkbox**: 슬림하고 현대적인 토글 인터페이스 적용.

### 2.3 Badge (Tag)
상태나 카테고리를 나타내는 소형 컴포넌트.
- **Types**: `neutral`, `primary`, `success`, `warning`, `error`.
- **Style**: 연한 배경색 + 진한 텍스트 조합 (예: Success Badge - `bg-emerald-50 text-emerald-700`).

---

## 3. 복합 UI 컴포넌트 (Molecular Components)

### 3.1 Modal (Dialog)
- **Structure**: Overlay + Content (Header, Body, Footer).
- **Behavior**: 배경 클릭 시 닫기(선택 사항), ESC 키 대응, 포커스 트랩 적용.
- **Sizes**: `sm`(400px), `md`(600px), `lg`(800px), `xl`(1140px).

### 3.2 Data Table
- **Features**:
  - `Sticky Header`: 스크롤 시 헤더 고정.
  - `Row Selection`: 체크박스를 통한 다중 선택.
  - `Sorting`: 컬럼 헤더 클릭 시 정렬 토글.
  - `Pagination`: 하단 페이지 네비게이션.
  - `Empty State`: 데이터가 없을 때의 일러스트 및 안내 문구.

### 3.3 Toast (Notification)
- **Position**: 우측 상단 또는 하단 중앙.
- **Auto-dismiss**: 3~5초 후 자동 소멸.
- **Action**: 토스트 내부에 '취소'나 '확인' 버튼 포함 가능.

---

## 4. 특화 컴포넌트 (Domain Specific)

### 4.1 Flow Designer Node
`React Flow` 캔버스 위에서 사용되는 노드 UI.

- **Header**: 노드 타입 아이콘 + 이름 + 삭제/복사 버튼.
- **Handles (Ports)**:
  - `Input`: 좌측 중앙 (데이터 유입).
  - `Output`: 우측 중앙 (데이터 유출).
  - `Status Indicator`: 실행 성공(Green), 실패(Red), 실행 중(Spinning) 표시.
- **Config Area**: 노드의 핵심 설정 요약 정보 표시.

### 4.2 JSON Editor
- `monaco-editor` 또는 `react-simple-code-editor` 활용.
- 문법 강조(Syntax Highlighting) 및 자동 완성 지원.
- 유효하지 않은 JSON 입력 시 실시간 에러 마킹.

---

## 5. 레이아웃 컴포넌트 (Layout)

### 5.1 AppLayout
- **Sidebar**: 접기/펴기 기능을 포함한 좌측 메뉴 바.
- **Header**: 현재 위치(Breadcrumb), 전역 검색, 알림, 프로필.
- **Content Area**: `main` 태그 내부에 실제 페이지 컨텐츠 렌더링 (Padding: 1.5rem ~ 2rem).

---

## 6. 접근성 가이드 (Accessibility)

- **Contrast**: 텍스트와 배경의 명도 대비 4.5:1 이상 유지.
- **Keyboard**: 모든 인터랙티브 요소는 Tab 키로 접근 가능해야 하며, Focus Ring이 명확히 보여야 함.
- **Aria Labels**: 아이콘만 있는 버튼에는 `aria-label` 필수 부여.
