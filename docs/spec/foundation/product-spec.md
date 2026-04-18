# NexioOne Product Spec (Development Baseline)

## 1. 목적
- 본 문서는 NexioOne 제품 개발의 기준 범위와 우선 구현 대상을 정의한다.
- 기준 시점: `2026-04-18`

## 1.1 해석 규칙
- 이번 프로그램의 포함/제외 범위 판정은 `docs/spec/foundation/release-scope.md`를 단일 기준으로 사용한다.
- 본 문서는 제품 방향과 모듈별 목표 상태를 설명하는 상위 기준 문서이며, 범위 충돌 시 `release-scope.md`를 우선한다.

## 2. 제품 범위
- 프론트엔드: `my-console`
- 제어 평면 백엔드: `my-console-backend`
- 데이터 평면 런타임: `my-backend`
- API 게이트웨이: `api-gw`
- 실행 로그 수집 서비스: `logging-service`

## 3. 현재 상태
- 전체 모듈 구현 상태: **Not Implemented**
- 현재 저장소의 `docs/spec`는 구현 결과 문서가 아니라 개발 목표와 계약 초안 문서로 해석한다.

## 4. 이번 개발 프로그램의 In Scope
- 인증/인가 및 RBAC 기반 제어 평면 API
- 프로젝트/Flow/DataDefinition/Connection/Deployment 관리 API
- Flow 설계 데이터를 저장하고 배포 단위로 스냅샷하는 제어 평면
- `my-console-backend -> my-backend` 실행 요청 계약
- `my-backend`의 `dry-run`, `execute-stub`, 상태 조회 API
- `my-backend -> logging-service` 실행 이벤트 전달 계약
- `api-gw`의 제어 평면 연동 구조 및 기본 라우팅/보안 확장 포인트
- 운영/배포/오류 응답/테스트/추적성 문서 기준선

상세 릴리즈 범위는 `docs/spec/foundation/release-scope.md`를 우선 기준으로 사용한다.

## 5. 이번 개발 프로그램의 Out of Scope
- 운영 배포 완료 판정
- Program Exit, release sign-off, 운영 증적 완료 상태 선언
- 실연결 커넥터의 완전한 프로덕션 구현
- 고도화된 분산 스케줄링, 자동 DLQ 재주입, 장기 SLO 튜닝
- 대용량 스트리밍, L1/L2/L3 계층 저장소, SPLIT/JOIN, WAIT, FLOW_TASK의 완전 구현

## 5.1 차기 프로그램 후보 범위
- real connector 기반 실제 실행
- Redis / Quartz 기반 분산 스케줄링 및 config sync 고도화
- XA/2PC 또는 동등 수준의 분산 트랜잭션
- object storage archive, DLQ 재주입, 고급 운영 자동화
- 고급 node type (`WAIT`, `SPLIT/JOIN`, `FLOW_TASK`, file 계열, batch 계열)

## 6. 모듈별 목표 상태
| 모듈 | 현재 | 이번 프로그램 목표 |
|---|---|---|
| `my-console` | Not Implemented | 설계/운영 UI 기초 화면 및 API 연동 |
| `my-console-backend` | Not Implemented | 제어 평면 API, 배포/실행 오케스트레이션 |
| `my-backend` | Not Implemented | `dry-run`, `execute-stub`, 상태 조회, 내부 실행 계약 수용 |
| `api-gw` | Not Implemented | 기본 아키텍처, 설정 동기화 인터페이스 |
| `logging-service` | Not Implemented | 실행 이벤트 수신 및 기초 저장 모델 |

## 7. 우선 구현 순서
1. 제어 평면 데이터 모델 및 인증/권한
2. 배포/실행 API와 내부 실행 계약
3. `my-backend`의 `dry-run`, `execute-stub`, runtime 조회 API
4. `logging-service` 이벤트 계약 및 저장 모델
5. `api-gw` 설정 동기화 및 기본 프록시 구조

## 8. 완료 기준
- `release-scope.md`와 모듈 문서가 같은 현재/차기 범위 해석을 갖는다.
- In Scope 모듈별 최소 API 계약이 문서와 테스트로 고정된다.
- `dry-run`, `execute-stub`, deployment, runtime 조회 API의 request/response/error 형식이 문서화된다.
- `my-console-backend -> my-backend` payload 및 비동기/idempotency 규칙이 문서화된다.
- 구현 상태를 과장하는 표현(`Implemented`, `Program Exit Ready`, `Ready For Program Exit`)이 제거된다.

## 9. 참조
- `docs/spec/foundation/release-scope.md`
- `docs/spec/foundation/development-readiness-checklist.md`
- `docs/spec/api/api-spec.md`
- `docs/spec/api/internal-api-contract-design.md`
- `docs/spec/runtime/runtime-operations-spec.md`
