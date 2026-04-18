# Data Lifecycle & Persistence Detailed Design

**Status**: [Draft/Design]
**Related Modules**: [my-console-backend](../modules/my-console-backend.md), [logging-service](../modules/logging-service.md), [my-backend](../modules/my-backend.md)

## 1. 개요
본 문서는 NexioOne 시스템의 데이터 정합성을 유지하고, 대용량 데이터로 인한 성능 저하를 방지하기 위한 데이터 관리 전략을 정의한다. DB 스키마 변경 관리, 데이터 보관 정책, 파티셔닝 및 백업 방안을 포함한다.

## 2. DB 마이그레이션 전략 (Flyway)

모든 DB 스키마 변경은 소스 코드와 함께 버전 관리되어야 한다.

- **도구**: **Flyway** (Spring Boot 통합)
- **위치**: `src/main/resources/db/migration`
- **파일 명명 규칙**: `V{Version}__{Description}.sql` (예: `V1.0.1__add_trace_id_to_record.sql`)
- **운영 원칙**:
    - 이미 배포된 마이그레이션 파일은 수정할 수 없으며, 변경이 필요한 경우 새로운 버전 파일을 생성한다.
    - `Undo` 스크립트는 지원하지 않으며, 롤백이 필요한 경우 역방향 변경을 담은 새로운 버전(`V+1`)을 배포한다.

## 3. 대용량 데이터 관리 (Partitioning)

`logging-service`와 같이 쓰기 부하가 높고 데이터가 급증하는 테이블은 **Time-based Partitioning**을 적용한다.

### 3.1 파티셔닝 대상 및 기준
- **대상 테이블**: `TB_EXECUTION_EVENT`, `TB_EXECUTION_RECORD`, `TB_EXECUTION_STEP`
- **기준 컬럼**: `occurred_at` (또는 `started_at`)
- **단위**: **일별(Daily)** 또는 **월별(Monthly)** 파티션 생성.

### 3.2 파티션 관리 자동화
- **Partition Pruning**: 쿼리 시 조건에 맞는 파티션만 스캔하도록 최적화한다.
- **Auto Creation**: 다음 주기의 파티션을 미리 생성하는 백그라운드 작업을 수행한다.

## 4. 데이터 보관 및 정리 정책 (Housekeeping)

데이터의 성격에 따라 보관 기간과 처리 방식을 차별화한다.

| 데이터 유형 | 보관 기간 (Retention) | 처리 방식 | 비고 |
| :--- | :--- | :--- | :--- |
| **마스터/설정 데이터** | 영구 (또는 수동 삭제) | Soft Delete | Flow, Connection 정보 등 |
| **운영 로그 (Log)** | 30일 | **Partition Drop** | 실행 이력, 이벤트 원문 등 |
| **감사 로그 (Audit)** | 1년 | Archive to Storage | 관리자 작업 이력 |
| **임시 컨텍스트 (L2/L3)**| 실행 완료 후 즉시 | Delete / Purge | Redis 및 임시 파일 |

### 4.1 Housekeeper 컴포넌트
- **역할**: 설정된 Retention 정책을 초과한 데이터를 자동으로 정리한다.
- **동작**: 매일 새벽 부하가 적은 시간에 실행되며, 파티션을 Drop 하거나 Batch Delete를 수행한다.

## 5. 백업 및 복구 (Backup & DR)

### 5.1 데이터베이스 백업
- **Snapshot**: 클라우드/K8S 환경의 경우 매일 1회 전체 스냅샷 자동 생성을 권장한다.
- **Point-in-Time Recovery**: 실시간 복구를 위한 WAL(Write Ahead Log) 보관 전략을 수립한다.

### 5.2 오브젝트 스토리지 (MinIO/S3)
- **Versioning**: 런타임 결과물(Archive)에 대한 버전 관리를 활성화한다.
- **Lifecycle Policy**: 오래된 결과물을 저비용 저장소로 이동하거나 자동 삭제하도록 정책을 설정한다.

## 6. 데이터 마이그레이션 및 초기화 (Seeding)

- **Test Fixtures**: 개발 및 테스트 환경을 위해 `src/test/resources/sql`에 테스트 데이터 스크립트를 관리한다.
- **Initial Data**: 시스템 기동 시 필요한 기본 코드 및 초기 관리자 정보는 Flyway의 초기 버전(`V1__init.sql`)에 포함한다.
