# Development Gap Analysis

**Status**: [Baseline]
**Purpose**: `docs/spec` 기준으로 제품 개발 착수 전 보완이 필요한 항목을 우선순위와 결정 단위로 정리한다.
**As Of**: `2026-04-18`

## 1. 판정
- 현재 `docs/spec`는 개발 목표와 방향성은 충분히 담고 있다.
- 2026-04-18 보강 이후 범위, 오류 체계, 이벤트 스키마, fixture/traceability, 프론트 권한 정합성의 큰 충돌은 대부분 정리되었다.
- 하지만 구현 기준으로 바로 사용하기에는 binding 전환 규칙, connection 유형별 세부 계약, 문서 상태/링크 체계에서 보완 항목이 남아 있다.
- 따라서 현재 판정은 `개발 착수 가능 + P1 보완 잔여`다.

## 2. 핵심 리스크 요약

### P0-1. 릴리즈 범위와 외부 API 기준선 불일치
- `product-spec.md`는 인증/인가, 프로젝트/Flow/DataDefinition/Connection/Deployment 관리 API까지 이번 프로그램 범위로 정의한다.
- 하지만 `api-spec.md`는 `deployment`, `dry-run`, `execute-stub`, `runtime 조회`만 고정하고 CRUD/API 전반은 차기 보강 범위로 둔다.
- `control-plane-api-baseline.md`가 추가 보강되었으므로, 현재 이 항목은 "해결 진행 중" 상태로 본다.
- 영향:
  - `my-console-backend` 팀이 제어 평면 CRUD/API를 문서 없이 추정 구현하게 된다.
  - 프론트엔드와 백엔드 간 병렬 개발 기준이 없다.
- 필요 결정:
  - 이번 프로그램 범위에서 제어 평면 CRUD를 정말 포함할지 확정
  - 포함한다면 `api-spec.md` 또는 별도 부속 문서로 API 계약 고정

### P0-2. runtime 이벤트 계약 상충
- `internal-api-contract-design.md`, `runtime-execution-logging-architecture.md`, `runtime-event-schema.md`는 현재 `runtime.execution` + envelope + `execution.started/step.completed/completed/failed` 기준으로 정렬되었다.
- 영향:
  - `my-backend`와 `logging-service`를 병렬 구현할 수 없다.
  - contract test와 consumer schema validation 기준이 흔들린다.
- 필요 결정:
  - DLQ 재주입 정책과 운영 권한 모델 확정
  - logging read path를 DB direct read로 시작할지 read API로 시작할지 확정

### P0-3. 에러 응답 형식과 에러 코드 체계 불일치
- `api-spec.md`, `error-response-baseline.md`, `global-error-code-map.md`는 현재 `ApiErrorResponse` 단일 구조로 정렬되었다.
- 영향:
  - 공통 예외 핸들러와 프론트 에러 렌더링을 일관되게 설계할 수 없다.
  - contract test와 문서 예제가 서로 맞지 않는다.
- 필요 결정:
  - 없음. 구현 시 문서 준수 여부만 검증하면 된다.

### P0-4. 런타임 1차 구현 범위 미고정
- `product-spec.md`는 `dry-run`, `execute-stub`, 상태 조회 중심이다.
- 반면 `modules/my-backend.md`는 Redis, Quartz, JTA, SQL/File 노드, 분산 스케줄링까지 목표로 둔다.
- `internal-execution-schema.md`는 현재 stage 1 허용 노드를 `START`, `END`, `MAPPING`, `REST_CLIENT`, `SQL_EXECUTOR`로 반영했다.
- 영향:
  - 런타임 팀이 stub 우선 구현인지, 인프라/실노드 골격까지 포함인지 판단할 수 없다.
  - 기술 선택과 데이터 모델이 과도하게 선행될 위험이 있다.
- 필요 결정:
  - 노드별 stub output shape와 validation error 예시를 fixture 수준으로 고정
  - async execution status를 runtime API와 logging read model 사이에서 어떻게 노출할지 UI/API 단에서 확정

## 3. P1 보완 항목

