# my-console Frontend Code Structure Spec

**Status**: [Baseline]

## 1. 목적
- `my-console` 프론트 구현 시 폴더 구조, query key, hook, mapper, page/container 분리 기준을 고정한다.

## 2. Directory Baseline
- `src/app`
  - router, providers, query client, app shell bootstrap
- `src/pages`
  - route-level page components
- `src/features/{domain}`
  - domain-specific container, form, table, panel
- `src/components/common`
  - reusable primitives
- `src/components/layout`
  - shell/header/sidebar/breadcrumb
- `src/hooks/queries`
  - react-query hooks
- `src/lib/api`
  - fetch client, DTO types, api functions
- `src/lib/mappers`
  - DTO -> view-model mappers
- `src/lib/permissions`
  - authority/scope helpers
- `src/lib/validation`
  - local validation functions

## 3. Layer Rules
- `pages`: route param, query param, high-level composition
- `features`: domain 화면 조합과 local UI state
- `components/common`: domain knowledge 없음
- `lib/api`: side effect and transport only
- `lib/mappers`: server DTO를 page VM으로 변환

## 4. Query Hook Naming
- `useProjectListQuery`
- `useProjectDetailQuery`
- `useCreateProjectMutation`
- `useFlowDetailQuery`
- `useSaveFlowMutation`
- `useRuntimeSkeletonQuery`
- `useExecutionStatusQuery`

규칙:
- query는 `Query`, mutation은 `Mutation` suffix
- query key는 domain-first naming 사용

## 5. Mapper Rules
- `mapProjectResponseToListItemVm`
- `mapFlowDetailToEditorVm`
- `mapApiErrorToErrorVm`
- mapper는 side effect 없음

## 6. Page / Container Split
- page:
  - route param parse
  - permission gate
  - loader/error boundary composition
- container:
  - query/mutation orchestration
  - mapper 호출
  - child component props 조립
- presentational component:
  - render + event emit only

## 7. 참조
- `docs/spec/guides/frontend-development-guide.md`
- `docs/spec/guides/my-console-api-interaction-sequence-spec.md`
- `docs/spec/guides/my-console-component-prop-event-matrix.md`
