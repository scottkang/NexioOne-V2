# Deployment & Compatibility Guide

**Status**: [Draft/Design]
**Related Modules**: my-backend, my-console-backend, plugins

## 1. 개요
본 문서는 무중단 배포(Zero-downtime Deployment), 플러그인(Component)의 동적 업데이트 절차, 그리고 콘솔과 백엔드 엔진 간의 하위 호환성 유지 정책을 정의한다.

## 2. 배포 전략 (Deployment Strategies)

### 2.1 애플리케이션 무중단 배포 (Rolling / Blue-Green)
- `my-console-backend` 및 `my-backend`는 기본적으로 K8S의 **Rolling Update** 전략을 사용한다.
- 배포 시 기존 파드는 진행 중인 트랜잭션이 완료될 때까지 대기하며(Graceful Shutdown, 기본 30초 설정), 신규 파드가 Readiness Probe를 통과한 후 트래픽을 넘겨받는다.

### 2.2 플러그인(컴포넌트) 동적 배포
실행 엔진의 재시작 없이 새로운 커스텀 노드(예: 사내 레거시 연동 전용 플러그인)를 추가하는 절차:
1. 관리자가 신규 플러그인 바이너리(.jar 또는 패키지)와 메타데이터(node-catalog-schema 일치)를 Object Storage에 업로드한다.
2. `my-console-backend` API를 통해 플러그인 버전과 체크섬을 등록한다.
3. Redis Pub/Sub을 통해 모든 `my-backend` 인스턴스에 신규 플러그인 배포 이벤트가 전파된다.
4. 워커들은 백그라운드에서 신규 아티팩트를 다운로드하고, 동적 클래스로더(Custom ClassLoader)를 통해 메모리에 적재한 후 로컬 Catalog를 갱신한다.

## 3. 버전 및 하위 호환성 (Compatibility Matrix)

시스템 진화에 따라 Flow Definition 규격이나 노드 설정 포맷이 변경될 수 있으므로 아래 원칙을 준수한다.

### 3.1 Flow Definition 버전 관리
- `TB_FLOW` 테이블에 저장되는 JSON 데이터는 항상 상위에 `"$version"` 필드를 명시한다. (예: `"$version": "1.2.0"`)
- `my-backend`의 Flow 파서는 이전 버전의 JSON을 읽을 때 최신 객체 모델로 자동 업컨버팅(Upcasting)하는 Migration Adapter를 내장해야 한다.

### 3.2 노드 설정(Config) 호환성
- **필드 추가**: 신규 필드 추가 시 반드시 기본값(Default)을 정의하여 기존 Flow가 에러 없이 실행되도록 한다.
- **필드 삭제/변경**: 기존 필드는 최소 2개의 메이저 릴리즈 동안 `Deprecated` 상태로 유지하며, 파싱 에러를 발생시키지 않는다.
- 콘솔 UI는 구버전 Flow를 열었을 때 사용자에게 경고 아이콘을 표시하고, [Save] 시 최신 규격으로 자동 마이그레이션하여 저장한다.

## 4. Contract Testing
모듈이 분리되어 있으므로, `my-console-backend`가 생성하는 JSON 구조와 `my-backend`가 파싱하는 객체 구조 간의 정합성을 보장하기 위해 Spring Cloud Contract 기반의 자동화 테스트를 CI 파이프라인에 필수적으로 포함시킨다.
