# High Availability (HA) & Disaster Recovery (DR) Operations

**Status**: [Draft]
**Related Modules**: Infrastructure, Database, Redis, MinIO

## 1. 개요
본 문서는 NexioOne 시스템의 인프라 구성 요소에 대한 고가용성(HA) 아키텍처와 장애 발생 시의 재해 복구(DR) 목표 및 절차를 정의한다.

## 2. 복구 목표 (RTO / RPO)
| 컴포넌트 | 저장 데이터 | RPO (데이터 유실 허용 한계) | RTO (복구 목표 시간) |
| :--- | :--- | :--- | :--- |
| **Relational DB** | 프로젝트, Flow 메타, 유저 정보 | 5분 이내 (Point-in-Time) | 1시간 이내 |
| **Redis** | 세션, Flow 캐시, Rate Limit, 분산 락 | N/A (캐시 데이터) | 10분 이내 (자동 Failover) |
| **Object Storage**| Flow 아카이브, 대용량 페이로드, 실행 로그 | 1시간 이내 | 2시간 이내 |
| **RabbitMQ** | 비동기 실행 이벤트 로그 | 메세지 유실 Zero (DLQ) | 15분 이내 |

## 3. 고가용성 (HA) 구성
- **Application Pods**: K8S Deployment 기반으로 최소 3 Replica 이상 구성하며, Anti-Affinity를 적용해 다른 워커 노드(VM)에 분산 배치한다.
- **Database**: Primary-Replica 구조로 구성하며, 자동 Failover(예: AWS RDS Multi-AZ 또는 Patroni)를 지원한다.
- **Redis**: Redis Sentinel 또는 Redis Cluster 모드로 구성하여 마스터 노드 장애 시 즉각적인 승격(Promotion)이 이루어지도록 한다.
- **RabbitMQ**: Quorum Queue를 사용하여 노드 장애 시에도 메시지 복제본을 통해 가용성을 유지한다.

## 4. 백업 및 복구 (Backup & Restore) 정책

### 4.1 정기 백업
- **DB 백업**: 매일 새벽 02:00(시스템 시간 기준) 전체 풀 백업(Snapshot) 수행, 5분 주기로 WAL(트랜잭션 로그) 백업.
- **Object Storage 백업**: MinIO/S3의 경우 버전 관리(Versioning)를 활성화하고, 매일 1회 원격 스토리지(Cross-Region)로 비동기 복제한다.

### 4.2 장애 복구 절차 (Runbook)
1. **DB 장애(Primary Down)**:
   - 클러스터 매니저가 자동으로 Replica를 Primary로 승격시킨다.
   - 어플리케이션(my-console-backend)은 Connection Pool의 유효하지 않은 커넥션을 폐기하고 신규 Primary로 자동 재연결한다.
2. **데이터 오염에 의한 복구 (Point-in-Time Recovery)**:
   - 관리자 실수로 데이터가 삭제된 경우, 최근 풀 백업 + WAL 로그를 조합하여 장애 직전 시점(PITR)으로 신규 DB 인스턴스를 복원한다.
   - 복원된 DB 인스턴스의 엔드포인트를 애플리케이션의 환경 변수(Secret)에 업데이트 후 파드를 재시작(Rolling Update)한다.