# Functional DoD Matrix

**Status**: [Baseline]

## 1. 목적
- 이번 프로그램 범위 기능에 대해 구현 완료 판정 기준을 기능 단위로 고정한다.

## 2. 공통 DoD
- 코드가 메인 브랜치 기준으로 빌드 가능하다.
- API/이벤트/DTO 문서가 구현과 일치한다.
- 오류 응답이 `error-response-baseline.md`를 따른다.
- 권한 체크가 `security-authority-matrix.md`와 연결된다.
- 테스트가 최소 기준을 충족한다.

## 3. 기능별 DoD
| Feature | Code | Test | Docs | Evidence |
|---|---|---|---|---|
| Auth Login/Refresh | 로그인/토큰 갱신 API 구현 | auth success/failure API test | `control-plane-api-baseline.md` 반영 | 샘플 요청/응답 |
| Project CRUD | 목록/생성/조회/수정/삭제 구현 | API + service test | API 계약 반영 | happy path + not found 샘플 |
| Flow CRUD | flowDefinition 저장/조회/수정/삭제 구현 | API + validation test | API 계약 반영 | invalid flow 샘플 |
| DataDefinition CRUD | schema 저장/조회/수정/삭제 구현 | API + repository test | API 계약 반영 | validation error 샘플 |
| Connection CRUD | secret write-only 처리 포함 CRUD 구현 | API + masking test | API 계약 반영 | secret 미반환 샘플 |
| Flow Data Binding | binding 목록/저장 구현 | API + relation validation test | 역할 모델 문서 반영 | binding save 샘플 |
| Deployment API | 생성/상태변경/롤백 구현 | API + service test | `api-spec.md` 반영 | status transition 샘플 |
| Dry-Run | runtime 호출 orchestration + 결과 반환 | API + integration test | `api-spec.md` 반영 | success + invalid flow 샘플 |
| Execute-Stub Sync | 동기 실행 요청/응답 구현 | API + integration test | `api-spec.md` 반영 | success + failure 샘플 |
| Execute-Stub Async | 비동기 접수/상태 조회/idempotency 구현 | API + integration + contract test | 내부 실행 계약 반영 | duplicate request 샘플 |
| Runtime Status Read | skeleton/scheduler/trigger/dispatch/execution 조회 구현 | API test | `api-spec.md`, `runtime-operations-spec.md` 반영 | status 조회 샘플 |
| Runtime Event Publish | started/step/completed/failed 이벤트 발행 | producer contract test | `runtime-event-schema.md` 반영 | event fixture |
| Logging Consume/Persist | event consumer + read model 저장 구현 | consumer contract + persistence test | logging 아키텍처 반영 | persisted event 샘플 |

## 4. 최소 테스트 기준
### 4.1 API
- happy path
- validation error
- auth error
- permission error
- not found

### 4.2 Runtime
- dry-run success
- unsupported node type
- execute-stub sync success
- execute-stub async accepted/status
- idempotency key conflict

### 4.3 Event
- producer schema validation
- consumer idempotency
- persistence failure handling

## 5. 증적 형식
- API fixture JSON
- contract test 결과
- integration test 결과
- 필요 시 운영 화면 캡처 또는 로그 스니펫

## 6. 참조
- `docs/spec/foundation/release-scope.md`
- `docs/spec/api/control-plane-api-baseline.md`
- `docs/spec/quality/test-fixture-baseline.md`
- `docs/spec/api/api-spec.md`
- `docs/spec/runtime/runtime-event-schema.md`
- `docs/spec/foundation/traceability-spec.md`
