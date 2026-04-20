# Spec Index

## Documentation Rules
- Mermaid 노드 라벨 줄바꿈은 `\n` 대신 `<br/>`를 사용한다.
- 문서 인덱스에는 현재 저장소에 실제로 존재하는 파일만 유지한다.
- 구현 결과와 목표 설계를 혼용하지 않도록 각 문서의 상태를 아래 집합으로 명시한다.
- `Baseline`: 현재 프로그램 기준선으로 직접 참조하는 문서
- `Draft/Design`: 목표 설계 또는 후속 보강이 남은 문서
- `Not Implemented`: 제품 범위에는 포함되지만 구현 전 상태의 모듈/영역 설명 문서

## Recommended Reading Order

### 1. 신규 개발자 온보딩
1. `foundation/product-spec.md`
2. `foundation/release-scope.md`
3. `foundation/development-gap-analysis.md`
4. `api/control-plane-api-baseline.md`
5. `api/api-spec.md`
6. `api/internal-api-contract-design.md`
7. `runtime/runtime-node-support-matrix.md`
8. `modules/README.md`

### 2. 백엔드 API 구현 시작
1. `foundation/release-scope.md`
2. `api/control-plane-api-baseline.md`
3. `api/api-spec.md`
4. `api/error-response-baseline.md`
5. `api/internal-api-contract-design.md`
6. `api/internal-execution-schema.md`
7. `data/connection-profile-contract.md`
8. `foundation/functional-dod-matrix.md`

### 3. 런타임 구현 시작
1. `foundation/release-scope.md`
2. `runtime/runtime-node-support-matrix.md`
3. `api/internal-api-contract-design.md`
4. `api/internal-execution-schema.md`
5. `runtime/runtime-event-schema.md`
6. `runtime/runtime-execution-logging-architecture.md`
7. `modules/my-backend.md`

### 4. 프론트엔드 구현 시작
1. `foundation/release-scope.md`
2. `api/control-plane-api-baseline.md`
3. `api/api-spec.md`
4. `data/flow-owned-data-definition-roles.md`
5. `guides/frontend-development-guide.md`
6. `guides/ui-component-spec.md`
7. `modules/my-console.md`

### 5. 테스트/검증 준비
1. `foundation/functional-dod-matrix.md`
2. `foundation/traceability-spec.md`
3. `quality/qa-testing-strategy.md`
4. `quality/test-fixture-baseline.md`
5. `api/error-response-baseline.md`
6. `runtime/runtime-event-schema.md`

## Folder Guide
- `foundation`: 범위, 우선순위, 착수 기준, DoD, 추적성 같은 상위 기준 문서
- `api`: 외부 API, 내부 계약, DTO, 오류 체계
- `runtime`: 런타임 동작, 이벤트, 노드 지원 범위, 운영/성능 설계
- `data`: DataDefinition, Flow binding, Connection, 샘플 데이터 구조
- `security`: 인증, 권한, 암호화, 보안 운영
- `quality`: 테스트 전략, fixture, 운영 준비 점검표
- `guides`: 구현/운영/사용 가이드성 문서
- `modules`: 모듈별 역할과 상세 설계

## Module Specifications
- [Module Index](modules/README.md): 각 모듈별 역할, 현재 상태, 목표 범위
  - [my-console](modules/my-console.md)
  - [my-console-backend](modules/my-console-backend.md)
  - [api-gw](modules/api-gw.md)
  - [my-backend](modules/my-backend.md)
  - [logging-service](modules/logging-service.md)
  - [plugins](modules/plugins.md)
  - [k8s](modules/k8s.md)

## Foundation
- `foundation/product-spec.md`: 제품 개발 기준 범위와 목표 상태
- `foundation/release-scope.md`: 이번 프로그램의 단일 릴리즈 범위 기준
- `foundation/system-overview-and-operations.md`: 전체 모듈 구성, 처리 흐름, 트랜잭션, 운영 절차 통합 개요
- `foundation/development-readiness-checklist.md`: 개발 착수 전 보완 체크리스트
- `foundation/development-gap-analysis.md`: 개발 착수 전 P0/P1 보완 backlog와 충돌 항목 정리
- `foundation/p0-development-start-checklist.md`: 개발 착수 전 먼저 닫아야 할 P0 실행 체크리스트
- `foundation/development-execution-backlog.md`: 개발 착수 전후 세션 단위 실행 backlog
- `foundation/traceability-spec.md`: 기능-문서-테스트 추적 규격
- `foundation/functional-dod-matrix.md`: 기능별 완료 기준과 최소 테스트 기준

## API
- `api/api-spec.md`: 우선 구현 runtime/deployment API 계약 기준선
- `api/control-plane-api-baseline.md`: 제어 평면 CRUD/API 기준선
- `api/api-dto-baseline.md`: 우선 구현 API DTO/응답 구조 기준선
- `api/error-response-baseline.md`: 공통 오류 응답 구조와 코드 기준선
- `api/global-error-code-map.md`: 전역 오류 코드 체계
- `api/internal-api-contract-design.md`: 제어 평면과 데이터 평면 간 내부 실행 계약
- `api/internal-execution-schema.md`: 내부 실행 payload schema 기준선

