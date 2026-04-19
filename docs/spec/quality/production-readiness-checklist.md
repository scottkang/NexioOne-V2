# Production Readiness & Unified Checklist

**Status**: [Draft/Design]
**Related Modules**: All Modules

## 1. 개요
본 문서는 NexioOne 시스템(my-console, my-console-backend, my-backend, logging-service, api-gw) 전체가 상용 환경(Production)에 배포되기 전 반드시 확인해야 할 통합 점검 항목을 정의한다.

## 2. 통합 검증 체크리스트

### 2.1 기능 및 통합 (Functional & Integration)
- [ ] UI(my-console)에서 Flow 설계 후 저장 시 my-console-backend 정상 연동 확인
- [ ] Config Sync를 통해 my-backend 및 api-gw로 설정이 1분 이내 동기화되는지 확인
- [ ] 실행 엔진(my-backend)에서 Flow(동기/비동기/스케줄) 정상 실행 확인
- [ ] 실행 결과가 logging-service로 유실 없이 전달되고, UI에서 트랜잭션 로그가 조회되는지 확인

### 2.2 성능 및 용량 산정 (Performance & Capacity)
- [ ] **목표 TPS 검증**: 목표 부하(예: 1,000 TPS) 상황에서 P95 응답시간이 SLO(API GW < 50ms, Console < 200ms)를 만족하는지 nGrinder/JMeter로 측정 완료
- [ ] **수용량(Capacity) 계획**: CPU/Memory 자원 한계치 도달 지점 확인 및 K8S HPA 임계치 설정 완료
- [ ] **배압(Backpressure) 검증**: RabbitMQ 메시지 적체 시 my-backend 워커가 OOM 없이 안정적으로 처리 속도를 조절하는지 확인

### 2.3 보안 및 규정 준수 (Security & Compliance)
- [ ] 모든 API 엔드포인트에 대한 RBAC 권한 제어 적용 여부 확인 (Security Matrix 기준)
- [ ] 민감 정보(DB 비밀번호, API Key) 암호화 저장 및 로그 마스킹 처리 확인
- [ ] Master Key 외부 주입(Vault, K8S Secret 등) 방식 적용 완료

### 2.4 운영 및 관측성 (Observability & Ops)
- [ ] Prometheus/Grafana 통합 메트릭 대시보드(DB Pool, XA Leak, TPS, 에러율 등) 구성 완료
- [ ] 핵심 SLO 지표 위반 시 Alert(Slack, Email, Webhook) 발송 채널 연동 확인
- [ ] Runbook 및 장애 대응(Incident) 템플릿 작성 및 담당자 숙지 완료

## 3. 승인 (Sign-off)
- **QA/테스트 리드**: [서명/일자]
- **보안 담당자**: [서명/일자]
- **인프라/운영 리드**: [서명/일자]
- **프로젝트 오너(PO)**: [서명/일자]
