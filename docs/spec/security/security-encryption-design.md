# Security & Encryption Design Specification

## 1. 개요
본 문서는 NexioOne 시스템 내의 민감 정보(연결 비밀번호, API Key 등)를 안전하게 관리하기 위한 암호화 알고리즘, 키 관리 정책 및 데이터 처리 메커니즘을 정의한다.

## 2. 암호화 알고리즘 표준
- **데이터 암호화 (Encryption at Rest)**: `AES-256-GCM` (Authenticated Encryption)
  - IV(Initialization Vector)는 매 실행 시마다 고유하게 생성하여 저장한다.
- **키 유도 (Key Derivation)**: `PBKDF2WithHmacSHA256`
  - 암호화 키를 생성할 때 고유한 Salt와 최소 10,000회 이상의 Iteration을 사용한다.
- **해싱 (Hashing)**: `SHA-512` 또는 `BCrypt` (비밀번호 저장 시)

## 3. 키 관리 전략 (Key Management)

### 3.1 마스터 키 (Master Key)
- 시스템의 모든 하위 키를 보호하는 최상위 키이다.
- 환경 변수(`NEXIO_MASTER_KEY`) 또는 K8S Secret, 외부 KMS(AWS KMS/Vault)를 통해 주입된다.
- 소스 코드나 설정 파일에 직접 기록해서는 안 된다.

### 3.2 프로젝트별 키 (Project-scoped Key)
- 각 프로젝트 생성 시 고유한 프로젝트 암호화 키가 생성된다.
- 해당 프로젝트 내의 Connection 설정 정보는 이 키로 암호화된다.

## 4. 데이터 암호화 프로세스 (Connection Config)
1. 사용자가 UI에서 DB 비밀번호 입력.
2. `my-console-backend` 수신.
3. 마스터 키 + 프로젝트 Salt를 이용해 암호화 키 생성.
4. AES-256-GCM 알고리즘으로 암호화.
5. `TB_CONNECTION` 테이블의 `config` 필드에 JSON 포맷으로 저장 (IV 포함).

### 4.1 Connection Secret 저장 범위
- Control Plane 저장 시 암호화 대상은 `Connection.secrets` 하위 값이다.
- `properties`는 비민감 설정만 저장하며 암호화 대상이 아니다.
- 유형별 허용 secret key:
  - JDBC: `password`
  - REST: `username`, `password`, `token`, `apiKey`
  - MQ: `username`, `password`
  - SFTP: `password`, `privateKey`, `passphrase`

### 4.2 Runtime 전달 규칙
- runtime 요청에는 secret 원문을 전달하지 않는다.
- `my-console-backend`는 deployment snapshot 또는 execute request 생성 시 `secretsRef`만 포함한다.
- `secretsRef` key 이름은 `connection-profile-contract.md`의 타입별 허용 key를 따른다.
- secret rotation 후 새 reference는 다음 deployment snapshot 생성 시점부터 반영한다.

## 5. 민감 정보 마스킹 (Data Masking)
- **로그 출력**: 로그 서비스 전송 전 `password`, `secret`, `token`, `key` 등의 키워드가 포함된 필드는 자동으로 `***` 처리한다.
- **UI 노출**: 비밀번호 필드는 조회 API 호출 시 값을 포함하지 않거나(Null), 마스킹된 상태로 반환한다.

### 5.1 Connection 조회 마스킹 규칙
- Connection 상세/목록 응답은 `secrets`, `secretsRef` 원문을 반환하지 않는다.
- 프론트엔드는 secret 존재 여부를 별도 boolean 또는 placeholder로만 표시할 수 있다.
- runtime/logging/event payload에는 `secretsRef` key 이름 외의 secret 값이 노출되면 안 된다.
