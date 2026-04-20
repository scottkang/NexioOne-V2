# my-console Accessibility Checklist

**Status**: [Baseline]

## 1. 목적
- 구현 시 최소 접근성 기준을 페이지/컴포넌트 공통 체크리스트로 제공한다.

## 2. Global
- 모든 interactive element는 keyboard focus 가능
- focus ring 시각적으로 명확해야 함
- icon-only button은 `aria-label` 필요
- dialog는 focus trap 필요
- ESC close 지원 여부 명시

## 3. Form
- label과 input programmatic association 필요
- validation error는 text + aria-live 고려
- required field는 시각/접근성 양쪽으로 표시

## 4. Table
- sortable header 상태 전달 필요
- row action button은 row click과 구분 가능해야 함
- empty/loading/error 상태가 screen reader에 전달 가능해야 함

## 5. Flow Editor
- keyboard로 selected node clear 가능
- validation summary는 screen reader friendly text 제공
- node inspector heading/section label 필요

## 6. Runtime / Execute
- polling 상태 변화가 과도한 aria spam을 만들지 않도록 제한
- async terminal status는 읽을 수 있는 상태 텍스트 제공

## 7. 참조
- `docs/spec/guides/ui-component-spec.md`
