# Control Plane API Baseline

**Status**: [Baseline]

## 1. 목적
- 이번 프로그램 범위에 포함된 제어 평면 CRUD/API의 최소 외부 계약을 고정한다.
- 상세 오류 구조는 `error-response-baseline.md`를, deployment/runtime 실행 API는 `api-spec.md`를 기준으로 사용한다.

## 2. 공통 규칙
- Base Path: `/api`
- Auth: Bearer JWT
- Content-Type: `application/json`
- 공통 목록 조회 규칙
  - `page`: 기본 `0`
  - `size`: 기본 `20`, 최대 `100`
  - `sort`: `field,direction` 형식, 기본 `updatedAt,desc`
- 공통 오류 응답: `error-response-baseline.md`

## 3. 인증 API

### 3.1 로그인
- `POST /api/auth/login`
- 권한: 없음

Request:
```json
{
  "username": "owner@nexioone.local",
  "password": "secret"
}
```

Response `200`:
```json
{
  "accessToken": "jwt-access-token",
  "refreshToken": "jwt-refresh-token",
  "tokenType": "Bearer",
  "expiresIn": 3600,
  "authorities": [
    "PROJECT_READ",
    "PROJECT_WRITE",
    "DEPLOYMENT_READ",
    "DEPLOYMENT_WRITE",
    "RUNTIME_READ",
    "RUNTIME_EXECUTE"
  ]
}
```

### 3.2 토큰 갱신
- `POST /api/auth/refresh`
- 권한: 없음

Request:
```json
{
  "refreshToken": "jwt-refresh-token"
}
```

Response `200`: 로그인 응답과 동일 구조

## 4. 프로젝트 API

### 4.1 프로젝트 목록 조회
- `GET /api/projects`
- 권한: `PROJECT_READ`

Response `200`:
```json
{
  "content": [
    {
      "id": 1,
      "name": "Customer Sync",
      "description": "customer integration project",
      "createdAt": "2026-04-18T09:00:00Z",
      "updatedAt": "2026-04-18T09:10:00Z"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 1,
  "totalPages": 1
}
```

### 4.2 프로젝트 생성
- `POST /api/projects`
- 권한: `PROJECT_WRITE`

Request:
```json
{
  "name": "Customer Sync",
  "description": "customer integration project"
}
```

Response `201`:
```json
{
  "id": 1,
  "name": "Customer Sync",
  "description": "customer integration project",
  "createdAt": "2026-04-18T09:00:00Z",
  "updatedAt": "2026-04-18T09:00:00Z"
}
```

Validation:
- `name` 필수, `1..120` chars
- 프로젝트 내 이름 중복 시 `409`

### 4.3 프로젝트 상세 조회
- `GET /api/projects/{projectId}`
- 권한: `PROJECT_READ`

Response `200`: 프로젝트 생성 응답과 동일 구조

### 4.4 프로젝트 수정
- `PUT /api/projects/{projectId}`
- 권한: `PROJECT_WRITE`

Request:
```json
{
  "name": "Customer Sync Updated",
  "description": "updated project description"
}
```

Response `200`: 수정된 프로젝트 구조 반환

### 4.5 프로젝트 삭제
- `DELETE /api/projects/{projectId}`
- 권한: `PROJECT_WRITE`
- Response `204`

## 5. Flow API

### 5.1 Flow 목록 조회
- `GET /api/projects/{projectId}/flows`
- 권한: `PROJECT_READ`

Response `200`:
```json
{
  "content": [
    {
      "id": 10,
      "projectId": 1,
      "name": "customer-sync",
      "description": "sync customer master",
      "version": 1,
      "status": "DRAFT",
      "createdAt": "2026-04-18T09:00:00Z",
      "updatedAt": "2026-04-18T09:10:00Z"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 1,
  "totalPages": 1
}
```

### 5.2 Flow 생성
- `POST /api/projects/{projectId}/flows`
- 권한: `PROJECT_WRITE`

Request:
```json
{
  "name": "customer-sync",
  "description": "sync customer master",
  "flowDefinition": {
    "nodes": [],
    "edges": []
  }
}
```

Response `201`:
```json
{
  "id": 10,
  "projectId": 1,
  "name": "customer-sync",
  "description": "sync customer master",
  "version": 1,
  "status": "DRAFT",
  "createdAt": "2026-04-18T09:00:00Z",
  "updatedAt": "2026-04-18T09:00:00Z"
}
```

### 5.3 Flow 상세 조회
- `GET /api/projects/{projectId}/flows/{flowId}`
- 권한: `PROJECT_READ`

Response `200`:
```json
{
  "id": 10,
  "projectId": 1,
  "name": "customer-sync",
  "description": "sync customer master",
  "version": 1,
  "status": "DRAFT",
  "flowDefinition": {
    "nodes": [
      {
        "id": "start",
        "type": "START",
        "config": {}
      },
      {
        "id": "end",
        "type": "END",
        "config": {}
      }
    ],
    "edges": [
      {
        "sourceNodeId": "start",
        "targetNodeId": "end"
      }
    ]
  },
  "createdAt": "2026-04-18T09:00:00Z",
  "updatedAt": "2026-04-18T09:00:00Z"
}
```

