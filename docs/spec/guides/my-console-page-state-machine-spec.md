# my-console Page State Machine Spec

**Status**: [Baseline]

## 1. 목적
- 페이지별 주요 상태와 이벤트 전이를 상태 머신 수준으로 고정한다.

## 2. Common Machine
| From | Event | To |
|---|---|---|
| `idle` | `ROUTE_ENTER` | `loading` |
| `loading` | `FETCH_SUCCESS_EMPTY` | `empty` |
| `loading` | `FETCH_SUCCESS_READY` | `ready` |
| `loading` | `FETCH_FORBIDDEN` | `forbidden` |
| `loading` | `FETCH_NOT_FOUND` | `not-found` |
| `loading` | `FETCH_ERROR` | `system-error` |
| `ready` | `SUBMIT` | `submitting` |
| `submitting` | `SUBMIT_SUCCESS` | `ready` |
| `submitting` | `SUBMIT_VALIDATION_ERROR` | `validation-error` |
| `submitting` | `SUBMIT_CONFLICT` | `conflict` |
| `submitting` | `SUBMIT_ERROR` | `system-error` |

## 3. Page-Specific Notes
- `FlowEditorPage`
  - `ready + DIRTY_CHANGE -> dirty-ready`
  - `dirty-ready + SAVE_SUCCESS -> ready`
- `ExecutePage`
  - `submitting + ASYNC_ACCEPTED -> polling`
  - `polling + TERMINAL_STATUS -> ready`
- `RuntimeOpsPage`
  - `ready + REFRESH -> section-loading`

## 4. 참조
- `docs/spec/guides/my-console-page-state-transition-spec.md`
- `docs/spec/guides/my-console-api-interaction-sequence-spec.md`