### P1-1. DataDefinition과 Flow binding 모델 확정 필요
- `flow-owned-data-definition-roles.md`는 역할 중립 `DataDefinition`과 Flow별 binding 모델을 권장한다.
- 현재 baseline은 `TB_FLOW_DATA_BINDING`과 deployment snapshot binding 포함으로 정리되었다.
- 필요 결정:
  - `DataDefinition.type` 제거 시점
  - binding versioning 규칙과 update 충돌 정책

### P1-2. Connection Profile 세부 계약 부족
- `internal-execution-schema.md`는 공통 shape만 제시하고 유형별 필수 필드는 없다.
- JDBC/REST/MQ/SFTP별 최소 속성, secret key 이름, validation 규칙이 필요하다.
- 필요 결정:
  - 유형별 필수/옵션 필드 표 작성
  - 암호화 저장과 runtime 전달 규칙 연결

### P1-3. 기능별 DoD와 테스트 매핑 부족
- `qa-testing-strategy.md`, `traceability-spec.md`, `functional-dod-matrix.md`, `fixtures/` baseline이 보강되어 이 항목은 해결 완료로 본다.

### P1-4. 문서 링크와 상태 체계 정리 필요
- 프론트 권한 예시와 화면/API/상태 모델 매핑은 정리되었지만, 일부 문서의 상태 표기와 링크 정합성은 여전히 추가 정리가 필요하다.
- 필요 결정:
  - 깨진 링크 제거 또는 문서 생성
  - 각 문서 상단 상태와 용도를 일관되게 표기

## 4. 권장 착수 순서
1. `DataDefinition` / binding 전환 규칙을 확정한다.
2. `Connection` 유형별 계약을 확정한다.
3. 문서 상태 표기와 링크 체계를 정리한다.

## 5. 권장 산출물
- `flow-owned-data-definition-roles.md`
  - binding versioning, deprecated field transition, update conflict 정책
- `connection-profile-contract.md`
  - JDBC/REST/MQ/SFTP별 필수/옵션 필드, secret key, runtime 전달 규칙
- 문서 상태/링크 정리 diff
  - `Baseline`, `Draft/Design`, `Not Implemented` 표기 정렬
  - 깨진 링크 제거 또는 보완

## 6. 문서별 조치 제안
| 문서 | 조치 | 우선순위 |
|---|---|---|
| `flow-owned-data-definition-roles.md` | 채택 여부와 전환 시점 명시 | P1 |
| `control-plane-api-baseline.md` | binding conflict / deprecated field 처리 규칙 연결 | P1 |
| `internal-execution-schema.md` | binding snapshot/versioning 규칙 명문화 | P1 |
| `connection-profile-contract.md` | 유형별 계약 표와 validation 규칙 보강 | P1 |
| `security-encryption-design.md` | connection secret 저장/전달 규칙 연결 | P1 |
| `qa-testing-strategy.md` | 기능별 테스트 매핑 표 추가 | P1 |
| `traceability-spec.md` | 기능-계약-테스트-증적 연결 보강 | Done |
| `frontend-development-guide.md` | 화면/API/권한/상태 모델 매핑 보강 | Done |

## 7. 개발 착수 기준
- 아래가 충족되면 개발 착수 가능으로 본다.
  - 범위 문서가 단일 해석으로 고정됨
  - 외부 API 계약과 내부 계약이 서로 충돌하지 않음
  - 에러 응답과 코드 체계가 통일됨
  - 런타임 1차 지원 범위가 고정됨
  - 기능별 최소 테스트와 contract test 책임 주체가 정의됨
- 현재 판정:
  - 위 기준은 충족되며, 남은 항목은 구현 리스크를 더 줄이기 위한 P1 보강으로 관리한다.

## 8. 참조
- `docs/spec/foundation/product-spec.md`
- `docs/spec/foundation/development-readiness-checklist.md`
- `docs/spec/api/api-spec.md`
- `docs/spec/api/api-dto-baseline.md`
- `docs/spec/api/internal-api-contract-design.md`
- `docs/spec/api/internal-execution-schema.md`
- `docs/spec/runtime/runtime-execution-logging-architecture.md`
- `docs/spec/api/global-error-code-map.md`
- `docs/spec/data/flow-owned-data-definition-roles.md`
- `docs/spec/quality/qa-testing-strategy.md`
- `docs/spec/foundation/traceability-spec.md`
