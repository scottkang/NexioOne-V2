# Quality Assurance & Testing Strategy Detailed Design

**Status**: [Draft/Design]
**Related Modules**: All Modules

## 1. 개요
본 문서는 NexioOne 제품 개발의 전 과정에서 기능적 정확성과 비기능적 안정성을 검증하기 위한 종합 테스트 전략을 기술한다.

## 2. 테스트 계층 구조 (Test Pyramid)

| 테스트 유형 | 도구 및 프레임워크 | 검증 범위 | 수행 시점 |
| :--- | :--- | :--- | :--- |
| **Unit Test** | JUnit 5, Vitest | 개별 메서드, 컴포넌트 로직 | 매 빌드 시 (CI) |
| **Slice Test** | @WebMvcTest, @DataJpaTest | API 엔드포인트, DB 접근 레이어 | PR 생성 시 |
| **Integration Test** | Spring Integration Test | 모듈 내 서비스 간 연동 | PR 생성 시 |
| **E2E Test** | Playwright, Testcontainers | 전체 사용자 시나리오 (UI ~ BE) | 배포 전 (Staging) |
| **Contract Test** | Spring Cloud Contract | 모듈 간 API 규약 정합성 | 모듈 간 변경 시 |

## 3. 테스트 환경 및 데이터 관리

### 3.1 환경 구분
- **Development**: 로컬 개발자 환경 (In-memory DB, Mock 서비스).
- **CI (GitHub Actions)**: 자동화된 빌드 및 테스트 환경.
- **Staging (QA)**: 운영과 유사한 데이터 및 네트워크 환경 (Containerized Infra).

### 3.2 테스트 데이터 전략
- **Seeding**: Flyway 또는 별도 SQL 스크립트를 통해 테스트용 기초 데이터 로드.
- **Sandboxing**: 프로젝트별 독립적인 테넌트 ID를 부여하여 테스트 데이터 간섭을 방지한다.
- Fixture 기준선은 `test-fixture-baseline.md`를 따른다.

## 4. 외부 시스템 연동 및 모킹(Mocking) 전략

| 연동 대상 | 전략 | 도구 |
| :--- | :--- | :--- |
| **Database** | Lightweight Instance | Testcontainers (PostgreSQL, MySQL) |
| **External API** | HTTP Mock Server | MockServer, WireMock |
| **Messaging** | Embedded/Container MQ | Testcontainers RabbitMQ |
| **File System** | Temporary Directory | JUnit 5 @TempDir |
| **Mainframe/Legacy**| Protocol Stubbing | Custom Mock Runner |

## 5. 핵심 검증 시나리오

1.  **Control Plane CRUD**: 프로젝트, Flow, DataDefinition, Connection, Binding CRUD가 문서 계약대로 동작하는지 확인.
2.  **Runtime Dry-Run / Execute-Stub**: 지원 노드 범위와 비지원 노드 오류가 문서 기준과 일치하는지 확인.
3.  **Async Execution Control**: async accepted/status/idempotency conflict가 외부 API와 내부 계약대로 동작하는지 확인.
4.  **Logging Pipeline**: runtime 이벤트가 logging-service에 적재되고 read model이 갱신되는지 확인.

이번 프로그램 제외:
- XA/2PC 실구현 검증
- distributed scheduler 실운영 검증
- OTEL trace link end-to-end 검증
- real connector 연동 검증

## 5.1 Contract Test Ownership
- `my-console-backend <-> my-backend`: producer responsibility `my-console-backend`, consumer verification `my-backend`
- `my-backend <-> logging-service`: producer responsibility `my-backend`, consumer verification `logging-service`
- 공통 fixture source는 `test-fixture-baseline.md` naming 규칙을 따른다.

## 5.2 실행 단위별 테스트 책임
| Feature Group | Primary Module | Test Types | Contract Ownership | Evidence Source |
| :--- | :--- | :--- | :--- | :--- |
| Auth / Project / Flow / DataDefinition / Connection / Binding | `my-console-backend` | API, Slice, Service | 단일 모듈 책임 | `fixtures/api/auth/`, `fixtures/api/projects/`, `fixtures/api/flows/`, `fixtures/api/data-definitions/`, `fixtures/api/connections/` |
| Deployment API | `my-console-backend` | API, Service, Contract | 외부 API producer `my-console-backend` | `fixtures/api/deployments/` |
| Dry-Run / Execute-Stub / Execution Status | `my-console-backend`, `my-backend` | API, Integration, Contract | producer `my-console-backend`, consumer verification `my-backend` | `fixtures/api/runtime/`, `fixtures/runtime/` |
| Runtime Event Publish | `my-backend` | Producer Contract | producer `my-backend` | `fixtures/events/runtime/` |
| Logging Consume / Persist | `logging-service` | Consumer Contract, Persistence | consumer verification `logging-service` | `fixtures/events/runtime/` |

### 5.3 Fixture Seed Rule
- 공통 seed 데이터셋은 `fixtures/seeds/control-plane-minimal-seed.json`을 기준으로 한다.
- feature fixture는 seed에 정의된 `projectId`, `flowId`, `dataDefinitionId`, `connectionId`, `deploymentId`를 재사용한다.
- contract/integration 테스트는 feature fixture와 event fixture가 같은 식별자 세트를 공유해야 한다.

## 6. 테스트 자동화 및 파이프라인 (CI/CD 연동)

- **PR Guard**: 모든 Pull Request는 최소 80% 이상의 코드 커버리지를 달성해야 하며, 모든 단위/통합 테스트를 통과해야 한다.
- **Smoke Test**: 매 배포 후 핵심 기능 가용성을 즉시 확인하는 자동화 스크립트를 실행한다.
- **Mutation Test**: PITest 등을 활용하여 테스트 케이스의 유효성 및 견고함을 측정한다.

## 7. 참조
- `docs/spec/foundation/functional-dod-matrix.md`
- `docs/spec/quality/test-fixture-baseline.md`
