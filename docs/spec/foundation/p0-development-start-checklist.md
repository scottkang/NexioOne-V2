# P0 Development Start Checklist

**Status**: [Baseline]  
**Purpose**: `docs/spec` 기준으로 제품 개발 착수 전에 먼저 닫아야 할 P0 부족분을 한 장의 실행 체크리스트로 정리한다.  
**As Of**: `2026-04-18`

## 1. 사용 방법
- 본 문서는 `development-gap-analysis.md`와 `development-readiness-checklist.md`의 P0 항목만 실행 순서 기준으로 재정렬한 문서다.
- 각 항목은 "결정 완료 + 기준 문서 반영 + 후속 구현 팀이 재질문 없이 착수 가능" 상태가 되면 완료로 본다.
- P1/P2는 본 문서 범위에서 제외한다.

## 2. 착수 판정
- 아래 5개 체크가 모두 완료되면 `my-console-backend`, `my-backend`, `my-console`의 MVP 병렬 개발 착수를 허용한다.
- 하나라도 미완료면 스펙 해석 차이로 재작업 가능성이 높으므로 선행 문서 보완을 우선한다.

## 3. P0 체크리스트

### P0-1. 릴리즈 범위 단일화
- [ ] 이번 프로그램의 단일 기준 문서를 `release-scope.md`로 확정한다.
- [ ] `product-spec.md`, `modules/*.md`, 상세 설계 문서가 같은 범위 해석을 갖도록 정렬한다.
- [ ] 이번 프로그램과 차기 프로그램 범위를 문서에서 분리한다.

완료 기준:
- `in scope`, `out of scope`, `dependency`, `target milestone`이 한 문서에 고정된다.
- `Not Implemented`, `Baseline`, `Draft/Design`의 의미가 팀 공통 용어로 합의된다.
- `my-backend` 차기 범위(Redis, Quartz, JTA, real connector)가 이번 MVP 범위와 혼동되지 않는다.

주요 문서:
- `docs/spec/foundation/release-scope.md`
- `docs/spec/foundation/product-spec.md`
- `docs/spec/modules/my-backend.md`
- `docs/spec/modules/my-console-backend.md`
- `docs/spec/modules/my-console.md`

### P0-2. 외부 API 계약 고정
- [ ] Control Plane CRUD와 runtime/deployment API를 구현 기준 문서로 연결한다.
- [ ] 공통 오류 응답과 에러 코드 체계를 API 문서 예시와 동일하게 맞춘다.
- [ ] 목록 조회의 paging/sort/filter 규칙과 권한 매핑을 API 단위로 추적 가능하게 유지한다.

완료 기준:
- 인증, 프로젝트, Flow, DataDefinition, Connection, Deployment, dry-run, execute-stub, runtime status API에 대해 path/method/request/response/error가 문서화된다.
- `ApiErrorResponse`와 HTTP status/code 매핑이 예제와 표에서 동일하다.
- UI와 백엔드가 같은 DTO 이름과 필드 의미를 사용할 수 있다.

주요 문서:
- `docs/spec/api/control-plane-api-baseline.md`
- `docs/spec/api/api-spec.md`
- `docs/spec/api/api-dto-baseline.md`
- `docs/spec/api/error-response-baseline.md`
- `docs/spec/api/global-error-code-map.md`
- `docs/spec/security/security-authority-matrix.md`

산출물:
- 필요 시 `control-plane-api-baseline.md` 부속 OpenAPI 또는 DTO 표
- validation/auth/conflict/not-found 샘플 JSON

### P0-3. 내부 실행 계약과 실행 상태 소유권 확정
- [ ] `my-console-backend -> my-backend` 실행 payload를 단일 계약으로 유지한다.
- [ ] sync/async/idempotency/status 조회 규칙을 API와 내부 계약에서 동일하게 맞춘다.
- [ ] "실행 제어용 상태 조회"와 "사용자용 실행 이력 조회"의 소유 모듈과 read path를 확정한다.

