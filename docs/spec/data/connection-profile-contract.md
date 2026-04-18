# Connection Profile Contract

**Status**: [Baseline]

## 1. 목적
- Control Plane의 Connection 저장 계약과 runtime 전달용 `connectionProfiles` 계약을 연결한다.
- 유형별 필수/옵션 필드, secret 처리, validation 규칙을 고정한다.

## 2. 모델 구분

### 2.1 Control Plane Connection
- 저장 주체: `my-console-backend`
- API 기준: `control-plane-api-baseline.md`
- 특징:
  - 사용자가 생성/수정하는 원본 연결 정보
  - secret은 write-only 입력으로 받고 암호화 저장
  - 조회 응답에서는 secret 원문을 반환하지 않음

### 2.2 Runtime Connection Profile
- 전달 주체: `my-console-backend`
- 수신 주체: `my-backend`
- API 기준: `internal-execution-schema.md`
- 특징:
  - deployment 또는 execution 요청 시점에 필요한 필드만 runtime으로 전달
  - 비민감 정보는 `properties`
  - 민감 정보는 `secretsRef`

## 3. 공통 필드

### 3.1 Control Plane Shape
```json
{
  "id": 12,
  "name": "source-db",
  "type": "JDBC",
  "properties": {
    "url": "jdbc:postgresql://db:5432/app",
    "username": "app"
  },
  "secrets": {
    "password": "write-only"
  }
}
```

### 3.2 Runtime Shape
```json
{
  "id": "source-db",
  "type": "JDBC",
  "properties": {
    "url": "jdbc:postgresql://db:5432/app",
    "username": "app"
  },
  "secretsRef": {
    "passwordKey": "secret/project-1/source-db/password"
  }
}
```

### 3.3 Secret Key Naming Rule
- runtime `secretsRef` key 이름은 타입별 허용 키만 사용한다.
- key path 형식은 기본적으로 `secret/project-{projectId}/{connectionName}/{secretName}`를 따른다.
- 같은 connection에서 여러 secret을 쓰는 경우에도 secret 이름 suffix는 타입별 고정 이름을 사용한다.

예:
- JDBC: `passwordKey`
- REST BASIC: `usernameKey`, `passwordKey`
- REST BEARER: `tokenKey`
- REST API_KEY: `apiKeyKey`
- MQ: `usernameKey`, `passwordKey`
- SFTP: `passwordKey`, `privateKeyKey`, `passphraseKey`

## 4. 유형별 계약

### 4.1 JDBC
Required `properties`:
- `url`
- `username`

Optional `properties`:
- `driverClassName`
- `schema`
- `catalog`
- `validationQuery`
- `connectionTimeoutMs`
- `readOnly`

Required `secrets`:
- `password`

Required `secretsRef`:
- `passwordKey`

Validation Rules:
- `url`은 `jdbc:` prefix 필수
- `password`는 Control Plane 조회 응답에 포함 금지
- metadata import 대상은 `JDBC`만 허용
- runtime 전달 시 `properties.url`, `properties.username`, `secretsRef.passwordKey`는 항상 포함돼야 한다

### 4.2 REST
Required `properties`:
- `baseUrl`

Optional `properties`:
- `connectTimeoutMs`
- `readTimeoutMs`
- `defaultHeaders`
- `authenticationType` (`NONE`, `BASIC`, `BEARER`, `API_KEY`)
- `apiKeyHeaderName`

Optional `secrets`:
- `username`
- `password`
- `token`
- `apiKey`

Allowed `secretsRef`:
- `usernameKey`
- `passwordKey`
- `tokenKey`
- `apiKeyKey`

Validation Rules:
- `baseUrl`은 `http://` 또는 `https://`
- `authenticationType` 기본값은 `NONE`
- `authenticationType=BASIC`이면 `username` + `password` 필요
- `authenticationType=BEARER`이면 `token` 필요
- `authenticationType=API_KEY`이면 `apiKeyHeaderName` + `apiKey` 필요
- `authenticationType=NONE`이면 secret 없이 저장 가능
- runtime 전달 시 secret 원문 대신 선택된 인증 방식에 필요한 `secretsRef` key만 포함한다

### 4.3 MQ
Required `properties`:
- `brokerUrl`
- `destinationType` (`QUEUE`, `TOPIC`)
- `destinationName`

Optional `properties`:
- `clientId`
- `ackMode`
- `consumerGroup`
- `virtualHost`
- `sslEnabled`

Optional `secrets`:
- `username`
- `password`

Allowed `secretsRef`:
- `usernameKey`
- `passwordKey`

