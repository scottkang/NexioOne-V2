# my-console Specification

**Status**: [Not Implemented] - 제품 범위에 포함되며, 본 문서는 목표 기능 기준만 정의한다.

## 개요
NexioOne의 프론트엔드 웹 애플리케이션으로, 사용자가 프로젝트를 설계하고 API를 관리하며 시스템 상태를 모니터링할 수 있는 직관적인 UI를 제공한다.

이번 프로그램 포함 범위는 `docs/spec/foundation/release-scope.md`를 우선 기준으로 해석한다. 본 문서의 상세 화면 아이디어는 차기 범위를 함께 포함할 수 있다.

## 기술 스택
- **Framework**: React (TypeScript)
- **Styling**: Tailwind CSS
- **Build Tool**: Vite
- **Routing**: React Router

## 주요 기능
...
### 1. Dashboard
- 차기 범위 중심.

### 2. API Management
- **API Designer**: 차기 범위
- **API Product**: 차기 범위

### 3. Project Management
- **Flow Designer**: 드래그 앤 드롭 방식의 워크플로우 설계 인터페이스.
- **Data Definitions**: 프로젝트 공통 데이터 구조 및 스키마 정의.

### 4. Monitoring & Ops
- **Transaction Viewer**: 차기 범위 또는 logging read model 연동 이후 확장
- **Deployment**: 환경별 배포 제어 및 롤백 기능.

## 이번 프로그램 포함
- 로그인 이후 기본 navigation
- 프로젝트/Flow/DataDefinition/Connection/Deployment 관리 화면
- dry-run / execute-stub 호출 화면
- runtime 상태 조회 화면

## 연관 문서
- [Product Spec](../foundation/product-spec.md)
- [API Spec](../api/api-spec.md)
