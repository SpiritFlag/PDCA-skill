# docs/RULE.md

이 프로젝트는 pdca-skill v1 체계를 따른다. 절차 본문은 스킬(`pdca-plan` `pdca-design` `pdca-do` `pdca-close` `cycle-propose` `backlog-sync`)이 정본이고, 이 파일은 **이 프로젝트만의 예외와 훅**만 담는다.

## 검증 수단

Plan §5의 SC에 붙일 수 있는 수단. 여기 없는 수단을 SC에 쓰지 않는다.

| 수단 | 방법 | 비고 |
|---|---|---|
| 자동 테스트 | `{명령}` | {어느 범위를 덮나} |
| 사용자 육안 | {무엇을 어디서 보나} | 사용자가 확인한 날짜를 do §3에 남긴다 |
| 로그·수치 | {무엇을 어떻게 측정하나} | |

## 브랜치

- 태그는 `{develop}` 계열에만 붙인다. `git describe`는 이 브랜치에서 실행한다.

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