Validation Rules:
- `brokerUrl`은 `amqp://` 또는 `amqps://` prefix 권장
- `destinationName`은 공백 문자열 금지
- 인증이 필요한 broker면 `username/password` 필요
- `destinationType=QUEUE|TOPIC` 외 값은 허용하지 않음
- runtime 전달 시 `destinationType`, `destinationName`은 항상 포함돼야 한다

### 4.4 SFTP
Required `properties`:
- `host`
- `port`
- `username`

Optional `properties`:
- `basePath`
- `knownHostsPolicy` (`STRICT`, `TRUST_ALL`)
- `connectTimeoutMs`
- `charset`

Optional `secrets`:
- `password`
- `privateKey`
- `passphrase`

Allowed `secretsRef`:
- `passwordKey`
- `privateKeyKey`
- `passphraseKey`

Validation Rules:
- `port`는 `1..65535`
- `password` 또는 `privateKey` 중 최소 하나 필요
- `knownHostsPolicy` 기본값은 `STRICT`
- `privateKey`를 사용하면 multiline secret도 허용하되 runtime에는 `privateKeyKey`만 전달한다

## 4.5 유형별 Control Plane / Runtime 매핑
| Type | Control Plane Required Properties | Required Secret Keys | Runtime Required Properties | Runtime Required `secretsRef` |
|---|---|---|---|---|
| `JDBC` | `url`, `username` | `password` | `url`, `username` | `passwordKey` |
| `REST` (`NONE`) | `baseUrl` | 없음 | `baseUrl` | 없음 |
| `REST` (`BASIC`) | `baseUrl`, `authenticationType` | `username`, `password` | `baseUrl`, `authenticationType` | `usernameKey`, `passwordKey` |
| `REST` (`BEARER`) | `baseUrl`, `authenticationType` | `token` | `baseUrl`, `authenticationType` | `tokenKey` |
| `REST` (`API_KEY`) | `baseUrl`, `authenticationType`, `apiKeyHeaderName` | `apiKey` | `baseUrl`, `authenticationType`, `apiKeyHeaderName` | `apiKeyKey` |
| `MQ` | `brokerUrl`, `destinationType`, `destinationName` | broker auth 시 `username`, `password` | `brokerUrl`, `destinationType`, `destinationName` | broker auth 시 `usernameKey`, `passwordKey` |
| `SFTP` | `host`, `port`, `username` | `password` 또는 `privateKey` | `host`, `port`, `username` | `passwordKey` 또는 `privateKeyKey` |

## 5. Secret 처리 규칙
- Control Plane request에서는 secret 원문 입력 허용
- Control Plane response에서는 secret 원문 반환 금지
- runtime request에서는 secret 원문 대신 `secretsRef`만 전달
- secret 이름은 타입별 허용 키만 사용
- `properties`에 `password`, `token`, `secret`, `privateKey`, `passphrase` 포함 금지
- secret 저장 시 암호화 정본은 `my-console-backend`가 관리하고 runtime은 key reference만 소비한다
- rotation 시 runtime snapshot은 다음 배포 생성 시점부터 새 `secretsRef` 값을 사용한다

## 6. 상태 및 검증
- `ACTIVE`: 정상 저장 및 사용 가능
- `INVALID`: validation 실패 또는 필수 필드 누락
- `INACTIVE`: 사용 중지

최소 검증 시점:
- 저장 시 request validation
- deployment snapshot 생성 시 참조 무결성 검증
- runtime 전달 직전 `connectionRef -> connectionProfiles.id` 일치 검증

유형별 최소 validation error 기준:
- JDBC: `url` prefix, `username` 공백, `password` 누락
- REST: `baseUrl` scheme, `authenticationType`별 필수 secret 누락
- MQ: `destinationType`, `destinationName`, broker auth secret 누락
- SFTP: `host`, `port`, `password/privateKey` 중 최소 하나 누락

## 7. Deployment / Runtime 반영 규칙
- deployment snapshot에는 실행에 필요한 연결의 비민감 정보와 `secretsRef`가 포함된다.
- 미참조 connection은 runtime 요청에 포함하지 않을 수 있다.
- `FlowNode.connectionRef`는 snapshot에 포함된 `connectionProfiles[].id`와 일치해야 한다.
- deployment snapshot은 connection의 `type`, `properties`, `secretsRef` key 이름만 고정하며 secret 원문을 포함하지 않는다.
- runtime이 실제 connector를 호출하지 않는 Stage 1에서도 validation과 trace 목적으로 동일 shape를 유지한다.

## 8. 참조
- `docs/spec/api/control-plane-api-baseline.md`
- `docs/spec/api/internal-execution-schema.md`
- `docs/spec/data/data-definition-import-from-db.md`
- `docs/spec/security/security-encryption-design.md`
