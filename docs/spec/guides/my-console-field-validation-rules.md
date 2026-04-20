# my-console Field Validation Rules

**Status**: [Baseline]

## 1. 목적
- `my-console` 현재 프로그램 범위에서 사용하는 필드의 validation, 기본값, placeholder, help text, hidden/disabled 조건을 고정한다.

## 2. 공통 규칙
- 문자열 입력은 기본적으로 trim 후 검증한다.
- 빈 문자열은 `null`과 동일하게 취급하지 않는다. optional 필드는 빈 문자열 허용 여부를 개별 규칙으로 명시한다.
- 숫자 query param은 파싱 실패 시 validation error 대신 기본값으로 정규화한다.
- JSON editor 계열 입력은 서버 요청 전 로컬 parse validation을 먼저 수행한다.
- read-only 필드는 focus 가능하되 submit payload에는 포함하지 않는다.

## 3. 페이지별 규칙

### 3.1 LoginPage
| Field | Type | Default | Placeholder | Validation | Help / Behavior |
|---|---|---|---|---|---|
| `username` | string | `''` | `아이디를 입력하세요` | trim 후 1자 이상 | browser autofill 허용 |
| `password` | string | `''` | `비밀번호를 입력하세요` | 1자 이상 | mask 표시, reveal toggle 선택 가능 |

### 3.2 Project
| Field | Type | Default | Placeholder | Validation | Help / Behavior |
|---|---|---|---|---|---|
| `name` | string | `''` | `예: Order Processing` | 1~100자 | 동일 프로젝트명 중복 여부는 서버 응답 기준 |
| `description` | string | `''` | `프로젝트 설명` | 0~500자 | 빈 문자열 허용 |

### 3.3 Flow
| Field | Type | Default | Placeholder | Validation | Help / Behavior |
|---|---|---|---|---|---|
| `name` | string | `''` | `예: Customer Sync Flow` | 1~100자 | 저장 전 trim |
| `description` | string | `''` | `Flow 설명` | 0~500자 | optional |
| `nodes` | array | `[]` | - | 최소 `START` 1개, `END` 1개 이상 | 미지원 node 금지 |
| `edges` | array | `[]` | - | cycle 금지, source/target required | dangling edge 금지 |

### 3.4 DataDefinition
| Field | Type | Default | Placeholder | Validation | Help / Behavior |
|---|---|---|---|---|---|
| `name` | string | `''` | `예: CustomerRequest` | 1~100자 | trim |
| `description` | string | `''` | `DataDefinition 설명` | 0~500자 | optional |
| `schemaJson` | object | `{}` | `JSON Schema 입력` | 유효한 JSON object | parse 실패 시 저장 버튼 비활성화 |
| `metadata.dataFormat` | enum/null | `null` | `선택` | enum only | `schemaJson.x-nexio`에 반영 |
| `metadata.structureType` | enum/null | `null` | `선택` | enum only | `schemaJson.x-nexio`에 반영 |

### 3.5 Connection Common
| Field | Type | Default | Placeholder | Validation | Help / Behavior |
|---|---|---|---|---|---|
| `name` | string | `''` | `예: main-order-db` | 1~100자 | trim |
| `type` | enum | none | `유형 선택` | `JDBC/REST/MQ/SFTP` 중 하나 | 생성 후 변경 비권장, edit 시 disabled 권장 |

#### JDBC
| Field | Default | Placeholder | Validation | Help / Behavior |
|---|---|---|---|---|
| `url` | `''` | `jdbc:postgresql://host:5432/db` | required, prefix `jdbc:` | full DSN 입력 |
| `username` | `''` | `db_user` | required, 1~120자 | secret 아님 |
| `driverClassName` | `''` | `org.postgresql.Driver` | optional, 0~255자 | 빈 값 허용 |
| `schema` | `''` | `public` | optional, 0~120자 | trim |
| `catalog` | `''` | `appdb` | optional, 0~120자 | trim |
| `validationQuery` | `''` | `SELECT 1` | optional, 0~500자 | trim |
| `connectionTimeoutMs` | `30000` | `30000` | optional, 1~120000 | 숫자만 |
| `readOnly` | `false` | - | boolean | switch |

