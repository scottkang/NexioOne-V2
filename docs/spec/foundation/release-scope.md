# Release Scope Baseline

**Status**: [Baseline]
**Target Milestone**: `MVP Runtime + Control Plane Baseline`
**As Of**: `2026-04-18`

## 1. 목적
- 이번 개발 프로그램에서 실제 구현 대상으로 간주하는 범위를 단일 문서로 고정한다.
- `product-spec.md`의 상위 방향성과 개별 설계 문서 사이 해석 차이를 줄인다.

## 1.1 단일 기준 규칙
- 이번 프로그램의 MVP 범위 해석은 본 문서를 최우선 기준으로 사용한다.
- `product-spec.md`, `modules/*.md`, `api/*.md`, `runtime/*.md`는 본 문서와 충돌하지 않도록 해석해야 한다.
- 개별 모듈 문서에 차기 범위가 함께 적혀 있어도, 이번 프로그램 포함 여부 판정은 본 문서를 따른다.
- 외부 API 계약의 단일 묶음은 아래 문서 집합으로 고정한다.
  - 제어 평면 CRUD/API: `docs/spec/api/control-plane-api-baseline.md`
  - Deployment/Runtime API: `docs/spec/api/api-spec.md`
  - 공통 DTO: `docs/spec/api/api-dto-baseline.md`
  - 공통 오류 응답: `docs/spec/api/error-response-baseline.md`

## 2. In Scope

### 2.1 Control Plane
- JWT 기반 인증/인가 기본 골격
- 프로젝트 관리 API
- Flow 관리 API
- DataDefinition 관리 API
- Connection 관리 API
- Deployment 생성/상태 변경/롤백 API
- Flow/DataDefinition/Connection/Deployment 영속화
- Deployment snapshot 생성

### 2.2 Runtime
- `DRY_RUN`
- `EXECUTE_STUB`
- execution status 조회
- runtime skeleton / scheduler status / trigger status / dispatch summary 조회
- `my-console-backend -> my-backend` 실행 요청 계약 수용

### 2.3 Logging
- `my-backend -> logging-service` 이벤트 발행
- execution / step 로그의 기초 read model 저장

### 2.4 UI
- 로그인 이후 기초 navigation
- 프로젝트/Flow/DataDefinition/Connection/Deployment 관리 화면
- dry-run / execute-stub 호출 화면
- runtime 상태 조회 화면

## 3. Out of Scope
- 실제 connector 기반 production execution
- `WAIT`, `SPLIT/JOIN`, `FLOW_TASK` 완전 지원
- 분산 스케줄러 실운영 구성
- XA/2PC 실구현
- object storage 결과 아카이브
- 자동 DLQ 재주입
- 고급 Rate Limit/Quota 운영 정책
- 운영 증적 기반 release sign-off

## 4. Runtime Support Level
| Level | 이번 프로그램 포함 여부 | 설명 |
|---|---|---|
| `SKELETON` | 포함 | runtime 상태 조회 및 실행 API 골격 |
| `DRY_RUN` | 포함 | flow 구조/입력 검증 및 trace 반환 |
| `EXECUTE_STUB` | 포함 | connector 실제 호출 없이 step 실행 결과 반환 |
| `REAL_CONNECTOR` | 제외 | 실제 JDBC/REST/MQ/SFTP 연결 수행 |
| `DISTRIBUTED_SCHEDULER` | 제외 | Redis lease/quartz cluster 기반 분산 실행 |

## 5. 의존성
- 인증/권한 기준: `security-authority-matrix.md`
- 외부 API 기준:
  - `control-plane-api-baseline.md`
  - `api-spec.md`
  - `api-dto-baseline.md`
  - `error-response-baseline.md`
- 내부 실행 계약: `internal-api-contract-design.md`
- 에러 응답 기준: `error-response-baseline.md`
- runtime 이벤트 기준: `runtime-event-schema.md`
- 노드 지원 범위: `runtime-node-support-matrix.md`

## 6. 완료 기준
- `release-scope.md`만 읽어도 이번 프로그램 포함/제외 범위를 판정할 수 있다.
- In Scope API가 요청/응답/에러 형식까지 문서로 고정된다.
- Control Plane CRUD와 runtime 실행 API가 같은 권한/에러 체계를 사용한다.
- runtime/logging 이벤트 계약이 단일 schema로 고정된다.
- 지원 노드와 비지원 노드가 문서로 구분된다.

## 7. 참조
- `docs/spec/foundation/product-spec.md`
- `docs/spec/foundation/development-gap-analysis.md`
