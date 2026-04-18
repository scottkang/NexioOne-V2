# Module Specifications

이 디렉토리는 NexioOne 프로젝트를 구성하는 각 모듈의 상세 명세서를 포함합니다.

## 시스템 아키텍처 그룹

### 1. Control Plane (제어 평면)
시스템의 설정, 관리 및 오케스트레이션을 담당하는 모듈입니다.
- [my-console](my-console.md): 프론트엔드 웹 애플리케이션 (React) - **[Not Implemented]**
- [my-console-backend](my-console-backend.md): 관리 및 제어용 백엔드 API (Spring Boot) - **[Not Implemented]**

### 2. Data Plane (데이터 평면)
실제 트래픽을 처리하고 워크플로우를 실행하는 런타임 모듈입니다.
- [my-backend](my-backend.md): 핵심 워크플로우 실행 엔진 (Spring Boot) - **[Not Implemented]**
  - [Detailed Design: Execution Context & Tiered Storage](my-backend-detailed-design.md)
- [api-gw](api-gw.md): Envoy 기반 API 게이트웨이 - **[Not Implemented]**
  - [Detailed Design: Envoy xDS & Filter](api-gw-detailed-design.md)

### 3. Support Services (지원 서비스)
실행 결과 수집, 영속화 및 부가 기능을 제공하는 서비스입니다.
- [logging-service](logging-service.md): 런타임 실행 결과 비동기 로깅 서비스 - **[Not Implemented]**
  - [Detailed Design: Event Consumption & Persistence](logging-service-detailed-design.md)
- [plugins](plugins.md): 런타임 컴포넌트 계열(`my-backend-component-api`, `my-backend-component`) 및 플러그인 어댑터 - **[Skeleton/Design]**
  - [Detailed Design: Custom ClassLoader & Lifecycle](plugins-detailed-design.md)

### 4. Infrastructure (인프라)
시스템이 구동되는 환경 및 배포 구성입니다.
- [k8s](k8s.md): Kubernetes 배포 및 운영 구성 - **[Not Implemented]**
  - [Detailed Design: Helm Chart & Resource Management](k8s-detailed-design.md)

## Future Roadmap (확장 계획)
다음 모듈들은 차기 프로그램 범위에서 추가될 예정입니다.
- **nexio-monitoring**: 메트릭, 분산 트레이싱 및 실시간 관제.
- **nexio-audit**: 보안 및 규정 준수를 위한 사용자 활동 감사 로그.
- **nexio-dev-portal**: 외부 개발자를 위한 API 명세 및 키 관리 포털.
- **nexio-batch**: 대량 데이터 처리 및 통계 생성 배치.
- **nexio-secret-manager**: 민감 데이터 암호화 관리 전용 모듈.

## 문서 규칙
- 각 모듈 명세서는 해당 모듈의 역할, 주요 기능, 핵심 아키텍처 및 연관 설계 문서를 포함한다.
- Mermaid 다이어그램 사용 시 라벨 줄바꿈은 `<br/>`를 사용한다.
