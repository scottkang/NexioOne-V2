# Data Definition Structure Metadata Guide

## Purpose
- `Data Definition`은 기본 JSON Schema 외에 `schemaJson.x-nexio` 아래에 구조 메타데이터를 함께 저장한다.
- 이 메타데이터는 실제 필드 validation 규칙을 대체하지 않고, 데이터의 transport/shape 힌트를 제공한다.
- 현재 입력 UI는 `my-console`의 Data Definition 편집 화면에서 관리하는 것을 전제로 한다.

## Storage Location
- 저장 위치: `schemaJson.x-nexio`
- 메타데이터가 기본값과 완전히 같으면 `x-nexio`는 생략된다.
- 직렬화/복원 기준 구현은 `my-console`의 Data Definition 편집 컴포넌트에서 일관되게 처리해야 한다.

## Fields

### Data Format
- 저장 키: `x-nexio.dataFormat`
- 의미: 데이터가 어떤 외형으로 전달되는지 나타낸다.
- 지원 값:
  - `JSON`
  - `XML`
  - `DB_ROW`
  - `DB_MASTER_DETAIL`

### Structure Type
- 저장 키: `x-nexio.structureType`
- 의미: 데이터 shape를 나타낸다.
- 지원 값:
  - `FLAT`
  - `NESTED`
  - `MASTER_DETAIL`

### XML Root Element
- 저장 키: `x-nexio.xml.rootElementName`
- 의미: XML 문서의 root tag 이름
- 사용 시점: `Data Format = XML`
- 예: `OrderMessage`

### DB Object Name
- 저장 키: `x-nexio.db.objectName`
- 의미: 대응하는 DB table/view/object 이름
- 사용 시점: `Data Format = DB_ROW` 또는 `DB_MASTER_DETAIL`
- 예: `TB_ORDER`

### DB Row Type / Alias
- 저장 키: `x-nexio.db.rowTypeName`
- 의미: DB row의 내부 별칭 또는 logical row type 이름
- 사용 시점: `Data Format = DB_ROW` 또는 `DB_MASTER_DETAIL`
- 예: `order_row`

### DB Key Columns
- 저장 키: `x-nexio.db.keyColumns`
- 의미: 해당 row 또는 master record를 식별하는 key column 목록
- UI 입력은 CSV 문자열이지만 저장 시 배열로 직렬화된다.
- 예 입력: `ORDER_ID, TENANT_ID`
- 저장 예: `["ORDER_ID", "TENANT_ID"]`

### Master Record Name
- 저장 키: `x-nexio.masterDetail.masterRecordName`
- 의미: master-detail 구조에서 상위 record 이름
- 사용 시점: `Structure Type = MASTER_DETAIL`
- 예: `order`

### Detail Collection Name
- 저장 키: `x-nexio.masterDetail.detailCollectionName`
- 의미: detail record 목록 이름
- 사용 시점: `Structure Type = MASTER_DETAIL`
- 예: `items`

### Detail Foreign Key
- 저장 키: `x-nexio.masterDetail.detailForeignKey`
- 의미: detail이 master를 참조하는 foreign key 이름
- 사용 시점: `Structure Type = MASTER_DETAIL`
- 예: `ORDER_ID`

## Example JSON

### XML Example
```json
{
  "type": "object",
  "properties": {
    "orderId": {
      "type": "string"
    }
  },
  "x-nexio": {
    "dataFormat": "XML",
    "structureType": "NESTED",
    "xml": {
      "rootElementName": "OrderMessage"
    }
  }
}
```

### DB Master-Detail Example
```json
{
  "type": "object",
  "properties": {
    "orderId": {
      "type": "string"
    },
    "items": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "itemId": {
            "type": "string"
          }
        }
      }
    }
  },
  "x-nexio": {
    "dataFormat": "DB_MASTER_DETAIL",
    "structureType": "MASTER_DETAIL",
    "db": {
      "objectName": "TB_ORDER",
      "rowTypeName": "order_row",
      "keyColumns": [
        "ORDER_ID"
      ]
    },
    "masterDetail": {
      "masterRecordName": "order",
      "detailCollectionName": "items",
      "detailForeignKey": "ORDER_ID"
    }
  }
}
```

## Notes
- 이 메타데이터는 현재 schema validation 규칙이 아니라 transport/structure hint 용도다.
- 필드 타입, required, enum, pattern 같은 실제 validation은 JSON Schema 본문이 담당한다.
- `trigger`, `runtime`, `connector`가 이 메타를 해석하는 방식은 향후 구현 범위에 따라 확장될 수 있다.
