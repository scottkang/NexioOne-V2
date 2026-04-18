# Development Execution Backlog

**Status**: [Baseline]
**Purpose**: `docs/spec` 기준의 개발 착수 보완 작업을 실제 실행 단위 backlog로 정리한다.
**As Of**: `2026-04-18`

## 1. 판정
- 현재 `docs/spec`는 MVP 범위와 핵심 계약의 큰 축은 정리됐다.
- 하지만 구현팀이 바로 병렬 개발에 들어가기에는 아직 "단일 기준 문서 + 단일 계약 산출물 + fixture"가 부족하다.
- 따라서 현재 상태는 `문서 기반 구현 착수 직전`이며, 아래 backlog를 먼저 닫는 것을 권장한다.

## 2. 실행 원칙
- P0는 "팀이 재질문 없이 착수 가능" 상태를 만드는 작업만 포함한다.
- P1은 MVP 구현 착수는 가능하지만, 구현 중 재작업 가능성을 줄이기 위해 빠르게 이어서 닫아야 하는 작업을 포함한다.
- 각 backlog 항목은 문서 수정으로 끝내지 않고 `산출물`까지 남겨야 완료로 본다.

## 3. 권장 실행 순서
1. 범위와 기준 문서의 단일 해석을 고정한다.
2. 외부 API 계약을 단일 산출물로 고정한다.
3. 내부 실행 payload, async/status, idempotency 규칙을 고정한다.
4. runtime event, DLQ, read model 반영 규칙을 고정한다.
5. runtime node 의미론을 fixture 수준으로 고정한다.
6. fixture, traceability, DoD를 구현 기준으로 연결한다.
7. DataDefinition/Connection 세부 계약과 UI 권한 정합성을 보강한다.

## 4. P0 Backlog

### P0-1. 범위 단일화
- 목표:
  - `release-scope.md`를 MVP 범위의 유일한 해석 기준으로 고정
  - `product-spec.md`, `modules/*.md`, 상세 설계 문서의 현재/차기 범위 표현 정렬
- 주요 문서:
  - `docs/spec/foundation/release-scope.md`
  - `docs/spec/foundation/product-spec.md`
  - `docs/spec/modules/my-backend.md`
  - `docs/spec/modules/my-console-backend.md`
  - `docs/spec/modules/my-console.md`
- 산출물:
  - 범위 충돌 제거 diff
  - 상태 용어 정의 표 (`Baseline`, `Draft/Design`, `Not Implemented`)
- 완료 기준:
  - Redis, Quartz, JTA, real connector가 차기 범위로 일관되게 읽힌다.
  - MVP 포함/제외 항목을 팀이 한 문서로 판단할 수 있다.

### P0-2. 외부 API 단일 계약화
- 목표:
  - Control Plane CRUD + Deployment/Runtime API를 단일 계약 묶음으로 제공
  - 에러 응답, 권한, 페이징 규칙을 예시와 표에서 동일하게 유지
- 주요 문서:
  - `docs/spec/api/control-plane-api-baseline.md`
  - `docs/spec/api/api-spec.md`
  - `docs/spec/api/api-dto-baseline.md`
  - `docs/spec/api/error-response-baseline.md`
  - `docs/spec/api/global-error-code-map.md`
  - `docs/spec/security/security-authority-matrix.md`
- 산출물:
  - OpenAPI 또는 동등 수준의 request/response DTO 표
  - validation/auth/conflict/not-found fixture 샘플
- 완료 기준:
  - 프론트엔드와 백엔드가 같은 DTO 이름과 필드 의미를 사용한다.
  - `ApiErrorResponse` 예시와 HTTP 매핑이 문서 간 일치한다.

### P0-3. 내부 실행 계약과 상태 조회 소유권 확정
- 목표:
  - `my-console-backend -> my-backend` 실행 payload를 단일 계약으로 잠금
  - sync/async/status/idempotency 규칙을 외부 API와 내부 계약에 동시에 반영
- 주요 문서:
  - `docs/spec/api/internal-api-contract-design.md`
  - `docs/spec/api/internal-execution-schema.md`
  - `docs/spec/api/api-spec.md`
  - `docs/spec/foundation/system-overview-and-operations.md`
- 산출물:
  - async accepted/running/succeeded/failed/idempotency conflict fixture
  - 상태 조회 ownership 메모
- 완료 기준:
  - `GET /api/projects/{projectId}/runtime/executions/{executionId}`의 정본이 `my-backend`로 고정된다.
  - 장기 이력 조회가 logging read model 기반임을 문서에서 명시한다.

### P0-4. Runtime Event 및 Logging 계약 고정
- 목표:
  - `my-backend -> logging-service` 이벤트 envelope, payload, projection 규칙 통일
  - DLQ 이동 조건과 최소 운영 규칙 명문화
- 주요 문서:
  - `docs/spec/runtime/runtime-event-schema.md`
  - `docs/spec/runtime/runtime-execution-logging-architecture.md`
  - `docs/spec/api/internal-api-contract-design.md`
  - `docs/spec/foundation/system-overview-and-operations.md`
- 산출물:
  - producer/consumer contract fixture 4종
  - DLQ rule 메모
- 완료 기준:
  - `execution.started`, `execution.step.completed`, `execution.completed`, `execution.failed` 필드가 문서와 fixture에서 일치한다.
  - 민감정보 제외 규칙과 `eventId` 기반 멱등 소비 규칙이 고정된다.

