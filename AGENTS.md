# NexioOne Agent Notes

## GitHub Issue/PR Rules
- 기능 구현, 테스트 추가, 리팩터링 등 코드 변경 작업을 시작할 때는 반드시 GitHub 이슈를 먼저 생성한다.
- 이슈 생성 후 브랜치 규칙(`type/{issue-number}-{slug}`)에 맞는 작업 브랜치를 생성하고 해당 브랜치에서만 작업한다.
- 작업 완료 후에는 반드시 PR을 생성해 `main`으로 병합 가능한 상태까지 만든다.
- PR 생성 후에는 기본적으로 `gh pr merge --auto --merge <pr-number>`를 설정해, 필수 CI 체크가 모두 통과하면 자동 머지되도록 한다.
- auto-merge는 필수 체크가 모두 등록된 PR에만 설정하며, 충돌이 있거나 추가 수동 검토가 필요한 경우에는 예외로 둔다.
- PR이 머지되면 관련 작업 브랜치는 삭제한다. 저장소의 자동 삭제 설정을 기본으로 사용하고, 자동 삭제가 적용되지 않은 경우에는 수동으로 정리한다.
- PR 머지 전에는 가장 최근 CI 실행 결과를 확인해 관련 워크플로우가 정상적으로 완료되었는지 검토한다.
- PR 머지 전에는 GitHub Actions 체크 결과를 반드시 확인하고, 필수 체크가 모두 성공(`success`)한 경우에만 머지한다.
- 체크가 실패(`failure`) 또는 미완료(`pending`) 상태면 머지를 보류하고 원인을 정리한 뒤 재확인한다.
- CLI 기준으로 `gh pr checks <pr-number> --watch`로 완료까지 대기하고, `gh pr checks <pr-number>`로 최종 상태를 확인한 후 머지한다.
- 이슈 생성/수정 시 `.github/ISSUE_TEMPLATE/*.yml` 형식을 먼저 확인하고, 해당 섹션 구조에 맞춰 작성한다.
- PR 생성/수정 시 `.github/pull_request_template.md` 형식을 먼저 확인하고, 필수 섹션을 모두 채운다.
- `gh issue create|edit`, `gh pr create|edit`에서 본문은 `--body-file` 또는 heredoc 기반 멀티라인 문자열로 전달한다.
- 본문에 `\n` 문자열을 직접 넣어 개행을 표현하지 않는다.
- Mermaid 다이어그램 라벨 줄바꿈은 `\n` 대신 `<br/>`를 사용한다.

## Session Close Rule
- 코드 또는 문서 변경 작업이 머지되었거나 세션을 종료할 때는 반드시 다음 세션용 handoff 문서를 `docs/worklog/` 아래에 작성하거나 기존 handoff를 최신 상태로 갱신한다.
- handoff는 최소한 아래 항목을 포함한다.
  - 현재 상태
  - 이번 세션에서 완료된 작업 / PR
  - 다음 세션 시작 순서
  - 우선순위 작업
  - 리스크 / 주의 사항
- handoff 작성 형식은 `docs/worklog/_next-session-handoff-template.md`를 기준으로 한다.