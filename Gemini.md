# NexioOne 프로젝트

## 프로젝트 개요
클라우드 SaaS 환경에서 프로젝트, API, 트랜잭션, 구성 등을 통합 관리하기 위한 웹 애플리케이션과 K8S 환경에서 실행되는 백엔드 서비스를 포함합니다.

## 프로젝트 특정 규약
- **Frontend:** React (Create React App), Tailwind CSS, React Router
- **Backend:** Spring Boot, Spring Data JPA, H2 Database (In-Memory)
- **통신:** RESTful API (JSON)

## 컨테이너 이미지 빌드 진입점
- `my-console-backend`: `cd my-console-backend && mvn -q -DskipTests -Pdocker verify`
- `my-backend`: `cd my-backend && mvn -q -DskipTests -Pdocker verify`
- `logging-service`: `cd logging-service && mvn -q -DskipTests -Pdocker verify`
- `my-console`: `cd my-console && npm run docker:build`

## 로컬 통합 실행
- 루트: `docker-compose -f compose.yaml up --build` 또는 `docker compose -f compose.yaml up --build`
- 스크립트: `./scripts/run-compose-stack.sh`
- Connection target profile 포함: `./scripts/run-compose-stack.sh --with-connection-targets`

## CI 실행 기준
- GitHub Actions 기본 러너 대상은 self-hosted mac runner(`self-hosted`, `macOS`)입니다.
- `ci.yml`, `pr-guard.yml`, `pit-mutation.yml`의 모든 job은 self-hosted mac runner label에서 실행합니다.
- CI는 세 모듈 모두 테스트와 coverage threshold 검증까지 포함합니다.
- `my-backend` 기본 CI는 경량 suite와 별도 integration suite로 분리하고, coverage threshold는 병합된 JaCoCo 결과를 별도 job에서 검증합니다.
- PR에서는 Actions summary와 자동 코멘트로 coverage 요약을 확인할 수 있고, 각 모듈 HTML report는 artifact로 다운로드할 수 있습니다.
- PIT mutation test는 기본 CI에서 제외되어 있으며, 별도 GitHub Actions workflow(`pit-mutation.yml`)를 수동(`workflow_dispatch`)으로 실행할 때만 동작합니다.
- self-hosted mac runner에서는 `actions/setup-java`의 GitHub Actions Maven cache를 사용하지 않고, runner 로컬 `~/.m2/repository` 재사용을 기본으로 합니다.

## 주요 모듈
- `/my-console/` - 웹 어플리케이션 프론트엔드 (React)
- `/my-console-backend/` - 웹 어플리케이션 Backend (Spring Boot, Admin API)
  - **API Governance**: API 엔드포인트, 정책(Rate Limit, Quota), OAS 스펙 관리
  - **Workflow Engine**: 워크플로우 데이터(JSON) 영속화 및 버전 관리
  - **Execution Engine**:
    - **API Orchestrator**: REST 호출 기반 워크플로우 매핑 및 동기/비동기 실행 제어
    - **Distributed Scheduler**: 분산 클러스터 기반 Cron 작업 관리 (Quartz/Spring Scheduler)
    - **Event Trigger Service**: MQ(Kafka/RabbitMQ) 메시지 리스너 및 sFTP 파일 감지 제어
    - **Instance Manager**: 실행 인스턴스별 상태 추적, 재시도(Retry) 및 Context 관리
  - **Connection Manager**: 외부 리소스 연결 정보 암호화 저장 및 상태 모니터링
  - **Security & RBAC**: 사용자 인증(JWT/OIDC), 권한 관리, 민감 정보 암호화(Secret)
  - **Observability**: 트랜잭션 로그 집계, 감사 로그(Audit), 시스템 상태 체크
  - **Config Sync**: API Gateway(Envoy) 및 백엔드 서비스 설정 동기화 및 배포 제어