### 5.4 Flow 수정
- `PUT /api/projects/{projectId}/flows/{flowId}`
- 권한: `PROJECT_WRITE`

Request:
```json
{
  "name": "customer-sync-v2",
  "description": "sync customer master updated",
  "flowDefinition": {
    "nodes": [
      {
        "id": "start",
        "type": "START",
        "config": {}
      },
      {
        "id": "map1",
        "type": "MAPPING",
        "config": {
          "mappings": [
            {
              "target": "customerId",
              "expression": "$.customerId"
            }
          ]
        }
      },
      {
        "id": "end",
        "type": "END",
        "config": {}
      }
    ],
    "edges": [
      {
        "sourceNodeId": "start",
        "targetNodeId": "map1"
      },
      {
        "sourceNodeId": "map1",
        "targetNodeId": "end"
      }
    ]
  }
}
```

Response `200`:
```json
{
  "id": 10,
  "projectId": 1,
  "name": "customer-sync-v2",
  "description": "sync customer master updated",
  "version": 2,
  "status": "DRAFT",
  "updatedAt": "2026-04-18T09:20:00Z"
}
```

Validation:
- `flowDefinition.nodes` 필수
- 이번 프로그램 미지원 노드 포함 시 `422` + `BE-2102`
- 순환 구조는 `422`

### 5.5 Flow 삭제
- `DELETE /api/projects/{projectId}/flows/{flowId}`
- 권한: `PROJECT_WRITE`
- Response `204`

## 6. DataDefinition API

### 6.1 DataDefinition 목록 조회
- `GET /api/projects/{projectId}/data-definitions`
- 권한: `PROJECT_READ`

Filter Query:
- `q`: 이름 검색
- `dataFormat`: 선택 필터
- `structureType`: 선택 필터

Response `200`:
```json
{
  "content": [
    {
      "id": 7,
      "projectId": 1,
      "name": "CustomerRequest",
      "description": "input payload schema",
      "createdAt": "2026-04-18T09:00:00Z",
      "updatedAt": "2026-04-18T09:10:00Z"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 1,
  "totalPages": 1
}
```

### 6.2 DataDefinition 생성
- `POST /api/projects/{projectId}/data-definitions`
- 권한: `PROJECT_WRITE`

Request:
```json
{
  "name": "CustomerRequest",
  "description": "input payload schema",
  "schemaJson": {
    "type": "object",
    "properties": {
      "customerId": {
        "type": "string"
      }
    },
    "required": ["customerId"]
  }
}
```

Response `201`:
```json
{
  "id": 7,
  "projectId": 1,
  "name": "CustomerRequest",
  "description": "input payload schema",
  "createdAt": "2026-04-18T09:00:00Z",
  "updatedAt": "2026-04-18T09:00:00Z"
}
```

### 6.3 DataDefinition 상세 조회
- `GET /api/projects/{projectId}/data-definitions/{id}`
- 권한: `PROJECT_READ`

Response `200`:
```json
{
  "id": 7,
  "projectId": 1,
  "name": "CustomerRequest",
  "description": "input payload schema",
  "schemaJson": {
    "type": "object",
    "properties": {
      "customerId": {
        "type": "string"
      }
    },
    "required": ["customerId"]
  },
  "createdAt": "2026-04-18T09:00:00Z",
  "updatedAt": "2026-04-18T09:00:00Z"
}
```

### 6.4 DataDefinition 수정
- `PUT /api/projects/{projectId}/data-definitions/{id}`
- 권한: `PROJECT_WRITE`

Request:
```json
{
  "name": "CustomerRequestV2",
  "description": "updated input payload schema",
  "schemaJson": {
    "type": "object",
    "properties": {
      "customerId": {
        "type": "string"
      },
      "channel": {
        "type": "string"
      }
    },
    "required": ["customerId"]
  }
}
```

Response `200`:
```json
{
  "id": 7,
  "projectId": 1,
  "name": "CustomerRequestV2",
  "description": "updated input payload schema",
  "updatedAt": "2026-04-18T09:20:00Z"
}
```

Validation:
- `schemaJson.type` 필수
- `schemaJson.x-nexio`는 metadata이며 JSON Schema 본문 validation을 대체하지 않는다

### 6.5 DataDefinition 삭제
- `DELETE /api/projects/{projectId}/data-definitions/{id}`
- 권한: `PROJECT_WRITE`
- Response `204`

## 7. Connection API

### 7.1 Connection 목록 조회
- `GET /api/projects/{projectId}/connections`
- 권한: `PROJECT_READ`

Filter Query:
- `type`: `JDBC`, `REST`, `MQ`, `SFTP`
- `q`: 이름 검색

Response `200`:
```json
{
  "content": [
    {
      "id": 12,
      "projectId": 1,
      "name": "source-db",
      "type": "JDBC",
      "status": "ACTIVE",
      "createdAt": "2026-04-18T09:00:00Z",
      "updatedAt": "2026-04-18T09:10:00Z"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 1,
  "totalPages": 1
}
```

