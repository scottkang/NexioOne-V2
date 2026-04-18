# Next Session Handoff

## 현재 상태
- `P0-4`는 `#13` / PR `#14`로 `main` 머지 완료되었다.
- `P0-5`는 `#15` / PR `#16`으로 `main` 머지 완료되었다.
- `P1-3`는 `#17` / PR `#18`로 `main` 머지 완료되었다.
- `P1-4`는 `#19` / PR `#20`으로 `main` 머지 완료되었다.
- backlog 재정리는 `#21` / PR `#22`로 `main` 머지 완료되었다.
- `P1-1`은 `#23` / PR `#24`로 `main` 머지 완료되었다.
- `P1-2` Connection 유형별 계약 보강 작업을 진행 중이다.
- 이번 세션 기준 작업 브랜치는 `type/25-p1-connection-profile-contract`다.
- GitHub 이슈는 `#25 [feat] P1-2 Connection 유형별 계약 보강`이다.

## 이번 세션에서 완료된 작업 / PR
- 작업:
  - `#23` / PR `#24` 머지 상태 확인
  - `#25` 이슈 생성 및 `type/25-p1-connection-profile-contract` 브랜치 생성
  - JDBC/REST/MQ/SFTP 계약 및 fixture 보강 착수
- PR:
  - `#14` merged
  - `#16` merged
  - `#18` merged
  - `#20` merged
  - `#22` merged
  - `#24` merged
  - `#25` 아직 생성 전

## 다음 세션 시작 순서
1. `#25` 작업에서 Connection 유형별 필수 field / secret key / runtime shape 정렬을 마무리한다.
2. `#25` PR을 생성하고 CI 상태를 확인한다.
3. 다음으로 문서 상태/링크 체계 정리를 시작한다.

## 우선순위 작업
- `P1-2` Connection 유형별 계약 보강
- 문서 상태/링크 체계 정리

## 리스크 / 주의 사항
- control-plane/deployment/event fixture는 baseline seed를 만드는 단계라 이후 구현 상세에 따라 일부 응답 필드는 보강될 수 있다.
- `DataDefinition` / binding 전환 규칙과 `Connection` 유형별 계약은 아직 구현 리스크가 남아 있다.
- 저장소에는 여전히 `my-backend/`, `my-console-backend/` 로컬 자산이 남아 있다.
- 문서 상태 표기와 링크 체계는 아직 일부 정리가 남아 있다.
