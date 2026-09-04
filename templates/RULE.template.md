# docs/RULE.md

이 프로젝트는 pdca-skill v1 체계를 따른다. 절차 본문은 스킬(`pdca-plan` `pdca-design` `pdca-do` `pdca-close` `cycle-propose` `backlog-sync`)이 정본이고, 이 파일은 **이 프로젝트만의 예외와 훅**만 담는다.

## 검증 수단

Plan §5의 SC에 붙일 수 있는 수단. 여기 없는 수단을 SC에 쓰지 않는다.

| 수단 | 방법 | 비고 |
|---|---|---|
| 자동 테스트 | `{명령}` | 테스트 위치 `{경로}`. CI가 사이클 브랜치 푸시마다 실행 |
| 자동 테스트(느림·외부 자원) | `{명령}` | CI 밖. SC 검증 수단이면 do §3에 수동 실행 결과를 남긴다. 없으면 이 행을 지운다 |
| 사용자 육안 | {무엇을 어디서 보나} | 사용자가 확인한 날짜를 do §3에 남긴다 |
| 로그·수치 | {무엇을 어떻게 측정하나} | |

## 브랜치 · CI

- 통합 브랜치: `develop`. 릴리즈 포인터: `main`. 다르면 여기 적는다.
- 사이클 브랜치는 `{버전}-{사이클명}`, develop에서 분기.
- CI 확인 명령: `{예: gh run list --branch <브랜치> --limit 1}`. 없으면 "사용자 확인"이라 쓴다. close가 게이트와 main 전진 전에 이걸 쓴다.

## 종료 훅

close 6단계(docs 커밋 직전)에 이 프로젝트가 추가로 하는 일. 없으면 "없음".

1. {예: README.md 최신화}
2. {예: `ProjectSettings/ProjectSettings.asset`의 `bundleVersion`을 버전으로, `AndroidBundleVersionCode`를 +1}

## 릴리즈노트

- 제목 이모지: {예: 🎣}
- 톤: {예: 게임 공지 / 개발자 도구 안내}
- 제품명: {release 문서 제목에 쓸 이름. 비우면 `.pdcarc.json`의 프로젝트명}

## 예외

{이 프로젝트가 표준 규칙에서 벗어나는 것. 없으면 "없음".}
