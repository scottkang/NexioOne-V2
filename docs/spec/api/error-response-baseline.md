# Error Response Baseline

**Status**: [Baseline]

## 1. 목적
- 외부 API와 내부 API 문서에서 공통으로 사용할 오류 응답 구조와 코드 체계를 단일화한다.

## 2. ApiErrorResponse
```json
{
  "timestamp": "2026-04-18T09:00:00Z",
  "status": 422,
  "error": "Unprocessable Entity",
  "code": "SYS-1002",
  "message": "validation failed",
  "path": "/api/projects/1/runtime/flows/10/dry-run",
  "details": [
    {
      "field": "inputContext",
      "reason": "must be object"
    }
  ],
  "traceId": "trace-20260418-0001"
}
```

## 3. Field Rules
| Field | Required | Rules |
|---|---|---|
| `timestamp` | Y | ISO-8601 UTC |
| `status` | Y | HTTP status code |
| `error` | Y | HTTP reason phrase |
| `code` | Y | 전역 오류 코드 |
| `message` | Y | 사용자/운영자용 단문 |
| `path` | Y | request path |
| `details` | N | validation/field-level 오류 상세 |
| `traceId` | N | 추적 ID |

## 4. Code System
- `SYS`: 공통 시스템/인프라/인증/권한/검증
- `ADM`: `my-console-backend`
- `BE`: `my-backend`
- `GW`: `api-gw`
- `LOG`: `logging-service`

## 5. Baseline Codes
### SYS
- `SYS-0001`: unknown internal error
- `SYS-1001`: resource not found
- `SYS-1002`: validation failed
- `SYS-1100`: authentication required
- `SYS-1101`: permission denied
- `SYS-1409`: conflict
- `SYS-1503`: dependent service unavailable

### ADM
- `ADM-2001`: deployment version conflict
- `ADM-2002`: unsupported deployment status transition

### BE
- `BE-2001`: invalid flow definition
- `BE-2101`: connection profile not found
- `BE-2102`: unsupported node type
- `BE-2103`: idempotency key conflict
- `BE-2301`: stub execution failed

### LOG
- `LOG-3001`: event schema validation failed
- `LOG-3002`: execution event persistence failed

## 6. HTTP Mapping
- `400`: malformed JSON or unsupported parameter format
- `401`: `SYS-1100`
- `403`: `SYS-1101`
- `404`: `SYS-1001`
- `409`: `SYS-1409`, `ADM-2001`, `ADM-2002`, `BE-2103`
- `422`: `SYS-1002`, `BE-2001`, `BE-2102`
- `500`: `SYS-0001`, `LOG-3002`
- `503`: `SYS-1503`

## 7. Example
```json
{
  "timestamp": "2026-04-18T09:40:00Z",
  "status": 409,
  "error": "Conflict",
  "code": "BE-2103",
  "message": "idempotency key already used with different payload",
  "path": "/execute/async",
  "traceId": "trace-20260418-0002"
}
```

## 8. 참조
- `docs/spec/api/global-error-code-map.md`
