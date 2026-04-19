# k8s Detailed Design (Helm Chart & Resource Management)

**Status**: [Draft/Design]
**Parent Module**: [k8s](k8s.md)

## 1. 개요
본 문서는 NexioOne 시스템을 Kubernetes 클러스터에 안정적으로 배포하기 위한 Helm Chart 구조, 서비스별 리소스 할당(Resource Quota), 오토스케일링(HPA) 및 환경 설정 관리 전략을 정의한다.

## 2. Helm Chart 구조 (NexioOne Umbrella Chart)
개별 마이크로서비스를 Sub-chart로 관리하거나, 통합 Helm Chart를 통해 전체 스택을 제어한다.

```text
nexioone-chart/
├── Chart.yaml              # 차트 메타데이터
├── values.yaml             # 전체 기본 설정 (이미지 태그, 공통 env 등)
├── templates/
│   ├── _helpers.tpl        # 공통 이름/라벨 템플릿
│   ├── ingress.yaml        # api-gw(Envoy) 또는 클러스터 Ingress 설정
│   └── secrets.yaml        # DB 암호, API Key 등 민감 정보 (External Secrets 연동 권장)
├── charts/                 # Sub-charts (독립 배포 가능)
│   ├── my-console/
│   ├── my-console-backend/
│   ├── my-backend/         # 워커 그룹 (HPA 집중 관리)
│   └── logging-service/
└── values-prod.yaml        # 운영 환경 특화 설정
```

## 3. 서비스별 리소스 설계 (Resource Limits & Requests)
안정적인 운영을 위해 최소 보장(Request)과 최대 제한(Limit)을 설정한다. (Java 서비스의 경우 JVM Heap 설정과 연동)

| 서비스명 | CPU Request | CPU Limit | Memory Request | Memory Limit | JVM Heap (Xmx) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `my-console` | 100m | 200m | 128Mi | 256Mi | N/A (Nginx) |
| `my-console-backend`| 500m | 1000m | 1Gi | 2Gi | 1.5Gi |
| `my-backend` | 1000m | 2000m | 2Gi | 4Gi | 3Gi |
| `logging-service` | 300m | 1000m | 512Mi | 1Gi | 768Mi |
| `api-gw` (Envoy) | 200m | 1000m | 256Mi | 512Mi | N/A |

## 4. 오토스케일링 전략 (HPA)
워크플로우 실행 부하가 집중되는 `my-backend`를 중심으로 자동 확장을 구성한다.

- **대상 모듈**: `my-backend`
- **Metric**: CPU Utilization (Target 70%) 또는 Custom Metric (RabbitMQ Queue Depth).
- **Min Replicas**: 2 (고가용성 확보)
- **Max Replicas**: 10 (클러스터 노드 리소스 한도 내)

```yaml
# HPA 예시
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nexio-my-backend-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nexio-my-backend
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

## 5. 환경 설정 및 비밀 관리 (ConfigMap & Secrets)

### 5.1 ConfigMap
- 공통 환경 변수 (Redis/RabbitMQ/MinIO 엔드포인트).
- 로그 레벨 및 성능 튜닝 파라미터.
- `my-backend`의 경우 `RUNTIME_CONFIG_CACHE_REDIS_ENABLED` 등의 플래그 관리.

### 5.2 Secrets (External Secrets Operator 추천)
- **Database Credentials**: Postgres/MySQL 접속 정보.
- **Messaging Credentials**: RabbitMQ 유저/암호.
- **Object Storage Keys**: MinIO Access/Secret Key.
- **SDS Certificates**: `api-gw`용 TLS 인증서.

## 6. 스토리지 전략 (PVC)
- **Stateless 서비스**: `my-console`, `my-backend`, `my-console-backend`는 가급적 PVC를 사용하지 않고 외부 DB/Object Storage를 활용.
- **Local Cache**: 플러그인 JAR 로딩을 위한 `emptyDir` 활용 (Pod 재시작 시 초기화되나 MinIO에서 재다운로드 가능).

## 7. 가용성 및 배포 전략
- **Rolling Update**: `maxUnavailable: 25%`, `maxSurge: 25%` 설정으로 무중단 배포 보장.
- **Affinity & Anti-Affinity**: `my-backend` Pod들이 서로 다른 노드에 분산 배치되도록 설정 (`podAntiAffinity`).
- **Liveness/Readiness Probe**:
  - `my-backend`: `/health` 엔드포인트 체크.
  - `my-console-backend`: Spring Boot Actuator `/actuator/health` 연동.

## 8. 구현 로드맵
1.  **Phase 1**: 개별 서비스별 기본 Deployment/Service Manifest 작성.
2.  **Phase 2**: Umbrella Chart로 통합 및 `values.yaml` 환경 분리.
3.  **Phase 3**: HPA 및 리소스 쿼타 적용 및 부하 테스트.
4.  **Phase 4**: ArgoCD 등 GitOps 도구를 통한 배포 자동화 파이프라인 구축.
