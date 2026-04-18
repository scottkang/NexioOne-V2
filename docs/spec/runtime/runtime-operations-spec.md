# NexioOne Runtime & Operations Spec

## 1. 목적
- 본 문서는 `my-backend`의 목표 운영 모드와 우선 구현 대상 상태 조회 규격을 정의한다.
- 현재 구현은 없으므로, 본 문서는 운영 결과 문서가 아니라 개발 목표 문서로 사용한다.

## 2. 현재 상태
- 현재 런타임 구현 상태: `NOT_IMPLEMENTED`
- 아래 항목은 목표 운영 규격이며, 구현 완료 전까지 실제 동작을 보장하지 않는다.

## 3. 목표 런타임 모드
- `DRY_RUN`
- `EXECUTE_STUB`
- 차기 범위:
  - 실제 connector 실행
  - 분산 스케줄러
  - 자동 재주입 워커

## 4. 우선 구현 운영 API

### 4.1 Runtime Skeleton
- `GET /api/projects/{projectId}/runtime/skeleton`
- 목적: 현재 프로젝트의 런타임 모드와 제공 예정 API를 조회

### 4.2 Scheduler Status
- `GET /api/projects/{projectId}/runtime/scheduler/status`
- 목적: 스케줄 정책 수와 활성 상태를 조회

### 4.3 Trigger Status
- `GET /api/projects/{projectId}/runtime/triggers/status`
- 목적: webhook, mq, sftp 트리거 채널별 준비 상태를 조회

### 4.4 Dispatch Summary
- `GET /api/projects/{projectId}/runtime/dispatch/summary`
- 목적: 실행 요청 건수, 성공/실패 건수, 최근 실행 시각을 조회

### 4.5 Execution Status
- `GET /api/projects/{projectId}/runtime/executions/{executionId}`
- 목적: 비동기 dry-run 또는 execute-stub 실행의 상태를 조회
- 정본: `my-backend`
- 범위:
  - stage 1에서는 실행 제어용 상태 조회를 담당한다.
  - 장기 이력/검색/집계는 `logging-service` read model에서 별도로 제공한다.

## 5. 실패 처리 목표 규격
- Flow 구조 오류: `422`
- 권한 오류: `401`, `403`
- 워커 미가용: `503`
- 중복 비동기 요청 충돌: `409`

## 6. 운영 문서 원칙
- 구현 전 단계에서는 `현재 운영 중`, `Ready For Program Exit`와 같은 표현을 사용하지 않는다.
- 운영 증적 문서는 구현 이후 별도 증적 문서로 분리한다.

## 7. 참조
- `docs/spec/api/api-spec.md`
- `docs/spec/api/internal-api-contract-design.md`
- `docs/spec/runtime/runtime-execution-logging-architecture.md`
