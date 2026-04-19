# Development Readiness Checklist

**Status**: [Draft/Design]
**Related Modules**: `my-console`, `my-console-backend`, `my-backend`, `api-gw`, `logging-service`, `plugins`, `k8s`

## 1. 목적
- 본 문서는 `docs/spec` 기준으로 NexioOne 제품 개발을 시작하기 전에 반드시 보완해야 할 명세/계약/검증 항목을 정의한다.
- 목표는 "문서만 읽고도 구현 범위, API 계약, 완료 기준을 흔들림 없이 해석할 수 있는 상태"를 만드는 것이다.

## 2. 사용 원칙
- 본 체크리스트는 구현 착수 전 점검표로 사용한다.
- 각 항목은 문서 보완, 리뷰, 승인 주체가 명확해야 완료로 판단한다.
- 상태가 `Draft`, `Planned`, `Under Development`인 설계 문서는 구현 계약 문서로 승격되기 전까지 단독 기준으로 사용하지 않는다.

## 3. 핵심 판정

### 3.1 개발 착수 가능
- 아래 4개 게이트가 모두 충족되어야 한다.
  - 범위 게이트
  - 계약 게이트
  - 검증 게이트
  - 운영 준비 게이트

### 3.2 개발 착수 보류
- 아래 중 하나라도 해당하면 착수를 보류한다.
  - 제품 범위 문서와 모듈 상태 문서가 서로 다르게 해석된다.
  - API/이벤트/데이터 계약이 예시 수준에 머무르고 필수 필드가 고정되지 않았다.
  - 기능 완료 기준이 테스트/증적과 연결되지 않았다.
  - 인덱스 문서가 존재하지 않는 파일을 참조한다.

## 4. 체크리스트

### 4.1 범위 게이트 (Release Scope)
- [ ] 이번 릴리즈 대상 모듈이 문서 한 장으로 고정되었다.
  - 최소 포함 항목: `in scope`, `out of scope`, `dependency`, `target milestone`
  - 완료 기준: `product-spec.md` 또는 별도 범위 문서에서 `my-console`, `my-console-backend`, `my-backend`, `api-gw`, `logging-service`의 포함 여부가 명시된다.
- [ ] `product-spec.md`와 `modules/*.md`의 상태 표기가 서로 충돌하지 않는다.
  - 완료 기준: `Implemented`, `Core Implemented`, `Program Exit Ready`, `Planned/In Progress`의 의미가 정의되고, 모듈별 상태가 동일한 해석으로 읽힌다.
- [ ] 차기 프로그램 범위가 현재 개발 대상과 혼동되지 않게 분리되었다.
  - 완료 기준: MVP, 현재 릴리즈, 차기 프로그램 항목이 문서상 별도 섹션으로 나뉜다.

### 4.2 문서 정합성 게이트 (Spec Hygiene)
- [ ] `docs/spec/README.md`의 링크가 실제 파일 기준으로 정리되었다.
  - 완료 기준: 존재하지 않는 파일 참조가 제거되거나 해당 문서가 생성되었다.
- [ ] 인덱스 문서에서 권고/참고 문서와 구현 기준 문서가 구분되었다.
  - 완료 기준: 각 문서에 `Baseline`, `Design`, `Draft`, `Reference` 등 사용 목적이 표시된다.
- [ ] 모듈 문서의 상태와 상세 설계 문서의 상태가 일관된다.
  - 완료 기준: 예를 들어 `my-backend`가 `Core Implemented`라면 어떤 기능이 구현 완료이고 어떤 기능이 초안인지 세부 문서에 대응된다.

### 4.3 API 계약 게이트 (External API)
- [ ] 주요 REST API에 대해 요청/응답 DTO가 문서화되었다.
  - 대상: 인증, 프로젝트/Flow/DataDefinition CRUD, 배포, dry-run, execute-stub, runtime 상태 조회
  - 완료 기준: 각 API에 path, method, request body, response body, 필수/옵션 필드, 예시 JSON이 있다.
- [ ] 에러 응답 계약이 고정되었다.
  - 완료 기준: `ApiErrorResponse` 필드, 에러 코드, HTTP status 매핑, validation 실패 예시가 문서화되었다.
- [ ] 목록 조회 API의 페이징/정렬/필터 규칙이 정의되었다.
  - 완료 기준: query parameter, default 값, 정렬 키, 최대 page size가 명시된다.
- [ ] 권한 경계가 API 단위로 추적 가능하다.
  - 완료 기준: 각 주요 API가 `security-authority-matrix.md`의 권한 코드와 연결된다.

### 4.4 내부 계약 게이트 (Module-to-Module Contract)
- [ ] `my-console-backend -> my-backend` 실행 요청 payload가 예시가 아니라 계약으로 고정되었다.
  - 완료 기준: `flowDefinition`, `connectionProfiles`, `inputContext`, `options`의 JSON schema 또는 동등 수준의 필드 표가 있다.
- [ ] 동기/비동기/스케줄 실행의 응답 규칙이 분리 정의되었다.
  - 완료 기준: 성공/실패/접수/중복 요청 시 응답 예시와 상태 전이가 문서화되었다.
- [ ] `Idempotency-Key` 처리 규칙이 정의되었다.
  - 완료 기준: 키 충돌 시 응답, 보존 기간, 재시도 허용 범위, 상태 조회 방식이 명시된다.
- [ ] `my-backend -> logging-service` 이벤트 스키마가 이벤트 유형별로 완성되었다.
  - 완료 기준: `execution.started`, `execution.step.completed`, `execution.completed`, `execution.failed`, DLQ 이벤트의 공통 필드와 개별 필드가 정의된다.
