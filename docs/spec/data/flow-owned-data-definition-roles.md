# Flow-Owned Data Definition Roles

**Status**: [Baseline]
**Decision Date**: `2026-04-18`

## Background
- 기존 구현/프로토타입에서는 `DataDefinition` 엔티티에 `type(INPUT/OUTPUT)` 필드를 두는 방향이 검토되었다.
- 기존 UI/프로토타입에서도 `ALL / INPUT / OUTPUT` 필터가 함께 검토되었다.
- 하지만 실제 데이터 구조의 역할은 `Data Definition` 자체보다 `Flow` 문맥에서 결정되는 경우가 많다.
- 같은 구조라도 어떤 Flow에서는 입력, 다른 Flow에서는 출력, 또 다른 Flow에서는 내부 working data가 될 수 있다.

## Problem
- `DataDefinition.type`은 구조 정의에 역할을 고정해 재사용성을 낮춘다.
- `INPUT / OUTPUT` 2분류만으로는 Flow 내부 working data를 표현할 수 없다.
- 구조 라이브러리와 Flow 계약이 섞여 있어서 장기적으로 API/runtime 연결 시 모델 충돌이 생긴다.

## Goal
- `Data Definition`은 역할 중립적인 구조 라이브러리로 유지한다.
- `Flow`가 `input / output / internal` 역할 바인딩을 가진다.
- 동일한 `Data Definition`을 여러 Flow에서 서로 다른 역할로 재사용할 수 있게 한다.

## Target Model

### Data Definition
- 역할 없는 순수 구조 정의
- 책임:
  - JSON Schema 본문
  - `schemaJson.x-nexio` transport/structure metadata
  - 프로젝트 단위 재사용 가능한 schema catalog

예:
- `Order`
- `OrderItem`
- `CustomerProfile`

### Flow Role Bindings
- `Flow`가 어떤 `Data Definition`을 어떤 역할로 쓰는지 가진다.

권장 역할:
- `INPUT`
  - Flow 시작 시점 payload 계약
- `OUTPUT`
  - Flow 종료 시점 결과 계약
- `INTERNAL`
  - Flow 내부 intermediate/working data 계약

## Persistence Decision

- 이번 프로그램의 기준 persistence 모델은 `Option A: Dedicated Binding Table`로 고정한다.
- `Flow.flowData` 또는 임의 JSON 컬럼에 binding metadata를 넣는 `Option B`는 이번 프로그램 기준에서 채택하지 않는다.
- deployment snapshot에는 Flow별 binding 목록이 포함되어야 한다.

## Recommended Persistence Shape

### Option A: Dedicated Binding Table
- 채택안

테이블 예시:
- `TB_FLOW_DATA_BINDING`
  - `id`
  - `flow_id`
  - `data_definition_id`
  - `role` (`INPUT | OUTPUT | INTERNAL`)
  - `binding_key`
  - `display_order`
  - `required`
  - `notes`

설명:
- `binding_key`
  - 내부 working data를 구분하는 logical key
  - 예: `requestBody`, `responseBody`, `orderHeader`, `orderItems`
- `display_order`
  - UI 표시 순서 제어
- `required`
  - Flow 계약상 필수 여부

장점:
- 정규화가 명확하다.
- 한 Flow에 여러 internal binding을 둘 수 있다.
- 향후 validation/runtime 연결에 유리하다.

### Option B: Flow JSON/Settings Embedded Metadata
- 대안으로 검토했으나 이번 프로그램에서는 미채택
- `Flow.flowData` 또는 별도 JSON 컬럼 안에 binding metadata를 넣는 방식

단점:
- 검색/조회/검증이 어렵다.
- console-backend API와 runtime sync가 복잡해진다.

## Recommended API Shape

### Data Definition
- `GET /api/projects/{projectId}/data-definitions`
- `POST /api/projects/{projectId}/data-definitions`
- `PUT /api/projects/{projectId}/data-definitions/{id}`
- `DELETE /api/projects/{projectId}/data-definitions/{id}`

변경점:
- 이번 프로그램에서는 `type(INPUT/OUTPUT)`를 optional/deprecated로 유지한다.
- 신규 API/화면/배포 계약은 `Flow Data Binding`을 우선 기준으로 사용한다.

