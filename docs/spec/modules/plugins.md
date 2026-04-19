# plugins Specification

**Status**: [Draft/Design] - 런타임 컴포넌트 아티팩트 분리 및 로더 골격 설계 완료.

## 개요
NexioOne 워크플로우 내에서 사용되는 각종 어댑터, 함수, 변환기 등 개별 단위 기능(FlowNode)들을 제공하는 런타임 컴포넌트 계열의 논리 모듈이다.

현재 문서에서 `plugins`는 단일 배포 산출물 이름이라기보다 다음 아티팩트 계층을 묶어 부르는 상위 이름으로 사용한다.
- `my-backend-component-api`: `FlowNodeExecutor` 등 공유 API/인터페이스 계층
- `my-backend-component`: 노드별 표준 구현체 계층
- `my-backend`: 위 컴포넌트를 로드하고 실행에 위임하는 호스트 런타임

## 기술 스택
- **Language**: Java
- **Artifact**: JAR (Spring Boot Library)
- **Mechanism**: Dynamic JAR Loading

## 주요 기능
...
### 1. Standard Component Implementation
- `MAPPING`, `REST_CLIENT`, `SQL_EXECUTOR` 등 핵심 노드 로직 구현.

### 2. Runtime Component Manifest
- 컴포넌트 버전, 체크섬 정보를 관리하는 매니페스트 레지스트리.

### 3. Unified Execution Interface
- 공통 인터페이스(`FlowNodeExecutor`)를 통한 실행 엔진 위임 호출.

## 연관 문서
- `ADR-001: Runtime Component Artifact Separation` (문서 미존재)
- [Runtime Node Support Matrix](../runtime/runtime-node-support-matrix.md)
