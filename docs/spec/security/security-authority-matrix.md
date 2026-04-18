# NexioOne Security Authority Matrix (RBAC)

## 1. 개요
본 문서는 NexioOne 시스템의 역할(Role)과 권한(Permission) 간의 매핑을 정의한다. 모든 API 요청과 UI 기능 노출은 이 매트릭스를 기준으로 제어된다.

## 2. 역할 (Roles) 정의
- **`ADMIN`**: 시스템 전체 관리자. 모든 기능에 대한 읽기/쓰기 권한을 보유한다.
- **`OPERATOR`**: 운영 담당자. 프로젝트 관리 및 런타임 제어(배포, 실행) 권한을 보유하나, 보안 설정 및 사용자 관리는 제한된다.
- **`VIEWER`**: 모니터링 담당자. 시스템 상태 및 설정에 대한 읽기 권한만 보유하며, 변경 및 실행은 불가능하다.

## 3. 권한(Permission) 및 기능 매핑

| 기능 영역 | 권한 코드 | ADMIN | OPERATOR | VIEWER | 설명 |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **Authentication** | `AUTH_LOGIN` | O | O | O | 로그인 및 토큰 갱신 |
| **Settings** | `SETTINGS_READ` | O | O | O | 전역 설정 및 공급자 조회 |
| | `SETTINGS_WRITE` | O | X | X | 전역 설정 및 공급자 수정 |
| **RBAC** | `USER_MGMT` | O | X | X | 사용자 및 역할 관리 |
| **Project** | `PROJECT_READ` | O | O | O | 프로젝트, Flow, DataDefinition, Connection, Flow Data Binding 조회 |
| | `PROJECT_WRITE` | O | O | X | 프로젝트, Flow, DataDefinition, Connection, Flow Data Binding 생성/수정/삭제 |
| **Deployment** | `DEPLOYMENT_READ` | O | O | O | 배포 이력 및 상태 조회 |
| | `DEPLOYMENT_WRITE` | O | O | X | 배포 생성, 상태 변경, 롤백 요청 |
| | `DEPLOYMENT_EXECUTE`| O | O | X | 향후 real deployment 기반 실행 제어용 reserved 권한 |
| **Runtime** | `RUNTIME_READ` | O | O | O | 런타임 지표 및 알람 조회 |
| | `RUNTIME_EXECUTE` | O | O | X | 수동 트리거, dry-run, execute-stub 호출 |
| **API Docs** | `API_DOCS_READ` | O | O | O | Swagger/OAS 명세 조회 |
| | `API_DOCS_TEST` | O | O | X | API 테스트 호출 실행 |
| **Audit** | `AUDIT_READ` | O | X | X | 감사 로그 조회 |

## 4. 토큰 (JWT) 구조 내 반영
사용자가 로그인 시 발급되는 JWT의 `authorities` 클레임에는 해당 역할에 부여된 위 권한 코드들이 리스트 형태로 포함된다.
- 예: `OPERATOR` 역할의 경우 `["PROJECT_READ", "PROJECT_WRITE", "DEPLOYMENT_READ", ...]`

현재 프로그램 기준:
- `dry-run`, `execute-stub`, runtime execution status API는 `RUNTIME_EXECUTE` / `RUNTIME_READ`를 사용한다.
- `DEPLOYMENT_EXECUTE`는 reserved 권한으로 정의만 유지하며 외부 API baseline에는 아직 연결하지 않는다.

## 5. 프로젝트 Scope 판정 규칙
- 권한 코드는 "기능 가능 여부"를, 프로젝트 scope는 "대상 프로젝트 접근 가능 여부"를 의미한다.
- 아래 두 조건을 모두 만족해야 프로젝트 하위 API 접근을 허용한다.
  - 필요한 권한 코드 보유
  - 요청한 `projectId`가 사용자 scope에 포함
- `ADMIN`은 모든 프로젝트 scope를 가진다.
- `OPERATOR`, `VIEWER`는 JWT claim 또는 서버 조회 결과의 `projectScopes`에 포함된 프로젝트만 접근 가능하다.
- scope 밖 프로젝트에 대한 조회/수정/실행 요청은 `403`을 반환한다.
- 대상 프로젝트 자체가 존재하지 않으면 scope 판정 이전에 `404`를 반환할 수 있다.
- 권장 JWT claim shape:
```json
{
  "sub": "owner@nexioone.local",
  "authorities": ["PROJECT_READ", "RUNTIME_EXECUTE"],
  "projectScopes": [1, 2, 3]
}
```

## 6. 키 관리 (Key Management)
- **Encryption Key**: AES-256-GCM 방식을 사용하며, 마스터 키는 환경 변수 또는 외부 KMS를 통해 관리한다.
- **Key Rotation**: 보안 정책에 따라 90일 주기로 암호화 키 로테이션을 권장하며, 로테이션 시 기존 데이터의 재암호화 또는 다중 키 버전 지원 프로세스를 따른다.