### P0-5. Runtime 1차 의미론 고정
- 목표:
  - 지원 노드/비지원 노드/trace/output 규칙을 구현 가능한 수준으로 고정
  - `DRY_RUN`과 `EXECUTE_STUB`의 차이를 응답과 fixture로 확인 가능하게 정리
- 주요 문서:
  - `docs/spec/runtime/runtime-node-support-matrix.md`
  - `docs/spec/api/internal-execution-schema.md`
  - `docs/spec/api/api-spec.md`
- 산출물:
  - node별 happy path / validation error fixture
  - unsupported node fixture
- 완료 기준:
  - `START`, `END`, `MAPPING`, `REST_CLIENT`, `SQL_EXECUTOR`만 MVP 허용 노드로 유지된다.
  - `outputContext`, `steps/trace`, retry 가능 오류 분류가 구현 가능한 수준으로 잠긴다.

## 5. P1 Backlog

### P1-1. DataDefinition / Flow Binding 전환 규칙 보강
- 목표:
  - `DataDefinition.type`의 deprecated 처리와 제거 시점 명확화
  - binding versioning, update conflict 정책 확정
- 주요 문서:
  - `docs/spec/data/flow-owned-data-definition-roles.md`
  - `docs/spec/api/control-plane-api-baseline.md`
  - `docs/spec/api/internal-execution-schema.md`

### P1-2. Connection 유형별 계약 보강
- 목표:
  - JDBC/REST/MQ/SFTP별 validation, secret key, runtime 전달 규칙을 fixture와 연결
- 주요 문서:
  - `docs/spec/data/connection-profile-contract.md`
  - `docs/spec/security/security-encryption-design.md`
  - `docs/spec/api/internal-execution-schema.md`

### P1-3. 테스트 전략을 실행 단위로 연결
- 목표:
  - 기능별 테스트 책임, contract ownership, evidence 이름을 구현 task와 직접 연결
- 주요 문서:
  - `docs/spec/quality/qa-testing-strategy.md`
  - `docs/spec/quality/test-fixture-baseline.md`
  - `docs/spec/foundation/traceability-spec.md`
  - `docs/spec/foundation/functional-dod-matrix.md`
- 산출물:
  - `fixtures/` baseline 디렉터리
  - 기능별 happy path / error fixture seed

### P1-4. 프론트엔드 가이드 정합성 보정
- 목표:
  - reserved 권한과 실제 baseline API 권한이 혼동되지 않게 수정
  - 화면별 API/권한/상태 모델 매핑 추가
- 주요 문서:
  - `docs/spec/guides/frontend-development-guide.md`
  - `docs/spec/security/security-authority-matrix.md`
  - `docs/spec/modules/my-console.md`

## 6. 모듈별 착수 가능 시점
| 모듈 | 선행 완료 조건 | 비고 |
|---|---|---|
| `my-console-backend` | P0-1, P0-2, P0-3 | CRUD/API와 실행 오케스트레이션 착수 가능 |
| `my-backend` | P0-1, P0-3, P0-4, P0-5 | stub runtime, event publish, status API 착수 가능 |
| `logging-service` | P0-4 | consumer/read model 구현 착수 가능 |
| `my-console` | P0-1, P0-2 | 화면/DTO/권한 연동 착수 가능 |

## 7. 세션 단위 권장 작업 묶음

### Session A
- `release-scope`, `product-spec`, `modules/*.md` 범위 정렬
- 상태 용어 표 통일

### Session B
- 외부 API 단일 계약화
- 에러 응답/권한/페이징 정합성 점검

### Session C
- 내부 실행 계약, async/status/idempotency 고정
- runtime 상태 조회 ownership 확정

### Session D
- runtime event/DLQ/read model projection 규칙 고정
- producer/consumer fixture 작성 시작

### Session E
- node별 fixture, trace/output 규칙 고정
- 테스트/추적성 문서와 연결

## 8. 현재 리스크
- `fixtures/` 디렉터리가 아직 없어 contract test 기준물이 부재하다.
- 일부 문서는 `Baseline`이어도 실제 구현용 단일 산출물(OpenAPI/JSON Schema)까지는 내려오지 않았다.
- 프론트 권한 예시에 `DEPLOYMENT_EXECUTE`가 남아 있어 reserved 권한과 실제 사용 권한이 혼동될 수 있다.
- 기존 handoff는 `#1`, `type/1-docs-spec-p0-checklist` 기준이라 최신 세션 기준으로 갱신이 필요하다.

## 9. 권장 다음 액션
1. `P0-1`과 `P0-2`를 같은 PR 또는 연속 PR로 먼저 닫는다.
2. 이어서 `P0-3`과 `P0-4`를 contract/fixture 중심 PR로 묶는다.
3. 마지막으로 `P0-5`와 `P1-3`을 연결해 runtime fixture baseline을 만든다.

## 10. 참조
- `docs/spec/foundation/development-gap-analysis.md`
- `docs/spec/foundation/development-readiness-checklist.md`
- `docs/spec/foundation/p0-development-start-checklist.md`
- `docs/spec/foundation/functional-dod-matrix.md`
- `docs/spec/foundation/traceability-spec.md`
