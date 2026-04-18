# Development Execution Backlog

**Status**: [Baseline]
**Purpose**: `docs/spec` 기준의 개발 착수 보완 작업을 실제 실행 단위 backlog로 정리한다.
**As Of**: `2026-04-18`

## 1. 판정
- 현재 `docs/spec`는 MVP 범위, 핵심 계약, fixture baseline의 큰 축이 정리됐다.
- `P0-1`~`P0-5`, `P1-3`, `P1-4`는 문서 기준선 관점에서 완료로 본다.
- 현재 잔여 backlog는 `DataDefinition/Binding 전환 규칙`, `Connection 유형별 세부 계약`, `문서 링크/상태 체계` 보강으로 압축된다.
- 따라서 현재 상태는 `문서 기준 구현 착수 가능 + P1 보강 잔여`다.

## 2. 실행 원칙
- P0는 "팀이 재질문 없이 착수 가능" 상태를 만드는 작업만 포함한다.
- P1은 MVP 구현 착수는 가능하지만, 구현 중 재작업 가능성을 줄이기 위해 빠르게 이어서 닫아야 하는 작업을 포함한다.
- 각 backlog 항목은 문서 수정으로 끝내지 않고 `산출물`까지 남겨야 완료로 본다.

## 3. 권장 실행 순서
1. DataDefinition / Flow Binding 전환 규칙을 잠근다.
2. Connection 유형별 세부 계약과 validation 규칙을 fixture 수준으로 보강한다.
3. 문서 링크와 상태 체계를 최종 정리한다.

## 4. P0 Backlog

현황:
- `P0-1`: 완료
- `P0-2`: 완료
- `P0-3`: 완료
- `P0-4`: 완료
- `P0-5`: 완료

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

현황:
- `P1-1`: 미완료
- `P1-2`: 미완료
- `P1-3`: 완료
- `P1-4`: 완료

### P1-1. DataDefinition / Flow Binding 전환 규칙 보강
- 목표:
  - `DataDefinition.type`의 deprecated 처리와 제거 시점 명확화
  - binding versioning, update conflict 정책 확정
- 주요 문서:
  - `docs/spec/data/flow-owned-data-definition-roles.md`
  - `docs/spec/api/control-plane-api-baseline.md`
  - `docs/spec/api/internal-execution-schema.md`
- 권장 산출물:
  - binding versioning rule 표
  - update conflict / migration fixture
- 완료 기준:
  - `DataDefinition.type`의 유지/제거 시점이 문서에 명시된다.
  - binding update 충돌 정책이 API와 runtime snapshot 규칙에 연결된다.

### P1-2. Connection 유형별 계약 보강
- 목표:
  - JDBC/REST/MQ/SFTP별 validation, secret key, runtime 전달 규칙을 fixture와 연결
- 주요 문서:
  - `docs/spec/data/connection-profile-contract.md`
  - `docs/spec/security/security-encryption-design.md`
  - `docs/spec/api/internal-execution-schema.md`
- 권장 산출물:
  - 유형별 필수/옵션 필드 표
  - JDBC/REST/MQ/SFTP fixture seed
- 완료 기준:
  - 유형별 필수 필드와 secret key 이름이 문서에 잠긴다.
  - 외부 API request, 저장 모델, runtime 전달 shape가 같은 용어로 연결된다.

### P1-3. 테스트 전략을 실행 단위로 연결
- 상태: 완료 (`#17`, PR `#18`)
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
- 상태: 완료 (`#19`, PR `#20`)
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

### Session F
- `P1-1` DataDefinition / Flow Binding 전환 규칙 고정
- binding versioning / conflict 정책 문서화

### Session G
- `P1-2` Connection 유형별 계약 보강
- JDBC/REST/MQ/SFTP validation 및 secret key 규칙 fixture 연결

### Session H
- 문서 링크/상태 체계 정리
- `Baseline` / `Draft/Design` / `Not Implemented` 표기 규칙과 깨진 링크 정비

## 8. 현재 리스크
- `Connection` 유형별 세부 계약이 아직 공통 shape 수준이라 구현 시 추정이 섞일 수 있다.
- `DataDefinition` / binding 전환 규칙이 완전히 잠기지 않아 향후 CRUD/배포 snapshot 처리에서 재작업 가능성이 있다.
- 일부 문서는 상태 표기나 참조 링크가 여전히 균일하지 않다.
- 저장소에는 `my-backend/`, `my-console-backend/` 로컬 자산이 남아 있어 문서 작업과 구현 작업 범위가 섞여 보일 수 있다.

## 9. 권장 다음 액션
1. `P1-1` DataDefinition / Flow Binding 전환 규칙 보강을 먼저 닫는다.
2. 이어서 `P1-2` Connection 유형별 계약 보강을 fixture 중심으로 닫는다.
3. 마지막으로 문서 상태/링크 체계를 정리하고 구현 착수용 backlog를 다시 압축한다.

## 10. 참조
- `docs/spec/foundation/development-gap-analysis.md`
- `docs/spec/foundation/development-readiness-checklist.md`
- `docs/spec/foundation/p0-development-start-checklist.md`
- `docs/spec/foundation/functional-dod-matrix.md`
- `docs/spec/foundation/traceability-spec.md`