## Runtime
- `runtime/runtime-operations-spec.md`: 런타임 운영/상태 조회 목표 규격
- `runtime/runtime-event-schema.md`: runtime 실행 이벤트 단일 schema 기준선
- `runtime/runtime-node-support-matrix.md`: 이번 프로그램의 지원/비지원 노드 범위
- `runtime/runtime-execution-logging-architecture.md`: 실행 로그 비동기 처리 아키텍처
- `runtime/runtime-schedule-distribution-architecture.md`: 스케줄 분산 구조 설계
- `runtime/node-catalog-schema.md`: 노드 카탈로그 메타데이터 규격
- `runtime/expression-function-library.md`: 표준 표현식 함수 가이드
- `runtime/data-lifecycle-design.md`: 데이터 수명주기 설계
- `runtime/performance-scalability-design.md`: 성능 및 확장성 설계

## Data
- `data/data-definition-structure-metadata-guide.md`: `DataDefinition` 구조 메타데이터 설명
- `data/flow-owned-data-definition-roles.md`: Flow 중심 DataDefinition 역할 설계
- `data/data-definition-import-from-db.md`: DB 메타데이터 기반 DataDefinition 초안 생성 설계
- `data/connection-profile-contract.md`: Connection 유형별 저장/전달 계약
- `data/mysql-postgresql-master-detail-project-sample.md`: 샘플 프로젝트 구조
- `data/mysql-postgresql-master-detail-connections-sample.md`: 샘플 connection import 데이터

## Security
- `security/security-authority-matrix.md`: 권한 매트릭스
- `security/security-identity-design.md`: 인증/식별 설계
- `security/security-encryption-design.md`: 민감 정보 암호화 설계
- `security/security-operations-guide.md`: 보안 운영 가이드

## Quality
- `quality/qa-testing-strategy.md`: 테스트 전략
- `quality/test-fixture-baseline.md`: API/Event fixture 최소 세트와 naming 규칙
- `quality/production-readiness-checklist.md`: 상용 배포 전 통합 점검표
- `quality/ha-dr-operations.md`: HA/DR 운영 규격
- `quality/deployment-compatibility-guide.md`: 배포 호환성 정책

## Guides
- `guides/frontend-development-guide.md`: 프론트엔드 개발 가이드
- `guides/my-console-accessibility-checklist.md`: `my-console` 접근성 구현 체크리스트
- `guides/my-console-api-interaction-sequence-spec.md`: `my-console` 페이지/API 호출 순서 및 invalidate 기준
- `guides/my-console-component-contracts.md`: `my-console` 공통 컴포넌트 prop/event 계약
- `guides/my-console-component-prop-event-matrix.md`: `my-console` 컴포넌트 prop/event/default/책임 매트릭스
- `guides/my-console-error-message-mapping.md`: `my-console` API 오류 코드별 사용자 문구/처리 기준
- `guides/my-console-field-validation-rules.md`: `my-console` 필드 validation/default/placeholder 규칙
- `guides/my-console-flow-editor-interaction-spec.md`: `my-console` Flow Editor 상호작용 규칙
- `guides/my-console-flow-editor-node-contracts.md`: `my-console` Flow Editor node/edge/inspector 세부 계약
- `guides/my-console-frontend-code-structure-spec.md`: `my-console` 프론트 코드 구조와 hook/query/mapper 기준
- `guides/my-console-page-acceptance-checklist.md`: `my-console` 페이지별 구현 완료 체크리스트
- `guides/my-console-page-component-tree-spec.md`: `my-console` 페이지별 container/presenter 컴포넌트 트리
- `guides/my-console-page-layout-wire-spec.md`: `my-console` 페이지 섹션 배치와 wire 수준 구조
- `guides/my-console-page-presentation-attributes-spec.md`: `my-console` 페이지별 표현 속성/위치/widget/payload/dirty 기준
- `guides/my-console-page-field-matrix.md`: `my-console` 페이지별 필드 표시/편집/검증 기준
- `guides/my-console-page-entity-view-model-spec.md`: `my-console` 페이지별 entity/DTO/view-model/prop 계약
- `guides/my-console-page-state-machine-spec.md`: `my-console` 페이지 상태 머신 기준
- `guides/my-console-page-test-scenario-matrix.md`: `my-console` 페이지별 구현 검증 시나리오
- `guides/my-console-page-state-transition-spec.md`: `my-console` 페이지 상태 전이 및 에러 처리 기준
- `guides/my-console-query-mutation-spec.md`: `my-console` React Query key/options/invalidate 기준
- `guides/my-console-routing-ia-spec.md`: `my-console` 라우팅/정보구조/breadcrumb 기준
- `guides/my-console-table-and-form-behavior-spec.md`: `my-console` 목록/폼 상호작용 기준
- `guides/my-console-typescript-interface-spec.md`: `my-console` TypeScript interface shape 기준
- `guides/my-console-ui-copy-spec.md`: `my-console` empty/confirm/toast/helper copy 기준
- `guides/ui-component-spec.md`: UI 컴포넌트 상세 명세
- `guides/api-gw-extension-guide.md`: Envoy 확장 가이드
- `guides/user-manual-flow-designer.md`: Flow 설계/테스트/배포 사용자 가이드
- `guides/ollama-commercial-model-guide.md`: 상업 사용 가능 모델 검토 메모
