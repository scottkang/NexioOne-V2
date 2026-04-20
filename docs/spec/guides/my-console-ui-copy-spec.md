# my-console UI Copy Spec

**Status**: [Baseline]

## 1. 목적
- empty state, confirm, toast, helper text, 오류 문구의 기본 카피를 고정한다.

## 2. Empty State
| Page | Title | Description | CTA |
|---|---|---|---|
| ProjectList | `프로젝트가 없습니다` | `첫 프로젝트를 생성해 작업을 시작하세요.` | `프로젝트 생성` |
| FlowList | `Flow가 없습니다` | `프로젝트에 첫 Flow를 추가하세요.` | `Flow 생성` |
| DataDefinition | `DataDefinition이 없습니다` | `재사용할 데이터 구조를 먼저 정의하세요.` | `DataDefinition 생성` |
| Connection | `Connection이 없습니다` | `Flow에서 사용할 연결 정보를 등록하세요.` | `Connection 생성` |
| Deployment | `배포 이력이 없습니다` | `Flow를 배포해 실행 기준 스냅샷을 생성하세요.` | `배포 생성` |
| RuntimeOps | `표시할 실행 상태가 없습니다` | `실행이 발생하면 상태와 요약이 여기에 표시됩니다.` | none |

## 3. Confirm Messages
| Action | Title | Body | Confirm |
|---|---|---|---|
| delete project | `프로젝트를 삭제할까요?` | `연결된 Flow와 설정을 다시 확인하세요.` | `삭제` |
| delete flow | `Flow를 삭제할까요?` | `삭제 후 복구할 수 없습니다.` | `삭제` |
| discard dirty form | `변경 사항을 버릴까요?` | `저장하지 않은 내용이 사라집니다.` | `버리기` |
| rollback deployment | `롤백을 진행할까요?` | `선택한 배포 버전으로 되돌립니다.` | `롤백` |

## 4. Toast Messages
| Scenario | Copy |
|---|---|
| create success | `생성되었습니다.` |
| update success | `저장되었습니다.` |
| delete success | `삭제되었습니다.` |
| binding save success | `Data Binding이 저장되었습니다.` |
| deployment success | `배포 요청이 등록되었습니다.` |
| execute accepted | `실행 요청이 접수되었습니다.` |

## 5. Helper Text
| Field / Area | Copy |
|---|---|
| `bindingKey` | `Flow 안에서 구분되는 논리 키를 입력하세요.` |
| `idempotencyKey` | `같은 요청의 중복 실행을 제어할 때 사용합니다.` |
| `schemaJson` | `유효한 JSON Schema object를 입력하세요.` |
| `connectionRef` | `이 노드가 참조할 연결 정보를 선택하세요.` |
| Runtime polling toggle | `자동 새로고침을 켜거나 끌 수 있습니다.` |

## 6. Forbidden / Not Found
| State | Title | Description |
|---|---|---|
| forbidden | `접근 권한이 없습니다` | `현재 권한 또는 프로젝트 범위로는 이 화면을 열 수 없습니다.` |
| not-found | `대상을 찾을 수 없습니다` | `삭제되었거나 잘못된 주소일 수 있습니다.` |

## 7. 참조
- `docs/spec/guides/my-console-error-message-mapping.md`
- `docs/spec/guides/my-console-page-state-transition-spec.md`
