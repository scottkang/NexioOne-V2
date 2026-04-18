# Component Catalog & Property Schema Specification

## 1. 개요
본 문서는 Flow Designer에서 사용하는 노드 컴포넌트의 메타데이터 규격을 정의한다. 백엔드는 이 규격에 따라 현재 런타임에 등록된 모든 `FlowNodeExecutor`의 정보를 반환하며, 프론트엔드는 이를 기반으로 UI를 동적으로 구성한다.

## 2. Catalog Response Schema (JSON)

### 2.1 전체 구조
```json
{
  "version": "1.0.0",
  "generatedAt": "2026-04-18T10:00:00Z",
  "components": [
    {
      "type": "SQL_EXECUTOR",
      "version": "1.2.0",
      "category": "DATABASE",
      "displayName": "SQL 실행기",
      "description": "지정된 DB에서 SQL 쿼리를 실행합니다.",
      "icon": "database-icon",
      "tags": ["JDBC", "Read", "Write"],
      "properties": [
        {
          "name": "connectionRef",
          "label": "연결 설정",
          "uiComponent": "CONNECTION_SELECTOR",
          "required": true,
          "optionsUrl": "/api/v1/connections?type=JDBC"
        },
        {
          "name": "query",
          "label": "SQL 쿼리",
          "uiComponent": "CODE_EDITOR",
          "required": true,
          "language": "sql"
        }
      ],
      "outputSchema": {
        "type": "object",
        "properties": {
          "rows": { "type": "array" },
          "count": { "type": "integer" }
        }
      }
    }
  ]
}
```

### 2.2 UI Component 종류
- **TEXT**: 단일행 텍스트 입력
- **TEXTAREA**: 다행 텍스트 입력
- **SELECT**: 드롭다운 선택
- **SWITCH**: Boolean 토글
- **CODE_EDITOR**: SQL, JS, JSON 등 코드 에디터 (Monaco 기반)
- **CONNECTION_SELECTOR**: Shared Connection 참조 선택기

## 3. 동적 속성 제어 (Dependencies)
특정 속성의 값에 따라 다른 속성을 노출하거나 숨겨야 하는 경우 `dependencies` 필드를 사용한다.
```json
{
  "name": "advancedMapping",
  "label": "고급 매핑 사용",
  "uiComponent": "SWITCH"
},
{
  "name": "mappingScript",
  "label": "매핑 스크립트",
  "uiComponent": "CODE_EDITOR",
  "dependencies": {
    "showIf": { "advancedMapping": true }
  }
}
```
