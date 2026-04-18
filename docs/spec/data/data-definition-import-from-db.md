# Data Definition Import From DB

## Purpose
- DB 메타데이터를 읽어 `Data Definition` 초안을 자동 생성한다.
- 사용자는 table/view 구조를 직접 다시 입력하지 않고, DB schema에서 생성된 결과를 검토 후 저장한다.
- 1차 목표는 `단일 table/view -> DB_ROW Data Definition` 생성이다.

## Background
- 현재 `Data Definition`은 control plane 저장 모델에서 `schemaJson`을 직접 저장하는 방향을 기준으로 한다.
- 현재 `Data Definition` UI는 수동 작성 또는 schema builder 중심으로 설계돼 있다.
- 프로젝트에는 재사용 가능한 Connection 관리 모델이 있고, JDBC connection을 활용할 수 있다.

## Goal
- JDBC connection을 선택해 schema/table/view 메타데이터를 조회한다.
- 선택한 DB object로부터 JSON Schema + `x-nexio.db` 메타데이터를 생성한다.
- 사용자는 preview를 수정한 뒤 `Data Definition`으로 저장한다.
- 현재 구현 기준으로는 단일 table/view의 `DB_ROW`와, FK 기반 master-detail 1쌍의 `DB_MASTER_DETAIL` preview까지 지원한다.

## Scope

### In Scope
- JDBC connection 기반 메타데이터 조회
- schema/catalog 선택
- table/view 목록 조회
- column/type/nullability/PK 기반 JSON Schema 생성
- `DB_ROW` metadata 자동 입력
- preview 후 저장

### Out of Scope
- trigger/data import
- 복잡한 SQL query 결과셋 import
- 완전 자동 master-detail 역설계
- DB vendor별 고급 domain/enum/UDT 완전 지원
- runtime 자동 validation 연결

## Recommended UX

### Entry Point
- `Data Definitions` 탭에 `Import From DB` 버튼 추가

### Flow
1. JDBC connection 선택
2. schema/catalog 선택
3. table/view 선택
4. preview 생성
5. 이름 / 설명 / metadata 보정
6. `Save As Data Definition`

### Preview Content
- Definition name 제안
- detected columns 표
- generated JSON Schema
- generated `x-nexio` metadata
- inferred key columns

## Recommended Backend API

### 1. List Importable DB Objects
- `GET /api/projects/{projectId}/data-definitions/import/db/connections`
  - JDBC 가능한 connection 목록 반환

- `GET /api/projects/{projectId}/data-definitions/import/db/connections/{connectionId}/schemas`
  - schema/catalog 목록 반환

- `GET /api/projects/{projectId}/data-definitions/import/db/connections/{connectionId}/objects?schema=...`
  - table/view 목록 반환

응답 예시:
```json
[
  {
    "objectName": "TB_ORDER",
    "objectType": "TABLE"
  },
  {
    "objectName": "VW_ORDER_SUMMARY",
    "objectType": "VIEW"
  }
]
```

### 2. Generate Preview
- `POST /api/projects/{projectId}/data-definitions/import/db/preview`

요청 예시:
```json
{
  "connectionId": 12,
  "schemaName": "PUBLIC",
  "objectName": "TB_ORDER"
}
```

응답 예시:
```json
{
  "suggestedName": "TB_ORDER",
  "objectType": "TABLE",
  "schemaJson": "{ ... }",
  "metadata": {
    "dataFormat": "DB_ROW",
    "structureType": "FLAT",
    "dbObjectName": "TB_ORDER",
    "dbRowTypeName": "tb_order_row",
    "dbKeyColumns": ["ORDER_ID"]
  },
  "columns": [
    {
      "name": "ORDER_ID",
      "dbType": "BIGINT",
      "nullable": false,
      "jsonSchemaType": "integer",
      "required": true,
      "primaryKey": true
    }
  ]
}
```

### 3. Save Imported Definition
- 기존 `POST /api/projects/{projectId}/data-definitions` 재사용
- preview 결과를 사용자가 확인 후 저장

## Mapping Rules

### DB -> JSON Schema
- `CHAR`, `VARCHAR`, `TEXT`, `CLOB`
  - `type: string`
- `INTEGER`, `INT`, `BIGINT`, `SMALLINT`
  - `type: integer`
- `DECIMAL`, `NUMERIC`, `FLOAT`, `DOUBLE`, `REAL`
  - `type: number`
- `BOOLEAN`, `BIT`
  - `type: boolean`
- `DATE`
  - `type: string`, `format: date`
- `TIME`
  - `type: string`, `format: time`
- `TIMESTAMP`, `DATETIME`
  - `type: string`, `format: date-time`
- `BLOB`, `BINARY`, `VARBINARY`
  - 1차는 `type: string` + 설명 보강 또는 제외 판단 필요

### Nullability
- `nullable = false`면 `required`에 포함

### Primary Keys
- PK 컬럼은 `x-nexio.db.keyColumns`에 반영

### Naming
- Definition name
  - 기본값은 object name
- `db.rowTypeName`
  - 기본값은 `snake_case(objectName) + "_row"` 또는 lower alias

## Generated Metadata Shape
- `dataFormat = DB_ROW`
- `structureType = FLAT`
- `db.objectName = <selected table/view>`
- `db.rowTypeName = inferred alias`
- `db.keyColumns = inferred PK columns`

예시:
```json
{
  "x-nexio": {
    "dataFormat": "DB_ROW",
    "structureType": "FLAT",
    "db": {
      "objectName": "TB_ORDER",
      "rowTypeName": "tb_order_row",
      "keyColumns": ["ORDER_ID"]
    }
  }
}
```

## Master-Detail Expansion Plan

### Phase 1
- 단일 table/view 지원
- 결과는 `DB_ROW`

### Phase 2
- FK를 따라 parent-child 관계를 선택적으로 확장
- 사용자가:
  - master object
  - detail object
  - FK column
  - detail collection name
  를 선택
- 결과는 `DB_MASTER_DETAIL` + `MASTER_DETAIL`
- 현재 구현 상태: 1쌍의 detail candidate 선택까지 지원

### Phase 3
- 다단계 관계 또는 복수 detail collection 지원

## Recommended Technical Approach

### my-console-backend
- JDBC connection `config`를 해석해 metadata query 수행
- vendor neutral 우선:
  - `DatabaseMetaData#getSchemas`
  - `getTables`
  - `getColumns`
  - `getPrimaryKeys`
- connection type이 `JDBC`가 아닌 경우 import 대상에서 제외

### my-console
- wizard 또는 inline panel UI
- preview 표 + schema preview + save form 조합

### my-backend
- 1차에는 직접 영향 없음
- 향후 runtime contract validation과 연결 가능

## Risks
- DB vendor별 type 이름 차이
- composite PK/quoted identifier 처리
- large schema/table 목록에서 UX 복잡도 증가
- encrypted connection config 해석 정책 정리 필요

## Recommendation
- 1차는 `JDBC connection + single table/view import`로 제한한다.
- preview 생성 후 사용자가 수정하는 `semi-automatic` 방식으로 간다.
- `MASTER_DETAIL` 자동 생성은 후속 단계로 분리한다.