### 7.2 Connection 생성
- `POST /api/projects/{projectId}/connections`
- 권한: `PROJECT_WRITE`

Request:
```json
{
  "name": "source-db",
  "type": "JDBC",
  "properties": {
    "url": "jdbc:postgresql://db:5432/app",
    "username": "app"
  },
  "secrets": {
    "password": "masked-or-write-only"
  }
}
```

Response `201`:
```json
{
  "id": 12,
  "projectId": 1,
  "name": "source-db",
  "type": "JDBC",
  "status": "ACTIVE",
  "createdAt": "2026-04-18T09:00:00Z",
  "updatedAt": "2026-04-18T09:00:00Z"
}
```

### 7.3 Connection 상세 조회
- `GET /api/projects/{projectId}/connections/{id}`
- 권한: `PROJECT_READ`

Response Rule:
- secret 원문은 반환하지 않는다.

Response `200`:
```json
{
  "id": 12,
  "projectId": 1,
  "name": "source-db",
  "type": "JDBC",
  "properties": {
    "url": "jdbc:postgresql://db:5432/app",
    "username": "app"
  },
  "status": "ACTIVE",
  "createdAt": "2026-04-18T09:00:00Z",
  "updatedAt": "2026-04-18T09:00:00Z"
}
```

### 7.4 Connection 수정
- `PUT /api/projects/{projectId}/connections/{id}`
- 권한: `PROJECT_WRITE`

Request:
```json
{
  "name": "source-db",
  "type": "JDBC",
  "properties": {
    "url": "jdbc:postgresql://db:5432/app",
    "username": "app_rw"
  },
  "secrets": {
    "password": "rotated-write-only"
  }
}
```

Response `200`:
```json
{
  "id": 12,
  "projectId": 1,
  "name": "source-db",
  "type": "JDBC",
  "status": "ACTIVE",
  "updatedAt": "2026-04-18T09:20:00Z"
}
```

Validation:
- 유형별 필수/옵션 필드는 `connection-profile-contract.md`를 따른다.
- secret은 request에서 write-only로 받고 response에는 포함하지 않는다.

### 7.5 Connection 삭제
- `DELETE /api/projects/{projectId}/connections/{id}`
- 권한: `PROJECT_WRITE`
- Response `204`

## 8. Flow Data Binding API
- 이번 프로그램에서 Flow별 Data Binding은 포함한다.
- 상세 모델은 `flow-owned-data-definition-roles.md`를 따른다.

### 8.1 Binding 목록 조회
- `GET /api/projects/{projectId}/flows/{flowId}/data-bindings`
- 권한: `PROJECT_READ`

Response `200`:
```json
[
  {
    "id": 101,
    "role": "INPUT",
    "bindingKey": "requestBody",
    "displayOrder": 1,
    "required": true,
    "dataDefinitionId": 7,
    "dataDefinitionName": "CustomerRequest"
  }
]
```

### 8.2 Binding 일괄 저장
- `PUT /api/projects/{projectId}/flows/{flowId}/data-bindings`
- 권한: `PROJECT_WRITE`

Request:
```json
{
  "bindingVersion": 3,
  "bindings": [
    {
      "role": "INPUT",
      "bindingKey": "requestBody",
      "required": true,
      "dataDefinitionId": 7
    },
    {
      "role": "OUTPUT",
      "bindingKey": "responseBody",
      "required": true,
      "dataDefinitionId": 8
    }
  ]
}
```

Response `200`:
- 저장 후 전체 binding 목록과 새 `bindingVersion`을 반환한다.

Validation:
- `role`: `INPUT`, `OUTPUT`, `INTERNAL`
- 같은 Flow 내 `role + bindingKey` 조합은 유일
- `INPUT`, `OUTPUT`는 각 0..1개, `INTERNAL`은 복수 허용
- `dataDefinitionId`는 같은 프로젝트의 정의만 참조 가능
- 요청의 `bindingVersion`이 최신 버전과 다르면 `409` + `SYS-1409`
- `DataDefinition.type`는 deprecated/reference 용도로만 남으며 저장 판정에는 사용하지 않는다

## 9. 권한 매핑 요약
| API Group | Read Permission | Write Permission |
|---|---|---|
| Auth | 없음 | 없음 |
| Project | `PROJECT_READ` | `PROJECT_WRITE` |
| Flow | `PROJECT_READ` | `PROJECT_WRITE` |
| DataDefinition | `PROJECT_READ` | `PROJECT_WRITE` |
| Connection | `PROJECT_READ` | `PROJECT_WRITE` |
| Flow Data Binding | `PROJECT_READ` | `PROJECT_WRITE` |
| Deployment | `DEPLOYMENT_READ` | `DEPLOYMENT_WRITE` |

## 10. 참조
- `docs/spec/foundation/release-scope.md`
- `docs/spec/api/api-spec.md`
- `docs/spec/api/error-response-baseline.md`
- `docs/spec/data/connection-profile-contract.md`
- `docs/spec/data/flow-owned-data-definition-roles.md`
- `docs/spec/security/security-authority-matrix.md`
