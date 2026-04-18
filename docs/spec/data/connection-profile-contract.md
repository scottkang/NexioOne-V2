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

Required `secrets`:
- `password`

Required `secretsRef`:
- `passwordKey`

Validation Rules:
- `url`은 `jdbc:` prefix 필수
- `password`는 Control Plane 조회 응답에 포함 금지
- metadata import 대상은 `JDBC`만 허용

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
- `authenticationType=BASIC`이면 `username` + `password` 필요
- `authenticationType=BEARER`이면 `token` 필요
- `authenticationType=API_KEY`이면 `apiKeyHeaderName` + `apiKey` 필요

### 4.3 MQ
Required `properties`:
- `brokerUrl`
- `destinationType` (`QUEUE`, `TOPIC`)
- `destinationName`

Optional `properties`:
- `clientId`
- `ackMode`
- `consumerGroup`

Optional `secrets`:
- `username`
- `password`

Allowed `secretsRef`:
- `usernameKey`
- `passwordKey`

Validation Rules:
- `destinationName`은 공백 문자열 금지
- 인증이 필요한 broker면 `username/password` 필요

### 4.4 SFTP
Required `properties`:
- `host`
- `port`
- `username`

Optional `properties`:
- `basePath`
- `knownHostsPolicy` (`STRICT`, `TRUST_ALL`)
- `connectTimeoutMs`

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

## 5. Secret 처리 규칙
- Control Plane request에서는 secret 원문 입력 허용
- Control Plane response에서는 secret 원문 반환 금지
- runtime request에서는 secret 원문 대신 `secretsRef`만 전달
- secret 이름은 타입별 허용 키만 사용
- `properties`에 `password`, `token`, `secret`, `privateKey`, `passphrase` 포함 금지

## 6. 상태 및 검증
- `ACTIVE`: 정상 저장 및 사용 가능
- `INVALID`: validation 실패 또는 필수 필드 누락
- `INACTIVE`: 사용 중지

최소 검증 시점:
- 저장 시 request validation
- deployment snapshot 생성 시 참조 무결성 검증
- runtime 전달 직전 `connectionRef -> connectionProfiles.id` 일치 검증

## 7. Deployment / Runtime 반영 규칙
- deployment snapshot에는 실행에 필요한 연결의 비민감 정보와 `secretsRef`가 포함된다.
- 미참조 connection은 runtime 요청에 포함하지 않을 수 있다.
- `FlowNode.connectionRef`는 snapshot에 포함된 `connectionProfiles[].id`와 일치해야 한다.

## 8. 참조
- `docs/spec/api/control-plane-api-baseline.md`
- `docs/spec/api/internal-execution-schema.md`
- `docs/spec/data/data-definition-import-from-db.md`
