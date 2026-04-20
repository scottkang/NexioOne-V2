# my-console Error Message Mapping

**Status**: [Baseline]

## 1. 목적
- API 오류 코드를 `my-console` UI 문구와 처리 방식에 매핑한다.

## 2. 공통 규칙
- `details[].field`가 있으면 해당 필드에 인라인 오류를 연결한다.
- 사용자 문구는 짧고 조치 가능해야 한다.
- 운영 추적이 필요한 경우 상세 영역 또는 개발자용 panel에 `traceId`를 노출한다.

## 3. 코드 매핑
| Code | User Message | UI Handling |
|---|---|---|
| `SYS-0001` | `처리 중 문제가 발생했습니다. 잠시 후 다시 시도해 주세요.` | toast + retry |
| `SYS-1001` | `요청한 정보를 찾을 수 없습니다.` | not-found state |
| `SYS-1002` | `입력값을 다시 확인해 주세요.` | field error + summary banner |
| `SYS-1100` | `로그인이 필요합니다. 다시 로그인해 주세요.` | refresh 시도 후 login 이동 |
| `SYS-1101` | `이 작업을 수행할 권한이 없습니다.` | forbidden state / disable |
| `SYS-1409` | `다른 사용자의 변경 사항이 먼저 반영되었습니다. 최신 데이터를 확인해 주세요.` | conflict banner + reload CTA |
| `SYS-1503` | `연결된 서비스를 일시적으로 사용할 수 없습니다.` | retry banner |
| `ADM-2001` | `배포 버전이 충돌했습니다. 목록을 새로고침해 주세요.` | deployment conflict dialog |
| `ADM-2002` | `현재 상태에서는 이 배포 작업을 수행할 수 없습니다.` | status action disabled + banner |
| `BE-2001` | `Flow 설정이 올바르지 않습니다. 필수 설정을 확인해 주세요.` | flow/execute validation panel |
| `BE-2101` | `참조한 연결 정보를 찾을 수 없습니다.` | node/connection field error |
| `BE-2102` | `지원하지 않는 노드 또는 설정입니다.` | flow editor validation panel |
| `BE-2103` | `같은 요청 키가 다른 실행에 이미 사용되었습니다.` | execute conflict panel |
| `BE-2301` | `Stub 실행이 실패했습니다. 입력과 노드 설정을 확인해 주세요.` | execute result error panel |
| `LOG-3001` | `실행 이벤트 형식이 올바르지 않습니다.` | operator-oriented banner |
| `LOG-3002` | `실행 로그를 저장하지 못했습니다.` | non-blocking warning toast |

## 4. 필드 매핑 예시
| `details.field` | UI Target |
|---|---|
| `name` | create/edit form의 `name` input |
| `description` | textarea |
| `schemaJson` | JSON editor root error |
| `inputContext` | Execute JSON editor root error |
| `bindings[0].bindingKey` | 해당 binding row key input |
| `nodes[2].config.sql` | Flow editor inspector SQL field |
| `nodes[0].type` | Flow editor 상단 validation panel |

## 5. 페이지별 문구 원칙
- 목록 페이지: toast 또는 banner 중심
- dialog/form 페이지: summary banner + first field inline error
- FlowEditor: 상단 validation summary + inspector inline error
- Execute: request form error와 result error를 구분

## 6. 참조
- `docs/spec/api/error-response-baseline.md`
- `docs/spec/guides/my-console-page-state-transition-spec.md`
