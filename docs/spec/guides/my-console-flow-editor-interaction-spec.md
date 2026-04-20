# my-console Flow Editor Interaction Spec

**Status**: [Baseline]

## 1. 목적
- Flow Editor의 palette, canvas, node 생성/삭제/복제, edge 연결, inspector 열기 규칙을 고정한다.

## 2. Palette Rules
- palette groups:
  - `Control`: `START`, `END`
  - `Transform`: `MAPPING`
  - `Connector`: `REST_CLIENT`, `SQL_EXECUTOR`
- `START`는 새 flow 생성 시 기본 포함
- `END`는 palette에서 추가 가능

## 3. Canvas Interaction
- node create:
  - palette drag-drop 또는 click-to-add
  - 새 node는 viewport center 근처 기본 위치
- node select:
  - single select only
  - select 시 inspector open
- node delete:
  - delete key 또는 action button
  - `START` 마지막 1개는 삭제 금지
- node duplicate:
  - selected node 기준 복제
  - `id` 재생성, position offset 적용

## 4. Edge Interaction
- drag from output handle to input handle
- invalid target는 hover 단계에서 연결 금지 표시
- edge delete는 select 후 delete key 또는 context action
- auto-layout은 이번 프로그램 비범위

## 5. Inspector Interaction
- node 선택 시 우측 inspector 즉시 갱신
- inspector field 변경은 debounced local state 반영 허용
- `connectionRef` field는 node type과 connection type이 맞지 않으면 option disabled
- validation error가 있으면 field 근처와 summary panel에 동시 표시

## 6. Save / Validate
- explicit save button만 영속화 트리거
- local validate는 save 전에 항상 수행
- server validate 결과는 local validate와 병합해 표시

## 7. Deep Link Rules
- `nodeId` query가 있으면 해당 node를 선택하고 inspector를 연다.
- `panel=node-inspector`면 우측 panel open 상태 유지

## 8. Keyboard
- `Delete`/`Backspace`: selected node/edge 삭제
- `Esc`: selection clear
- `Cmd/Ctrl + S`: save action

## 9. 참조
- `docs/spec/guides/my-console-flow-editor-node-contracts.md`
- `docs/spec/guides/my-console-routing-ia-spec.md`