- `/api-gw/` - API Gateway (envoy 기반)
  - **Control Plane (xDS)**: gRPC 기반 실시간 설정(Route, Cluster, Listener) 동기화
  - **External Authz**: API Key, JWT, OAuth2 기반 중앙 집중식 인증/인가 처리
  - **Global Rate Limit**: Redis 연동 기반 시스템 전역 사용량 제한 및 Quota 관리
  - **Wasm/Lua Filters**: `plugins`와 연동된 커스텀 로직 및 데이터 변환 처리
  - **Access Log Service (ALS)**: 실시간 트랜잭션 로그 수집 및 관제 시스템 전송
  - **SDS (Secret Discovery)**: TLS/SSL 인증서 동기 배포 및 자동 갱신
- `/my-backend/` - 실제 비즈니스 로직을 처리하는 Backend Service
- `/my-backend-component/` - Backend Service에서 비지니스 로직을 처리하기 위해 단계별로 실행되는 컴포넌트들의 집합
- `/my-backend-component-api/` - Backend Service와 my-console-backend에서 backend component들을 조회하거나 실행을 위해 로드하는 대 사용하는 api
- `/logging-service/` - runtime execution 결과 수집/저장 및 조회용 백엔드 서비스
- `/k8s/` - Kubernetes configuration files

## 성능 및 캐싱 전략
- **In-Memory Config Cache**: 실행 엔진(Engine/GW) 로컬 메모리에 프로젝트 및 플로우 설정을 캐싱하여 DB I/O 병목 제거
- **Config Hot-Reload**: 시스템 재시작 없이 배포 즉시 설정이 반영되는 동적 갱신 메커니즘 지원
- **Sync Architecture**:
  - **Push 기반**: `my-console` 배포 시 Redis Pub/Sub 또는 gRPC를 통해 실행 엔진에 변경 알림 전송
  - **Version Validation**: 각 설정에 버전 번호를 부여하여 캐시와 DB 간의 데이터 정합성 보장

## DB 스키마 설계

### 핵심 엔티티 관계 (ERD)
- **Project (1)** : **Flow (N)**, **DataDefinition (N)**, **ExecutionPolicy (N)**
- **Flow (1)** : **FlowNode (N)** (워크플로우 단계별 설정)
- **Connection (N)** : **Project (N)** (공유 자원으로서 다대다 또는 참조 관계)
- **Api (N)** : **Flow (1)** (API 엔드포인트와 로직 매핑)

### 주요 테이블 상세
- **`TB_PROJECT`**: 최상위 프로젝트 컨테이너 (ID, Name, Status, Description)
- **`TB_CONNECTION`**: 공유 연결 정보 (ID, Name, Type[JDBC/REST/MQ/SFTP], Config[Encrypted JSON])
- **`TB_FLOW`**: 비즈니스 로직 단위 (ID, Project_ID, Name, Flow_Data[JSON], Version)
- **`TB_FLOW_NODE`**: 플로우 내 개별 단계 (ID, Flow_ID, Type, Name, Config[JSON], UI_Position)
- **`TB_FLOW_EDGE`**: 노드 간 연결 정보 (ID, Flow_ID, Source_Node_ID, Target_Node_ID, Condition)
- **`TB_DATA_DEFINITION`**: 데이터 구조 정의 (ID, Project_ID, Name, Type[In/Out], Schema[JSON])
- **`TB_EXECUTION_POLICY`**: 실행 정책 (ID, Project_ID, Flow_ID, Type[Schedule/Trigger], Config[JSON])
- **`TB_API`**: 외부 노출 엔드포인트 (ID, Project_ID, Flow_ID, Path, Method, Auth_Type)
- **`TB_API_PRODUCT`**: API 상품 및 정책 (ID, Name, Quota, Rate Limit)

## 추가 제안 모듈 (확장 계획)
- `/nexio-monitoring/` - 메트릭, 분산 트레이싱 및 실시간 관제 모듈 (Prometheus, Grafana 연동)
- `/nexio-audit/` - 보안 및 규정 준수를 위한 사용자 활동 감사 로그 시스템
- `/nexio-dev-portal/` - 외부 개발자를 위한 API 명세(Swagger/OpenAPI) 및 키 관리 포털
- `/nexio-batch/` - 대량 데이터 처리, 통계 생성 및 데이터 클린업(Housekeeping) 배치
- `/nexio-secret-manager/` - DB 접속 정보 등 민감 데이터 암호화 관리 전용 모듈

