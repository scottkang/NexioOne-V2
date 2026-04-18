# Global Error Code & Message Specification

## 1. 개요
본 문서는 NexioOne 시스템 전체에서 공통으로 사용할 에러 코드 구조와 메시지 규격을 정의한다. 이를 통해 각 모듈 간의 협업 및 문제 진단 효율을 높인다.

## 2. 공통 에러 응답 구조 (JSON)
```json
{
  "timestamp": "2026-04-18T10:15:00Z",
  "status": 503,
  "error": "Service Unavailable",
  "code": "SYS-1503",
  "message": "dependent service unavailable",
  "path": "/api/v1/execute",
  "traceId": "t-12345-67890"
}
```

단일 구조 기준은 `error-response-baseline.md`를 따른다.

## 3. 에러 코드 체계 (Naming Convention)

코드는 `[모듈 식별자]-[4자리 숫자]` 조합으로 구성한다.

| 모듈 식별자 | 설명 |
| :--- | :--- |
| **SYS** | 공통 시스템/인프라 오류 |
| **GW** | API Gateway (api-gw) 오류 |
| **BE** | Workflow Engine (my-backend) 오류 |
| **ADM** | Console Backend (my-console-backend) 오류 |
| **LOG** | Logging Service 오류 |

## 4. 코드 범위별 정의

### 4.1 SYS (System)
- **SYS-0001**: Unknown Internal Error
- **SYS-1001**: Resource Not Found
- **SYS-1002**: Invalid Parameter / Validation Failed
- **SYS-1100**: Authentication Required
- **SYS-1101**: Permission Denied (RBAC)
- **SYS-1409**: Conflict
- **SYS-1503**: Dependent Service Unavailable

### 4.2 GW (Gateway)
- **GW-3001**: Rate Limit Exceeded
- **GW-3002**: Quota Exceeded
- **GW-3101**: Upstream Service Unavailable
- **GW-3102**: Upstream Timeout

### 4.3 BE (Backend Engine)
- **BE-2001**: Flow Definition Parsing Error
- **BE-2101**: Connection Profile Not Found
- **BE-2102**: Unsupported Node Type
- **BE-2103**: Idempotency Key Conflict
- **BE-2301**: Stub Execution Failed

### 4.4 ADM (Control Plane)
- **ADM-2001**: Deployment Version Conflict
- **ADM-2002**: Unsupported Deployment Status Transition

### 4.5 LOG (Logging Service)
- **LOG-3001**: Event Schema Validation Failed
- **LOG-3002**: Execution Event Persistence Failed

## 5. HTTP 상태 코드 매핑 규칙
- **400 Bad Request**: malformed JSON, unsupported parameter format
- **401 Unauthorized**: SYS-1100
- **403 Forbidden**: SYS-1101
- **404 Not Found**: SYS-1001
- **409 Conflict**: SYS-1409, ADM-2001, ADM-2002, BE-2103
- **422 Unprocessable Entity**: SYS-1002, BE-2001, BE-2102
- **429 Too Many Requests**: GW-3001
- **500 Internal Server Error**: SYS-0001, LOG-3002
- **503 Service Unavailable**: SYS-1503
- **504 Gateway Timeout**: GW-3102
