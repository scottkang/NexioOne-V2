# my-console Query Mutation Spec

**Status**: [Baseline]

## 1. 목적
- React Query 기준의 query key, staleTime, retry, enabled, invalidate 정책을 고정한다.

## 2. Query Rules
| Query | Key | staleTime | retry | enabled |
|---|---|---|---|---|
| Project list | `['projects','list', projectId?, page, size, sort]` | `5m` | `1` | always |
| Project detail | `['projects','detail', projectId]` | `5m` | `1` | `projectId != null` |
| Flow list | `['flows','list', projectId, page, size, sort, q]` | `1m` | `1` | `projectId != null` |
| Flow detail | `['flows','detail', projectId, flowId]` | `0` | `1` | `projectId && flowId` |
| DataDefinition list | `['dataDefinitions','list', projectId]` | `1m` | `1` | `projectId != null` |
| Connection list | `['connections','list', projectId]` | `1m` | `1` | `projectId != null` |
| Flow bindings | `['flowBindings','detail', projectId, flowId]` | `0` | `0` | `projectId && flowId` |
| Deployment list | `['deployments','list', projectId, flowId?, status?]` | `30s` | `1` | `projectId != null` |
| Runtime skeleton | `['runtime','skeleton', projectId]` | `15s` | `1` | `projectId != null` |
| Runtime execution | `['runtime','execution', projectId, executionId]` | `0` | `0` | `projectId && executionId` |

## 3. Mutation Rules
| Mutation | Success Invalidate |
|---|---|
| create/update/delete project | `projects.list`, `projects.detail` |
| create/update/delete flow | `flows.list`, `flows.detail` |
| create/update/delete data definition | `dataDefinitions.list` |
| create/update/delete connection | `connections.list` |
| save flow bindings | `flowBindings.detail` |
| deployment create/status/rollback | `deployments.list` |
| dry-run | none |
| execute-stub | none |

## 4. Error Rules
- `401`: global auth recovery
- `403`, `404`: page state handling, retry 최소화
- `409`: automatic retry 금지

## 5. 참조
- `docs/spec/guides/my-console-api-interaction-sequence-spec.md`
- `docs/spec/guides/my-console-frontend-code-structure-spec.md`