## my-console

### 메뉴 및 상세 기능
- **Dashboard**: 실시간 트래픽 요약, 시스템 건전성 지표, 최근 작업 내역 시각화
- **API Management**
  - **API Designer**: 엔드포인트 생성, HTTP 메서드 정의, 보안 정책 설정
  - **API Product**: API 그룹화, 구독 관리, Rate Limiting 및 Quota 정책 설정
  - **API Docs**: Swagger/OpenAPI 기반 자동 문서화 및 테스트 환경
- **Project Management**
  - **Projects**: Flow, Data Definition, Schedule, Trigger 등을 포함하는 최상위 관리 단위
    - **Flow Designer**: 프로젝트 내 여러 비즈니스 로직(Flow)의 시각적 설계 및 연결
    - **Data Definitions**: 프로젝트 단위의 공통 데이터 구조 및 검증 규칙
    - **Execution Policies**: 프로젝트/Flow별 실행 정책 (Scheduler, Event Triggers) 관리
  - **Shared Connections**: 여러 프로젝트에서 재사용 가능한 외부 리소스(DB, API, MQ 등) 연결 정보
- **Monitoring & Ops**
  - **Transaction Viewer**: 트랜잭션 ID 기반 상세 추적 및 Payload 분석
  - **Deployment**: 환경별(Dev/Stage/Prod) 배포 관리 및 롤백 기능
- **Settings**
  - **Access Control**: 사용자(User), 역할(Role), 권한(Permissions) 기반 RBAC 관리
  - **Global Config**: 전역 파라미터 및 환경별 변수 관리
  - **Authentication**: OIDC, OAuth2, API Key 등 인증 프로바이더 설정

## GitHub Issue/PR Rules
- 기능 구현, 테스트 추가, 리팩터링 등 코드 변경 작업을 시작할 때는 반드시 GitHub 이슈를 먼저 생성한다.
- 이슈 생성 후 브랜치 규칙(`type/{issue-number}-{slug}`)에 맞는 작업 브랜치를 생성하고 해당 브랜치에서만 작업한다.
- 작업 완료 후에는 반드시 PR을 생성해 `main`으로 병합 가능한 상태까지 만든다.
- PR 생성 후에는 기본적으로 `gh pr merge --auto --merge <pr-number>`를 설정해, 필수 CI 체크가 모두 통과하면 자동 머지되도록 한다.
- auto-merge는 필수 체크가 모두 등록된 PR에만 설정하며, 충돌이 있거나 추가 수동 검토가 필요한 경우에는 예외로 둔다.
- PR이 머지되면 관련 작업 브랜치는 삭제한다. 저장소의 자동 삭제 설정을 기본으로 사용하고, 자동 삭제가 적용되지 않은 경우에는 수동으로 정리한다.
- PR 머지 전에는 가장 최근 CI 실행 결과를 확인해 관련 워크플로우가 정상적으로 완료되었는지 검토한다.
- PR 머지 전에는 GitHub Actions 체크 결과를 반드시 확인하고, 필수 체크가 모두 성공(`success`)한 경우에만 머지한다.
- 체크가 실패(`failure`) 또는 미완료(`pending`) 상태면 머지를 보류하고 원인을 정리한 뒤 재확인한다.
- CLI 기준으로 `gh pr checks <pr-number> --watch`로 완료까지 대기하고, `gh pr checks <pr-number>`로 최종 상태를 확인한 후 머지한다.
- 이슈 생성/수정 시 `.github/ISSUE_TEMPLATE/*.yml` 형식을 먼저 확인하고, 해당 섹션 구조에 맞춰 작성한다.
- PR 생성/수정 시 `.github/pull_request_template.md` 형식을 먼저 확인하고, 필수 섹션을 모두 채운다.
- `gh issue create|edit`, `gh pr create|edit`에서 본문은 `--body-file` 또는 heredoc 기반 멀티라인 문자열로 전달한다.
- 본문에 `\n` 문자열을 직접 넣어 개행을 표현하지 않는다.
- Mermaid 다이어그램 라벨 줄바꿈은 `\n` 대신 `<br/>`를 사용한다.