- [ ] 버전 호환 정책이 문서화되었다.
  - 완료 기준: payload schema 변경 시 호환 방식, optional field 추가 규칙, 구버전 worker 처리 정책이 명시된다.

### 4.5 런타임 실행 게이트 (Runtime Semantics)
- [ ] `flowDefinition`의 최소 실행 계약이 고정되었다.
  - 완료 기준: node, edge, start/end, node config, connectionRef, expression 사용 규칙이 문서화되었다.
- [ ] 지원 노드와 설계만 존재하는 노드가 구분되었다.
  - 완료 기준: `MAPPING`, `REST_CLIENT`, `SQL_EXECUTOR` 등 실행 가능 노드와 `WAIT`, `SPLIT/JOIN`, `FLOW_TASK` 등 설계 단계 노드가 표로 정리된다.
- [ ] 노드별 입력/출력/실패 의미론이 정의되었다.
  - 완료 기준: 각 노드에 대해 입력 컨텍스트, 출력 키, 예외 코드, 재시도 가능 여부가 문서화된다.
- [ ] 컨텍스트 저장 전략의 구현 단계가 명확하다.
  - 완료 기준: L1/L2/L3 중 현재 구현 범위와 차기 범위가 구분된다.
- [ ] 응답 구성 규칙이 고정되었다.
  - 완료 기준: `RETURN_COMPOSER`, 기본 반환, 민감정보 마스킹, size limit 초과 시 동작이 명시된다.

### 4.6 데이터 및 스키마 게이트 (Data Contract)
- [ ] `DataDefinition`의 현재 구현 범위와 목표 모델이 분리되었다.
  - 완료 기준: 현재 지원 필드와 미지원 고급 구조가 문서에서 명확히 구분된다.
- [ ] Flow와 DataDefinition의 바인딩 규칙이 고정되었다.
  - 완료 기준: 참조 키, 버전 정책, 배포 시 스냅샷 여부가 명시된다.
- [ ] Connection Profile 계약이 문서화되었다.
  - 완료 기준: JDBC, REST, MQ, SFTP 유형별 공통 필드와 민감 필드 처리 규칙이 정의된다.
- [ ] 영속 데이터 모델의 소유권이 정의되었다.
  - 완료 기준: 어느 모듈이 어떤 엔티티를 저장/조회/동기화하는지 표로 정리된다.

### 4.7 테스트 및 수용 게이트 (QA / DoD)
- [ ] 기능별 완료 기준(DoD)이 문서화되었다.
  - 완료 기준: 기능마다 코드, 테스트, 문서, 운영 증적 필요 여부가 체크리스트로 있다.
- [ ] 테스트 전략이 기능 명세와 연결되었다.
  - 완료 기준: 핵심 기능별로 unit, integration, contract, e2e 대상이 매핑된다.
- [ ] 계약 테스트 대상이 확정되었다.
  - 완료 기준: `my-console-backend <-> my-backend`, `my-backend <-> logging-service`의 contract test 책임 주체와 도구가 정해진다.
- [ ] 샘플 요청/응답 및 테스트 데이터셋이 준비되었다.
  - 완료 기준: 최소 happy path, validation error, auth error, runtime failure 샘플이 존재한다.
- [ ] 추적성 문서가 현재 범위 기준으로 갱신되었다.
  - 완료 기준: sprint/task/code/test/evidence 매핑이 현재 릴리즈 기능 기준으로 보강된다.

### 4.8 운영 준비 게이트 (Ops / Production Readiness)
- [ ] 개발 단계와 운영 단계의 모드 차이가 문서화되었다.
  - 완료 기준: `SKELETON + EXECUTE_STUB`와 실제 runtime worker 모드의 차이가 명시된다.
- [ ] 장애 처리 및 재처리 규칙이 고정되었다.
  - 완료 기준: trigger failure, dead-letter, manual reprocess, 자동 재주입 미구현 범위가 정리된다.
- [ ] 관측성 필수 지표가 정의되었다.
  - 완료 기준: 최소 API latency, execution success rate, queue lag, config sync drift, worker health 지표가 문서화된다.
- [ ] 운영 Runbook의 최소 세트가 확보되었다.
  - 완료 기준: 배포, 롤백, config sync 불일치, alert notification 실패, DLQ 조회/재처리 절차가 연결된다.

## 5. 우선순위 권고

### P0
- 범위 문서 확정
- 인덱스 깨진 링크 정리
- 외부 API DTO/에러 계약 고정
- 내부 실행 payload 및 이벤트 스키마 고정

### P1
- 런타임 노드 지원 범위/제약 표 정리
- 테스트 매핑 및 기능별 DoD 정리
- Connection/DataDefinition 계약 보강

### P2
- 운영 대시보드/알람/SLO 구체화
- 차기 프로그램 범위 문서 정리

## 6. 권장 산출물
- `product-spec.md` 보완 또는 별도 `release-scope.md`
- API 계약서 또는 OpenAPI/JSON Schema 기반 부속 문서
- `my-backend` 실행 payload schema 문서
- 이벤트 스키마 문서
- 기능별 DoD 및 테스트 매핑 표
- 링크 정리된 `docs/spec/README.md`

## 7. 판정 기록
| 항목 | 상태 | 담당 | 근거 문서 | 확인 일자 |
|---|---|---|---|---|
| 범위 게이트 | TODO |  |  |  |
| 문서 정합성 게이트 | TODO |  |  |  |
| API 계약 게이트 | TODO |  |  |  |
| 내부 계약 게이트 | TODO |  |  |  |
| 런타임 실행 게이트 | TODO |  |  |  |
| 데이터 및 스키마 게이트 | TODO |  |  |  |
| 테스트 및 수용 게이트 | TODO |  |  |  |
| 운영 준비 게이트 | TODO |  |  |  |
