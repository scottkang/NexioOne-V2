# my-console Table & Form Behavior Spec

**Status**: [Baseline]

## 1. 목적
- `my-console` 목록/상세/폼의 상호작용 규칙을 고정한다.

## 2. 테이블 공통 규칙
- 기본 정렬:
  - `Project`, `Flow`, `DataDefinition`, `Connection`, `Deployment` 목록은 `updatedAt desc`
- 기본 페이지 크기: `20`
- 빈 상태:
  - 권한이 있으면 생성 CTA 노출
  - 권한이 없으면 안내 문구만 노출
- row click:
  - primary navigation이 있는 페이지는 row 전체 클릭으로 상세 진입
  - destructive action은 row click과 분리된 action cell에 둔다
- refresh:
  - 현재 page/sort/filter/query를 유지한 채 재조회

## 3. 컬럼 및 액션 기준
| Page | Default Columns | Row Actions |
|---|---|---|
| ProjectList | `name`, `description`, `updatedAt`, `createdAt` | open, edit, delete |
| FlowList | `name`, `status`, `version`, `updatedAt` | open, edit, delete |
| DataDefinition | `name`, `description`, `updatedAt` | select, edit, delete |
| Connection | `name`, `type`, `status`, `updatedAt` | select, edit, delete |
| Deployment | `flowId`, `version`, `status`, `requestedBy`, `updatedAt` | change status, rollback |

## 4. 폼 공통 규칙
- submit button 활성화 조건:
  - 필수 필드 충족
  - local parse/validation 성공
  - `submitting=false`
- dirty 판정:
  - 초기값 대비 trim/normalized 결과가 달라지면 dirty
  - secret field는 값 입력 시 dirty
- cancel/close:
  - `dirty=false`면 즉시 close
  - `dirty=true`면 discard confirm
- submit 성공:
  - dialog close
  - 관련 query invalidate
  - 성공 toast 표시
- submit 실패:
  - validation error: dialog 유지, field focus 이동
  - conflict/system error: dialog 유지, 상단 배너 표시

## 5. 페이지별 동작

### 5.1 Project / Flow / DataDefinition / Connection Create/Edit
- create는 modal/dialog 우선 허용
- edit는 dialog 또는 우측 detail panel 편집 허용
- delete는 always confirm dialog 필요

### 5.2 FlowEditor
- canvas 조작, inspector 편집, edge 연결 변경은 모두 dirty로 간주
- route leave, project change, browser refresh 시 dirty confirm 적용
- selected node가 삭제되면 inspector는 close
- unsupported node는 저장 전 local validation에서 차단

### 5.3 FlowBinding
- 저장은 전체 set replace 의미
- binding 행 reorder는 `displayOrder` 갱신으로 반영
- 중복 role/key는 로컬에서 먼저 차단
- `409` conflict 시 최신 bindingVersion 재조회 전까지 submit 재시도 금지

### 5.4 Deployment
- 상태 변경과 롤백은 row-level dialog 사용
- transition 불가 상태는 버튼 비활성화
- 성공 후 해당 row만 optimistic update하지 않고 목록 재조회

### 5.5 Runtime / Execute
- Runtime auto refresh는 page-level toggle로 제어
- Execute submit 중 mode 전환 금지
- async accepted 후 polling 시작 시 request form은 read-only 유지, 새 실행은 명시적 reset 후만 허용

## 6. Focus / Accessibility Rules
- dialog open 시 첫 editable control에 focus
- validation 실패 시 첫 오류 필드로 focus 이동
- destructive confirm에서 기본 focus는 cancel

## 7. 참조
- `docs/spec/guides/my-console-field-validation-rules.md`
- `docs/spec/guides/my-console-page-state-transition-spec.md`
- `docs/spec/guides/my-console-component-contracts.md`
