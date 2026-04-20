# my-console Page State Transition Spec

**Status**: [Baseline]

## 1. 목적
- `my-console` 주요 페이지의 로딩, 빈 상태, 권한 오류, 충돌, 저장 성공 후 전이를 고정한다.

## 2. 공통 상태 집합
| State | Meaning | UI Rule |
|---|---|---|
| `bootstrap-loading` | 초기 route 진입 및 auth/session 확인 중 | 전역 skeleton 또는 blocking loader |
| `page-loading` | 페이지 기본 데이터 조회 중 | 본문 skeleton 표시, destructive action 숨김 |
| `empty` | 정상 응답이나 표시할 데이터 없음 | empty title/guide/action 노출 |
| `ready` | 조회 성공, 상호작용 가능 | 기본 상태 |
| `submitting` | create/update/delete 요청 중 | 관련 버튼 disable, 중복 제출 방지 |
| `forbidden` | `403` 또는 scope outside | 전용 접근 제한 상태 표시 |
| `not-found` | `404` | resource missing 상태 표시, 상위 목록 이동 제공 |
| `conflict` | `409` | 최신 데이터 재조회 및 충돌 안내 |
| `validation-error` | `400/422` field error | 필드 인라인 + 요약 배너 |
| `system-error` | `5xx` 또는 처리 불가 오류 | 재시도 버튼 + traceId 노출 |

## 3. 페이지별 전이 규칙

### 3.1 LoginPage
- 초기 상태: `ready`
- 로그인 요청: `submitting`
- 성공: auth context 저장 후 이전 보호 경로 또는 `/projects`로 이동
- 실패:
  - 인증 실패: `ready` + 인라인 오류
  - 시스템 오류: `system-error`

### 3.2 ProjectListPage
- 초기 진입: `page-loading`
- 데이터 0건: `empty`
- 행 존재: `ready`
- 생성 dialog 저장:
  - `submitting` 후 성공 시 목록 재조회, dialog close
  - `validation-error`면 dialog 유지
  - `forbidden`면 dialog close + 접근 제한 알림

### 3.3 ProjectDetailPage
- 초기 진입: `page-loading`
- `404`: `not-found`
- `403`: `forbidden`
- 수정/삭제:
  - 성공 시 상세 재조회 또는 `/projects` 이동
  - `conflict`는 없으나 최신 조회 후 사용자 안내 가능

### 3.4 FlowListPage / FlowEditorPage
- 목록: `page-loading` -> `empty` 또는 `ready`
- 편집기:
  - 진입 시 `page-loading`
  - dirty 상태에서 route leave 시 confirm
  - 저장 성공 시 dirty 해제 및 최신 version 반영
  - `validation-error`면 inspector/bottom panel에 오류 유지
  - `forbidden`/`not-found`면 편집기 read-only가 아니라 전용 상태 화면 전환

### 3.5 DataDefinitionPage
- schema parse 실패는 서버 오류가 아니라 local `validation-error`로 처리
- 저장 성공 시 목록/detail 동기화
- `schemaJson` 서버 validation 실패 시 `fieldErrors`를 JSON editor와 요약 패널에 동시 노출

### 3.6 ConnectionPage
- secret 미입력은 기존 secret 유지 의미로 해석한다.
- 저장 실패:
  - field error: type-specific field에 인라인 표시
  - forbidden: 편집 차단
  - system-error: secret 노출 없이 일반 오류만 표시

### 3.7 FlowBindingPage
- 진입 시 `page-loading`
- 저장 시 `bindingVersion` 포함
- `409 conflict`:
  - `conflict` 상태로 전환
  - 서버 최신 binding set 재조회 유도
  - 자동 overwrite 금지

### 3.8 DeploymentPage
- 목록 0건은 `empty`
- 생성/상태 변경/롤백 요청 중 해당 row action만 `submitting`
- `ADM-2001` 등 version conflict는 `conflict` 상태로 매핑
- 성공 후 목록 재조회

### 3.9 RuntimeOpsPage
- 각 카드가 독립 조회 가능하더라도 첫 진입은 aggregate `page-loading`
- 이후 refresh는 page blocking 없이 section-level loading 사용
- 특정 execution 조회 실패 `404`는 execution panel만 `not-found`

### 3.10 ExecutePage
- 요청 전: `ready`
- 실행 중: `submitting`
- `DRY_RUN` 성공:
  - sync result 즉시 표시
  - trace/output preview 갱신
- `EXECUTE_STUB` accepted:
  - accepted payload 표시
  - execution status polling 시작
- `BE-2103` idempotency conflict:
  - `conflict`
  - 기존 execution lookup 안내

## 4. 에러 매핑 규칙
| Condition | UI Handling |
|---|---|
| `400/422` + `details` 존재 | 필드 인라인 + 상단 요약 배너 |
| `401` | refresh 시도 후 실패하면 `/login` 이동 |
| `403` | 권한/범위 부족 상태 화면 |
| `404` | 리소스 없음 상태 화면 |
| `409` | 최신 데이터 재조회 안내, 자동 overwrite 금지 |
| `5xx` | toast + 재시도 버튼 + traceId 표시 |

## 5. 성공 후 후속 동작
- 생성 성공: 목록 invalidate + 생성 결과 선택 상태 반영
- 수정 성공: 상세 invalidate + dirty reset
- 삭제 성공: 상위 목록 또는 기본 탭으로 이동
- 실행 성공: 결과 패널 포커스 이동 허용

## 6. 참조
- `docs/spec/api/error-response-baseline.md`
- `docs/spec/guides/my-console-page-field-matrix.md`
- `docs/spec/guides/my-console-routing-ia-spec.md`
