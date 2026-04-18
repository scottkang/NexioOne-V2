# Next Session Handoff

## 현재 상태
- `P0-1` 범위 단일화와 `P0-2` 외부 API 기준 정렬 작업을 진행 중이다.
- 이번 세션 기준 작업 브랜치는 `type/9-p0-scope-api-alignment`다.
- GitHub 이슈는 `#9 [feat] P0-1 P0-2 범위 및 외부 API 기준 정렬`이다.

## 이번 세션에서 완료된 작업 / PR
- 작업:
  - `release-scope.md`, `product-spec.md`, 모듈 문서의 범위 해석 기준 정렬
  - `api-spec.md`를 `control-plane-api-baseline.md`와 함께 외부 API 단일 계약 묶음으로 정리
  - 이번 handoff 문서를 현재 세션 기준으로 갱신
- PR:
  - 아직 생성 전

## 다음 세션 시작 순서
1. PR `#9` 머지 여부를 확인한다.
2. 이어서 `P0-3` 내부 실행 계약과 상태 조회 ownership 정렬 작업에 착수한다.
3. 다음으로 `P0-4`, `P0-5`를 fixture 중심으로 보완한다.

## 우선순위 작업
- `P0-3` 내부 실행 계약과 상태 조회 ownership 확정
- `P0-4` runtime event / logging 계약 고정
- `P0-5` runtime fixture 및 의미론 고정

## 리스크 / 주의 사항
- `P0-1`, `P0-2`는 문서 해석 정렬이 중심이라 fixture/OpenAPI 산출물까지는 아직 만들지 않았다.
- 저장소에는 여전히 `my-backend/`, `my-console-backend/` 로컬 자산이 남아 있다.
- 프론트 가이드의 reserved 권한 예시와 fixture 부재 문제는 다음 문서 보완 세션에서 이어서 정리해야 한다.
