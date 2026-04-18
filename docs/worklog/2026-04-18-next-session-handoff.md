# Next Session Handoff

## 현재 상태
- 로컬에만 있던 `.github` 설정 자산을 원격 저장소에 반영하는 작업을 진행 중이다.
- 이번 세션 기준 작업 브랜치는 `type/7-upload-github-config`다.
- GitHub 이슈는 `#7 [chore] GitHub 설정 자산 반영`이다.

## 이번 세션에서 완료된 작업 / PR
- 작업:
  - `.github/ISSUE_TEMPLATE`, `.github/workflows`, `.github/pull_request_template.md`, `.github/java-upgrade`를 반영 범위로 정리
  - 이번 handoff 문서를 현재 세션 기준으로 갱신
- PR:
  - 아직 생성 전

## 다음 세션 시작 순서
1. PR `#7`이 머지되었는지 확인하고 워크플로우가 기본 브랜치에 반영됐는지 점검한다.
2. 이후 남아 있는 `my-backend/`, `my-console-backend/` 반영 여부를 별도 이슈로 판단한다.
3. 구현 재개 시 `development-execution-backlog.md`의 `P0-1`부터 착수한다.

## 우선순위 작업
- `.github` 반영 PR 생성 및 병합
- GitHub 템플릿/워크플로우 반영 여부 점검
- 이후 `P0-1`, `P0-2`, `P0-3` 순으로 실제 스펙 보완 작업 재개

## 리스크 / 주의 사항
- 저장소에는 여전히 `my-backend/`, `my-console-backend/` 로컬 자산이 남아 있다.
- 새 워크플로우 반영 후 다음 PR부터 checks 동작 방식이 달라질 수 있다.
- 프론트 가이드의 reserved 권한 예시와 fixture 부재 문제는 다음 문서 보완 세션에서 이어서 정리해야 한다.
