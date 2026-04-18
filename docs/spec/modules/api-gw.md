# api-gw Specification

**Status**: [Not Implemented] - 제품 범위에 포함되며, 본 문서는 목표 아키텍처만 정의한다.

## 개요
Envoy Proxy 기반의 API 게이트웨이로, 시스템의 입구(Ingress) 역할을 수행하며 실시간 트래픽 제어, 인증/인가, 보안 및 모니터링 필터를 제공한다.

## 기술 스택
- **Engine**: Envoy Proxy
- **Control Plane**: gRPC (xDS 프로토콜)
- **Extension**: Wasm (WebAssembly), Lua Filters

## 주요 기능
...
### 1. Control Plane (xDS)
- gRPC 기반으로 `my-console-backend`로부터 Route, Cluster, Listener 등 설정의 실시간 동기화.

### 2. External Authz
- API Key, JWT, OAuth2 기반의 중앙 집중식 인증 및 인가 처리.

### 3. Global Rate Limit
- Redis 연동을 통해 시스템 전역 사용량 제한 및 Quota 관리.

## 연관 문서
- [API Spec](../api/api-spec.md)
- [Runtime Operations Spec](../runtime/runtime-operations-spec.md)