#### REST
| Field | Default | Placeholder | Validation | Help / Behavior |
|---|---|---|---|---|
| `baseUrl` | `''` | `https://api.example.com` | required, absolute URL | trailing slash 허용 |
| `authenticationType` | `''` | `NONE / BASIC / API_KEY` | optional enum | 값에 따라 secret field 노출 |
| `connectTimeoutMs` | `30000` | `30000` | optional, 1~120000 | 숫자 |
| `readTimeoutMs` | `30000` | `30000` | optional, 1~120000 | 숫자 |
| `defaultHeaders` | `{}` | `{\"X-App\":\"nexio\"}` | optional object | JSON editor |
| `apiKeyHeaderName` | `''` | `X-API-Key` | conditional | `authenticationType=API_KEY`일 때만 표시 |

#### MQ
| Field | Default | Placeholder | Validation | Help / Behavior |
|---|---|---|---|---|
| `brokerUrl` | `''` | `amqp://broker:5672` | required | absolute connection string |
| `destinationType` | none | `QUEUE / TOPIC` | required enum | select |
| `destinationName` | `''` | `orders.queue` | required, 1~200자 | trim |
| `clientId` | `''` | `nexio-client` | optional | trim |
| `ackMode` | `''` | `AUTO / CLIENT` | optional enum | select |
| `consumerGroup` | `''` | `order-workers` | optional | trim |
| `virtualHost` | `''` | `/` | optional | trim |
| `sslEnabled` | `false` | - | boolean | switch |

#### SFTP
| Field | Default | Placeholder | Validation | Help / Behavior |
|---|---|---|---|---|
| `host` | `''` | `sftp.example.com` | required, hostname/IP | trim |
| `port` | `22` | `22` | required, 1~65535 | 숫자 |
| `username` | `''` | `fileuser` | required, 1~120자 | trim |
| `basePath` | `''` | `/inbound/orders` | optional | trim |
| `knownHostsPolicy` | `''` | `STRICT / ACCEPT_NEW / INSECURE` | optional enum | select |
| `connectTimeoutMs` | `30000` | `30000` | optional, 1~120000 | 숫자 |
| `charset` | `UTF-8` | `UTF-8` | optional, 1~32자 | trim |

### 3.6 FlowBinding
| Field | Type | Default | Placeholder | Validation | Help / Behavior |
|---|---|---|---|---|---|
| `role` | enum | none | `역할 선택` | required | role별 duplicate 규칙 적용 |
| `bindingKey` | string | `''` | `예: requestBody` | required, 1~100자 | 같은 Flow에서 `role + bindingKey` 유일 |
| `displayOrder` | integer | `0` | `0` | required, 0 이상 | 숫자 |
| `required` | boolean | `false` | - | boolean | switch |
| `dataDefinitionId` | integer | none | `DataDefinition 선택` | required, positive | cross-project 참조 금지 |

### 3.7 Deployment
| Field | Type | Default | Placeholder | Validation | Help / Behavior |
|---|---|---|---|---|---|
| `flowId` | integer | none | `Flow 선택` | required, positive | select |
| `version` | string | `''` | `예: v1` | required, 1~64자 | create only if UI exposes manual input |
| `description` | string | `''` | `배포 설명` | 0~500자 | optional |
| `status` | enum | API value | `상태 선택` | transition map 내 값만 허용 | invalid transition 금지 |

### 3.8 Execute
| Field | Type | Default | Placeholder | Validation | Help / Behavior |
|---|---|---|---|---|---|
| `deploymentId` | integer/null | `null` | `배포 선택` | optional, positive | 없으면 최신/직접 실행 모드 |
| `inputContext` | object | `{}` | `실행 입력 JSON` | required object | parse 실패 시 submit 금지 |
| `options.trace` | boolean | `true` | - | boolean | 기본 활성화 |
| `options.validateOnly` | boolean | `false` | - | boolean | dry-run only |
| `options.async` | boolean | `true` | - | boolean | execute-stub only |
| `idempotencyKey` | string | `''` | `선택: 재시도 식별 키` | optional, 0~120자 | whitespace-only 금지 |

## 4. Secret Field Rules
- secret 입력 필드는 항상 empty default로 렌더링한다.
- edit 화면에서 secret 미입력은 "기존 값 유지" 의미다.
- `secretPresence[key] = true`면 "설정됨" 뱃지만 표시하고 원문은 노출하지 않는다.

## 5. 참조
- `docs/spec/guides/my-console-page-field-matrix.md`
- `docs/spec/data/connection-profile-contract.md`
- `docs/spec/data/data-definition-structure-metadata-guide.md`
