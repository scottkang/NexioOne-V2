# Performance, Scalability & SLO Detailed Design

**Status**: [Draft/Design]
**Related Modules**: [my-gw](../modules/api-gw.md), [my-backend](../modules/my-backend.md), [logging-service](../modules/logging-service.md)

## 1. 개요
본 문서는 NexioOne 시스템의 서비스 수준 목표(SLO)를 정의하고, 대규모 데이터 처리 및 급격한 트래픽 증가 상황에서도 안정성을 유지하기 위한 기술적 전략을 기술한다.

## 2. 서비스 수준 목표 (SLO/SLI)

시스템의 가용성과 성능을 측정하기 위한 핵심 지표를 다음과 같이 설정한다.

| 서비스 그룹 | 주요 지표 (SLI) | 목표 수준 (SLO) | 측정 기준 |
| :--- | :--- | :--- | :--- |
| **API Gateway** | Request Latency (P95) | < 50ms | GW 유입 ~ 업스트림 전달 |
| **Control Plane** | Console API Latency (P95) | < 200ms | UI 응답 시간 |
| **Execution Engine**| Flow Execution Success Rate | > 99.9% | 전체 실행 시도 대비 성공 |
| **Logging Service** | Ingestion Delay | < 5s | 이벤트 발생 ~ 조회 가능 시점 |
| **All Services** | Availability | 99.9% (3 nines) | 연간 다운타임 8.76시간 이내 |

## 3. 부하 분산 및 확장 전략 (Scalability)

### 3.1 무상태 아키텍처 (Statelessness)
- 모든 서비스(Console-Backend, Worker, Logging)는 무상태(Stateless)를 유지하여 수평적 확장(Horizontal Scaling)이 가능하도록 설계한다.
- 세션 및 실행 컨텍스트는 로컬 메모리가 아닌 Redis/Object Storage에 외부화한다.

### 3.2 오토스케일링 (HPA)
- **K8S 환경**: CPU 및 Memory 사용량을 기준으로 Horizontal Pod Autoscaler (HPA)를 적용한다.
- **Worker 특화**: `my-backend`는 처리 대기 중인 메시지 수(Queue Length)를 기준으로 스케일 아웃을 트리거한다.

### 3.3 분산 트랜잭션 부하 관리 (Seata TC Scaling)
- Seata TC 서버는 Raft 또는 DB-based HA 모드로 클러스터링하여 고가용성을 확보한다.

## 4. 부하 제어 및 탄력성 (Resilience & Backpressure)

### 4.1 서킷 브레이커 (Circuit Breaker)
- 외부 시스템(Target DB, API) 호출 시 Resilience4j를 사용하여 장애 전파를 방지한다.
- 에러율 50% 초과 또는 지연 시간 2초 초과 시 서킷을 개방(Open)하고 Fallback 로직을 수행한다.

### 4.2 배압 제어 (Backpressure)
- **RabbitMQ Consumer**: prefetch 개수를 설정(예: 10~50)하여 워커가 감당할 수 있는 만큼의 메시지만 가져오도록 조절한다.
- **API Rate Limiting**: `api-gw` 레벨에서 테넌트/프로젝트별 초당 호출 수(TPS)를 제한한다.

## 5. 캐싱 및 리소스 최적화

### 5.1 Redis 캐싱 전략
- **Config Cache**: 배포된 Flow 및 Connection 정보는 Redis에 캐싱하여 DB 부하를 최소화한다.
- **TTL 정책**:
    - 실행 컨텍스트: 실행 완료 후 30분(복구용) 유지 후 자동 삭제.
    - 정적 코드/설정: 무기한 유지하되 배포 알림 시 무효화(Invalidation).

### 5.2 커넥션 풀링 (Connection Pooling)
- **HikariCP**: 각 워커 인스턴스당 DB 커넥션 수를 최적화(예: default 10~20)한다.
- **Idle Timeout**: 불필요한 커넥션 점유를 방지하기 위해 짧은 유휴 타임아웃을 설정한다.

## 6. 성능 테스트 가이드라인
- **Load Test**: 목표 TPS의 2배 수준에서 부하 테스트(nGrinder, JMeter)를 정기적으로 수행한다.
- **Stress Test**: 자원 한계치까지 부하를 주어 시스템 중단 지점을 파악하고 우아한 퇴보(Graceful Degradation) 여부를 확인한다.