### Flow Binding
- `GET /api/projects/{projectId}/flows/{flowId}/data-bindings`
- `PUT /api/projects/{projectId}/flows/{flowId}/data-bindings`
- 이번 프로그램의 baseline write API는 `PUT` 일괄 저장으로 고정한다.
- 개별 `POST` / `DELETE`는 차기 편의 API로 둔다.

응답 예시:
```json
[
  {
    "id": 101,
    "role": "INPUT",
    "bindingKey": "requestBody",
    "required": true,
    "dataDefinitionId": 7,
    "dataDefinitionName": "OrderRequest"
  },
  {
    "id": 102,
    "role": "OUTPUT",
    "bindingKey": "responseBody",
    "required": true,
    "dataDefinitionId": 8,
    "dataDefinitionName": "OrderResponse"
  },
  {
    "id": 103,
    "role": "INTERNAL",
    "bindingKey": "orderItems",
    "required": false,
    "dataDefinitionId": 9,
    "dataDefinitionName": "OrderItemList"
  }
]
```

## UI Direction

### Data Definitions Tab
- `INPUT / OUTPUT` 필터 제거
- 전체 구조 목록만 관리
- 이름, 설명, format/structure metadata 중심

### Flow Designer / Flow Settings
- 새 섹션: `Data Bindings`
- 그룹:
  - `Input`
  - `Output`
  - `Internal`
- 각 binding에서:
  - Data Definition 선택
  - binding key 입력
  - required 여부
  - 설명/notes

권장 UX:
- `Flow Details` 또는 `Flow Settings` 상단에서 input/output binding
- canvas 아래 inspector 또는 별도 panel에서 internal binding 관리

## Runtime/Deployment Impact
- deployment snapshot에는 프로젝트 전체 `dataDefinitions`뿐 아니라 Flow별 binding 정보도 포함돼야 한다.
- runtime은 이 binding을 사용해:
  - 시작 payload validation
  - 종료 payload validation
  - 내부 node contract 검증
  - API/webhook contract 노출
로 확장할 수 있다.

Stage 1 snapshot 최소 shape:
```json
{
  "flowId": 10,
  "bindings": [
    {
      "role": "INPUT",
      "bindingKey": "requestBody",
      "required": true,
      "dataDefinitionId": 7,
      "dataDefinitionVersion": 3
    }
  ]
}
```

Internal execution payload에서는 위 binding snapshot을 `flowBindings` 필드로 전달하며, 기준 문서는 `api/internal-execution-schema.md`다.

## Migration Strategy

### Step 1: Add Flow Bindings
- 새 binding 테이블/API/UI 추가
- 기존 `DataDefinition.type`는 유지
- 신규 Flow부터 binding 사용 가능

### Step 2: Backfill Existing Records
- 기존 `INPUT` 타입 정의는 Flow별 기본 `INPUT` 후보로 매핑
- 기존 `OUTPUT` 타입 정의는 Flow별 기본 `OUTPUT` 후보로 매핑
- 자동 매핑이 불가능한 경우 수동 확인 필요

주의:
- 현재 `DataDefinition.type`만으로는 어떤 Flow에 연결돼야 하는지 알 수 없으므로, 완전 자동 마이그레이션은 불가능하다.
- 실제 backfill은:
  - 비어 있는 상태로 시작하거나
  - Flow별 사용자가 binding을 수동 지정하는 방식이 더 안전하다.

### Step 3: Deprecate DataDefinition.type
- UI에서 더 이상 `INPUT / OUTPUT` 필터를 노출하지 않음
- backend request/response에서 `type`을 optional/deprecated 처리

### Step 4: Remove DataDefinition.type
- DB column 제거
- service/repository/filter 정리

## Trade-Offs

### Pros
- 구조 정의 재사용성이 높아진다.
- Flow 문맥에서 역할을 정확히 표현할 수 있다.
- internal working data까지 모델링 가능하다.
- runtime/API 계약과 연결하기 쉬워진다.

### Cons
- 초기 구현 범위가 커진다.
- 기존 데이터는 명시적 binding migration이 필요하다.
- 단순 CRUD보다 Flow 설정 UX가 조금 복잡해진다.

## Recommendation
- `DataDefinition.type`는 장기적으로 제거하는 것이 맞다.
- 먼저 `Flow` binding 모델을 추가하고, 기존 `type`은 transition 기간 동안만 유지한다.
- migration은 자동 변환보다 사용자 확인 기반 수동 연결이 더 안전하다.
