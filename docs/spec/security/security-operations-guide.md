# Operational Security & Key Management Guide

**Status**: [Draft/Design]
**Related Modules**: my-console-backend, my-backend

## 1. 개요
본 문서는 NexioOne 시스템 운영 중 발생할 수 있는 키 교체(Key Rotation), 민감 정보 갱신 및 보안 사고 발생 시의 대응 절차를 정의한다.

## 2. 키 로테이션 (Key Rotation) 정책

### 2.1 마스터 키 (Master Key) 교체
마스터 키는 모든 프로젝트 키를 보호하는 최상위 키로, 정기적(예: 1년) 또는 유출 의심 시 교체한다.
1. **신규 키 발급**: K8S Secret 또는 Vault에 신규 마스터 키(V2)를 등록한다.
2. **이중 키 지원(Dual Key Support)**: `my-console-backend`는 과도기 동안 구형 키(V1)와 신규 키(V2)를 모두 읽을 수 있도록 설정된다 (`NEXIO_MASTER_KEY_V1`, `NEXIO_MASTER_KEY_V2`).
3. **데이터 재암호화(Re-encryption)**: 백그라운드 배치(Batch) 작업을 통해 기존 V1으로 암호화된 모든 `TB_CONNECTION` 설정값을 복호화한 후, V2로 재암호화하여 저장한다.
4. **구형 키 폐기**: 모든 데이터가 V2로 마이그레이션되었음을 확인한 후, 설정에서 V1 키를 제거하고 서비스를 재시작한다.

### 2.2 외부 연동 비밀번호 갱신 (Secret Management)
외부 시스템(Target DB, Target API)의 비밀번호가 변경된 경우의 무중단 적용 절차:
1. `my-console` UI의 [Shared Connections] 메뉴에서 해당 Connection의 비밀번호를 신규 값으로 업데이트한다.
2. 저장 즉시 Config Sync(Redis Pub/Sub)가 발생하여 모든 `my-backend` 워커 노드에 변경 알림이 전송된다.
3. 워커 노드는 캐싱된 Connection 프로필을 무효화(Invalidate)하고, 다음 Flow 실행 시점에 DB에서 신규 암호화된 설정값을 조회하여 새로운 커넥션 풀을 맺는다. (기존 커넥션 풀은 Graceful shutdown 처리)

## 3. 보안 침해 대응 (Incident Response)
1. **토큰 유출**: 특정 사용자의 JWT 토큰 유출 의심 시, Admin UI에서 해당 사용자의 세션을 강제 만료(Revoke)시키고 Redis의 Refresh Token을 삭제한다.
2. **비정상 API 호출 감지**: api-gw의 Rate Limiter와 WAF 필터를 통해 특정 IP/Tenant의 비정상 트래픽이 감지되면 자동으로 접근을 차단(Block)하고 보안 관리자에게 Alert를 발송한다.