완료 기준:
- `ExecuteRequest` 필드, async accepted 응답, `Idempotency-Key` 충돌 규칙이 문서 간 일치한다.
- `GET /api/projects/{projectId}/runtime/executions/{executionId}`의 정본이 `my-backend`임이 고정된다.
- 최종 이력/검색 API가 `my-console-backend` read model인지, 다른 경로인지 추가 결정이 남지 않는다.

주요 문서:
- `docs/spec/api/internal-api-contract-design.md`
- `docs/spec/api/internal-execution-schema.md`
- `docs/spec/api/api-spec.md`

산출물:
- contract test fixture: async accepted, running, succeeded, failed, idempotency conflict

### P0-4. runtime 이벤트 계약 단일화
- [ ] `my-backend -> logging-service` 이벤트 envelope와 payload schema를 단일 기준으로 고정한다.
- [ ] DLQ 이동 조건과 최소 운영 규칙을 문서화한다.
- [ ] event projection에 포함/제외할 필드를 고정한다.

완료 기준:
- `execution.started`, `execution.step.completed`, `execution.completed`, `execution.failed`의 필수 필드가 문서 간 동일하다.
- 민감정보 제외 규칙과 `eventId` 기반 멱등 소비 규칙이 고정된다.
- logging read model이 어떤 이벤트를 어떻게 저장하는지 consumer가 재질문 없이 구현 가능하다.

주요 문서:
- `docs/spec/runtime/runtime-event-schema.md`
- `docs/spec/api/internal-api-contract-design.md`
- `docs/spec/runtime/runtime-execution-logging-architecture.md`

산출물:
- producer/consumer contract fixture 4종
- DLQ 처리 기준 메모

### P0-5. 런타임 1차 의미론 고정
- [ ] 지원 노드/비지원 노드를 MVP 기준으로 잠근다.
- [ ] 노드별 required config, stub output, validation error를 fixture 수준으로 고정한다.
- [ ] 응답 조립 규칙과 trace 최소 필드를 명시한다.

완료 기준:
- `START`, `END`, `MAPPING`, `REST_CLIENT`, `SQL_EXECUTOR`만 1차 허용 노드로 유지된다.
- `DRY_RUN`과 `EXECUTE_STUB`의 차이가 문서와 예시 응답에서 일치한다.
- `outputContext`, `steps/trace`, 민감정보 마스킹, retry 가능 오류 분류가 구현 가능한 수준으로 정리된다.

주요 문서:
- `docs/spec/runtime/runtime-node-support-matrix.md`
- `docs/spec/api/internal-execution-schema.md`
- `docs/spec/api/api-spec.md`

산출물:
- node별 happy path / validation error fixture
- unsupported node fixture

## 4. 선행 순서
1. `release-scope.md`를 기준 문서로 확정한다.
2. 외부 API 계약과 에러 응답 규격을 닫는다.
3. 내부 실행 payload, async/status, idempotency 규칙을 닫는다.
4. runtime 이벤트와 DLQ 규칙을 닫는다.
5. 지원 노드와 stub 의미론을 fixture 수준으로 고정한다.

## 5. 모듈별 착수 조건
| 모듈 | 착수 전 최소 충족 조건 |
|---|---|
| `my-console-backend` | P0-1, P0-2, P0-3 완료 |
| `my-backend` | P0-1, P0-3, P0-4, P0-5 완료 |
| `logging-service` | P0-4 완료 |
| `my-console` | P0-1, P0-2 완료 |

## 6. 이번 문서의 비범위
- DataDefinition/Binding versioning 세부 정책
- Connection 유형별 추가 확장 설계
- 기능별 DoD 전체 매트릭스
- 운영 SLO/대시보드/Runbook 전체 상세화

위 항목은 후속 P1/P2로 분리한다.

## 7. 참조
- `docs/spec/foundation/development-gap-analysis.md`
- `docs/spec/foundation/development-readiness-checklist.md`
- `docs/spec/foundation/release-scope.md`
- `docs/spec/foundation/traceability-spec.md`
