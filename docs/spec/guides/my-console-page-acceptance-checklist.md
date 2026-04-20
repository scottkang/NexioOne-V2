# my-console Page Acceptance Checklist

**Status**: [Baseline]

## 1. 목적
- 페이지 구현 완료 기준을 더 세밀한 체크리스트로 고정한다.

## 2. Checklist
| Page | Acceptance |
|---|---|
| LoginPage | submit/loading/error/success redirect 확인, keyboard submit 가능 |
| ProjectListPage | list/empty/create/edit/delete/forbidden 동작, sort/page 유지 |
| ProjectDetailPage | detail/tabs/edit/delete/not-found/forbidden 확인 |
| FlowListPage | list/create/search/basic navigation 확인 |
| FlowEditorPage | palette/canvas/inspector/save/dirty confirm/local validation/server validation 확인 |
| DataDefinitionPage | list/detail/schema edit/parse error/save/delete 확인 |
| ConnectionPage | type-specific fields/secret preserve/save/delete/forbidden 확인 |
| FlowBindingPage | add/remove/reorder/save/conflict handling 확인 |
| DeploymentPage | create/status change/rollback/invalid transition 확인 |
| RuntimeOpsPage | cards load/polling toggle/execution detail 확인 |
| ExecutePage | dry-run/sync execute/async execute/conflict/result render 확인 |

## 3. 참조
- `docs/spec/guides/my-console-page-test-scenario-matrix.md`
