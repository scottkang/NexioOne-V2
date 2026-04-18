# my-console Specification

**Status**: [Not Implemented] - 제품 범위에 포함되며, 본 문서는 목표 기능 기준만 정의한다.

## 개요
NexioOne의 프론트엔드 웹 애플리케이션으로, 사용자가 프로젝트를 설계하고 API를 관리하며 시스템 상태를 모니터링할 수 있는 직관적인 UI를 제공한다.

## 기술 스택
- **Framework**: React (TypeScript)
- **Styling**: Tailwind CSS
- **Build Tool**: Vite
- **Routing**: React Router

## 주요 기능
...
### 1. Dashboard
- 실시간 트래픽 요약 및 시스템 건전성 지표 시각화.

### 2. API Management
- **API Designer**: 엔드포인트 생성, HTTP 메서드 정의, 보안 정책 설정.
- **API Product**: API 그룹화, 구독 관리, Rate Limiting 및 Quota 정책 설정.

### 3. Project Management
- **Flow Designer**: 드래그 앤 드롭 방식의 워크플로우 설계 인터페이스.
- **Data Definitions**: 프로젝트 공통 데이터 구조 및 스키마 정의.

### 4. Monitoring & Ops
- **Transaction Viewer**: 트랜잭션 ID 기반의 상세 실행 로그 추적.
- **Deployment**: 환경별 배포 제어 및 롤백 기능.

## 연관 문서
- [Product Spec](../foundation/product-spec.md)
- [API Spec](../api/api-spec.md)
