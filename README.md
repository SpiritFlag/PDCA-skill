# pdca-skill

Claude Code로 개인 프로젝트를 PDCA 사이클로 굴리기 위한 스킬 묶음. 한국어 전용.

> 설계 근거는 `docs/redesign.md`에 있다. 이 문서는 **쓰는 법**만 다룬다.
> 상태: 설계 확정, 스킬 · CLI 구현 중. 아직 구현되지 않은 명령은 ⏳로 표시했다.

---

## 1. 이게 뭔가

한 사이클은 다섯 단계이고, 단계마다 **새 Claude Code 세션**을 연다. 세션은 자기 단계 문서만 쓰고, 이전 단계 문서는 읽기만 한다.

```
propose  →  plan  →  design  →  do (세션 N개)  →  close
 제안        범위      설계       구현+검증          감사·보고·릴리즈·태그·백로그
```

문서는 사이클당 여섯 개가 레포에 남는다. `plan` `design` `do` `analysis` `report` `release`.

서버(PDCA-workspace)는 백로그와 릴리즈를 맡는다. 문서의 정본은 git이고, 서버는 열람용이다.

### 설계가 필요 없는 수정은 패치

| | 사이클 | 패치 |
|---|---|---|
| 언제 | 설계가 필요한 일 | 해보니 안 맞아서 바로 고칠 것 |
| 문서 | 6종 | `release` 하나 |
| 버전 | minor 이상 | patch |
| 절차 | propose → plan → design → do → close | 고친다 → close |

## 2. 설치

### 2.1 스킬

`skills/` 아래 폴더 여섯 개를 Claude Code 스킬로 등록한다. 개인 스킬(`~/.claude/skills/`)이나 claude.ai 커스텀 스킬 어느 쪽이든 폴더 통째로.

| 스킬 | 하는 일 |
|---|---|
| `cycle-propose` | 백로그를 묶어 다음 사이클 후보 2~3안과 버전을 제안 |
| `pdca-plan` | plan 문서 작성, 웹 검수 반영, 확정 |
| `pdca-design` | design 문서 작성, 웹 검수 반영, 확정 |
| `pdca-do` | 배치 단위 구현 + 검증, do 문서 누적 |
| `pdca-close` | 감사 → report → release → 인덱스 → 머지 · 태그 → 업로드 → 백로그 동기화 |
| `backlog-sync` | 닫힌 사이클 기준으로 백로그 갱신. close가 마지막에 부르지만 단독 실행도 됨 |

### 2.2 pdcaw CLI ⏳

스킬이 서버를 만지는 유일한 통로. MCP는 쓰지 않는다(회사 계정처럼 MCP를 못 붙이는 환경에서도 돌게 하기 위해).

```bash
npx pdcaw --help
```

레포 루트에 두 파일이 필요하다.

```
.pdcarc.json     { "projectId": "<uuid>", "baseUrl": "https://..." }   커밋함
.env.local       PDCAW_PAT=pdcaw_...                                     커밋 안 함
```

PAT는 서버 `/settings/tokens`에서 발급한다. 서버(Neon 브랜치)별로 다르다.

### 2.3 프로젝트 준비

1. `templates/RULE.template.md`를 `docs/RULE.md`로 복사해 채운다. 검증 수단, 브랜치 이름, CI 확인 명령, 종료 훅, 릴리즈노트 톤. 이 파일은 **프로젝트만의 예외**만 담는다. 절차는 스킬이 안다.
2. `docs/PDCA/_INDEX.md`를 만든다. 아래 형식.
3. 브랜치 `develop`을 만들고 `main`을 보호한다.

```markdown
# PDCA Index

## v1 해돋이

| 버전 | 날짜 | 사이클 | 종류 | 한 줄 요약 | 문서 |
|---|---|---|---|---|---|

## v0

| 버전 | 날짜 | 사이클 | 종류 | 한 줄 요약 | 문서 |
|---|---|---|---|---|---|
```

## 3. 사이클 한 바퀴

아래는 `enhance-lyric-sync`라는 사이클을 `v1.2.0`으로 돌리는 예. 각 단계에서 **형이 치는 것**과 **스킬이 하는 것**, **형이 기다리는 것**을 갈랐다.

### 3.1 propose

```
새 세션 (Opus)
> /cycle-propose
```

스킬이 `pdcaw backlog list`로 열린 항목을 읽고, 직전 report를 읽고, 묶음 후보 2~3안을 낸다. 안마다 사이클명 · 포함 항목 · 근거 · 불확실 지점 · 크기 · **버전**이 붙는다.

형은 안 하나를 고른다. 이 시점에 버전과 사이클명이 확정된다. 백로그를 바꾸거나 문서를 쓰지 않는다.

백로그 없이 시작하려면 이 단계를 건너뛰고 plan에 직접 말한다.

### 3.2 plan

```
새 세션 (Fable)
> /pdca-plan v1.2.0 enhance-lyric-sync
   또는
> /pdca-plan v1.2.0 enhance-lyric-sync "가사 싱크가 0.3초쯤 밀린다, 원인 잡고 고치자"
```

스킬이 하는 것:
- `git switch -c v1.2.0-enhance-lyric-sync develop`
- `docs/PDCA/v1/v1.2.0-enhance-lyric-sync/` 폴더와 `v1.2.0-enhance-lyric-sync.plan.md` 골격
- 코드를 읽어 실측(F-n), 범위 고정, 요구사항(FR-n), 성공 기준(SC-n, 검증 수단 포함), 리스크(R-n)
- 초안을 쓰고 **경로만 보고하고 멈춘다**

형이 하는 것:
- plan.md를 웹 클로드에 옮겨 문답한다(다른 대화 맥락을 아는 쪽이 검수한다).
- 의견을 CC에 붙여넣는다. 스킬이 반영하고 Version History에 행을 남긴다. 반복.
- "확정"이라고 말한다. 헤더 상태가 `확정`으로 바뀌고 세션이 끝난다.

plan이 확정되기 전에는 design 세션을 열지 않는다. 열어도 스킬이 헤더를 보고 멈춘다.

### 3.3 design

```
같은 브랜치, 새 세션 (Opus. adopt · explore 사이클이면 Fable)
> /pdca-design
```

design · do · close는 현재 브랜치 `{버전}-{사이클명}`에서 사이클을 읽는다. 인자가 없다. 브랜치에 없을 때만 `/pdca-design v1.2.0`처럼 준다.

스킬이 하는 것:
- 확정 plan을 읽고 **결정 지점**을 나열한다(D-n).
- 갈림길인 결정에만 선택지를 펼친다. 개수는 실재하는 만큼. 각각의 대가와 추천 하나. 동률이면 동률.
- 변경 파일 목록, 인터페이스(시그니처 · 스키마)를 코드 수준으로.
- 구현 배치(B-n). 배치마다 "무엇을 확인하면 끝인가"가 붙는다. 배치는 세션 하나 분량의 **모듈**(이름 있음)로 묶인다.
- 경로만 보고하고 멈춘다.

형이 하는 것: plan과 같은 웹 검수 루프. "확정"으로 끝.

design은 **맥락 없는 Sonnet이 질문 없이 끝까지 구현할 수 있게** 쓰는 문서다. refine 사이클이면 한 페이지가 정상이다.

### 3.4 do

```
같은 브랜치, 새 세션 (Sonnet)
> /pdca-do                      다음 미착수 모듈
> /pdca-do --scope audio        모듈 하나
> /pdca-do --scope B3,B4        배치 지정
```

스킬이 하는 것:
- design과 do.md(있으면)를 읽고 맡을 모듈 · 배치를 정한다.
- 배치 하나를 **구현하고 검증까지** 한 뒤 do.md에 기록한다. 검증 기록에 없는 것은 검증 안 된 것이다.
- 다음 배치로. 세션이 길어지면 진행 로그를 남기고 멈춘다.

형이 하는 것:
- 검증 수단이 "사용자 육안"인 배치는 스킬이 멈추고 확인을 요청한다. 확인해 주면 날짜와 함께 기록된다.
- 스킬이 `v1.2.0-enhance-lyric-sync.q1.md` 같은 **질문 파일**을 만들고 멈추면 답한다. 설계를 바꿔야 하거나 design이 안 정한 갈림길을 만난 것이다. 답을 주면 design(설계 변경) 또는 do(세부 결정)에 반영하고 파일을 지운다.
- 코드는 평소처럼 커밋한다. 문서도 이 브랜치에서는 자유롭게 커밋한다.

세션을 여러 번 열어도 do.md는 하나다. 모든 배치가 `검증 완료`가 되면 do가 끝난다.

do 세션은 design을 고치지 않는다. 고칠 일이 생기면 질문 파일이다. 이게 "검증을 하는 건지 검증 기준을 고치는 건지" 헷갈리던 문제를 막는 장치다.

### 3.5 close

```
같은 브랜치, 새 세션 (Opus)
> /pdca-close
```

게이트: do.md 전 배치 `검증 완료`, 질문 파일 없음, 브랜치 CI 초록. 하나라도 아니면 멈춘다.

스킬이 하는 것:
1. `analysis.md`: do.md의 주장을 **감사**한다. SC마다 `실증 / 부분 실증 / 미실증 / 범위 변경`. 결함(I-n)은 즉시 수정 · 재검증하거나 이월. 합산 수치 없음.
2. `report.md`: 결과, 완료 항목, **미완료 4분류**(이월 / 보류 / 폐기 후보 / 범위 밖 발견), 결정과 결과, 회고 Keep · Problem · Try, 프로세스 개선, 다음 후보.
3. `release.md`: report와 git diff를 대조해 게임 공지 톤의 릴리즈노트. 확인 안 된 기능은 안 쓴다.
4. `_INDEX.md` 행 추가.
5. RULE.md 종료 훅(README 갱신, 버전 파일 수정 등).
6. 사이클 브랜치에 커밋 · 푸시.
7. develop에 `--no-ff` 머지 → 머지 커밋에 태그 → 푸시. **푸시 전에 묻는다.**
8. `pdcaw upload --version v1.2.0`: 서버에 릴리즈 생성 + 문서 업로드 + release.md를 릴리즈노트로.
9. develop CI 초록이면 `main`을 태그로 ff. **푸시 전에 묻는다.** 빨가면 main을 두고 패치를 안내한다.
10. `backlog-sync`: report §3을 읽어 백로그를 done / resolved / 신규 / 갱신. 폐기와 todo 복원은 형만 한다. 판단이 갈리는 건 질문으로 남긴다.

형이 하는 것: 7과 9에서 확인. 10의 질문에 답.

### 3.6 패치

```
git switch -c v1.2.1-fix-lyric-offset develop
(고친다, 커밋한다, 푸시한다, CI 본다)
> /pdca-close
```

plan.md가 없으니 close가 패치 모드로 돈다. release → 인덱스 → 훅 → 머지 · 태그 → 업로드 → main ff → backlog-sync. 백로그 항목에서 나온 패치면 done 처리, 아니면 아무것도 안 만든다.

## 4. 브랜치

```
develop ──●────────────●(merge, tag v1.2.0)────●(merge, tag v1.2.1)──
           \          /                       /
            v1.2.0-enhance-lyric-sync   v1.2.1-fix-lyric-offset
main    ─────────────────────────────●(ff → v1.2.0)──────────●(ff → v1.2.1)
```

- `main`: 보호. 태그로 `--ff-only`만. 체리픽 안 한다.
- `develop`: 사이클 · 패치 브랜치의 머지 커밋만. 태그는 여기 머지 커밋에.
- `{버전}-{사이클명}`: 작업. 폴더명과 같다. 문서 · 코드 자유롭게 커밋.
- CI는 사이클 브랜치에서 먼저 돈다. 초록이 close의 게이트다. 머지 뒤 develop이 빨가면 태그를 되돌리지 않고 패치로 잇는다. main은 초록일 때만 전진한다.

## 5. 문서 규칙 요약

- 위치 `docs/PDCA/v{N}/{버전}-{사이클명}/{버전}-{사이클명}.{stage}.md`. 폴더명 == 파일 어간. 안 옮긴다.
- 헤더는 표 하나. 값은 교체, 이력은 Version History에.
- 요약 절 없음. 링크 없음(이름 · 경로 인라인 코드 · id로 참조). 합산 수치 없음.
- 다른 문서 참조는 `Plan §3` `Design D-4`처럼 문서명을 붙인다. 다른 사이클은 버전으로.
- 질문은 `{어간}.qN.md`. 답 반영 후 삭제.
- release만 헤더 · Version History가 없다.

## 6. CLI 요약 ⏳

```
pdcaw upload   [--version vX.Y.Z] [--all] [--path <p>]...
pdcaw project  list
pdcaw cycle    list
pdcaw backlog  list [--status s,...] [--stale <days>] [--q <text>]
pdcaw backlog  get <id>
pdcaw backlog  create --title <t> --priority <p> --opened-on <d> [--detail-file <f>]
pdcaw backlog  update <id> [--status s] [--closed-on d] [--append-detail <t|@file>] ...
pdcaw doc      collect --stage <s> [--major vN] --out <dir|file>
```

모두 `--json`을 받는다. 스킬은 `--json`만 쓴다. `doc collect`는 로컬만 본다(리포트만 싹 긁어 웹 클로드에 올릴 때).

## 7. 자주 묻는 것

**plan과 design이 뭐가 다른가.** 독자가 다르다. plan은 형이 읽고 "이 범위에 시간을 쓴다"를 정한다. design은 맥락 없는 Sonnet이 읽고 질문 없이 구현한다. design이 plan을 다시 쓰고 있으면 잘못 쓴 것이다.

**Match Rate는 어디 갔나.** 없앴다. SC마다 항목별 판정만 있다. 합산은 숫자를 만들려고 산식을 바꾸게 만든다.

**사이클명이 겹쳐도 되나.** 된다. 버전이 열쇠다. `refine-backlog`가 v0.13.2와 v1.4.0에 둘 다 있어도 폴더 · 브랜치 · 서버 어디서도 안 부딪힌다.

**백로그 없이 시작해도 되나.** 된다. plan에 문장을 직접 주면 §2.1에 "직접 지시"로 남는다. 설계가 필요 없으면 패치 트랙.

**웹 검수는 왜 자동화 안 하나.** CC는 다른 대화의 맥락을 모른다. 웹 클로드가 그 맥락을 알고 검수하는 게 목적이라 사람이 옮기는 것이 곧 기능이다.

**회사 계정은 MCP가 안 되는데.** 스킬은 MCP를 안 쓴다. `.env.local`에 PAT만 있으면 `pdcaw`로 다 된다. MCP는 claude.ai 웹 대화에서 서버를 볼 때만 쓴다.

**리포트만 한꺼번에 보고 싶다.** `pdcaw doc collect --stage report --out reports.md`. 셸이면 `find docs/PDCA -name '*.report.md'`로도 충분하다. 파일 어간에 버전이 있어 정렬된다.

## 8. 레포 구조

```
skills/
  cycle-propose/SKILL.md
  pdca-plan/SKILL.md, plan.template.md
  pdca-design/SKILL.md, design.template.md
  pdca-do/SKILL.md, do.template.md
  pdca-close/SKILL.md, analysis.template.md, report.template.md, release.template.md
  backlog-sync/SKILL.md
templates/RULE.template.md
docs/redesign.md         설계 근거
README.md                이 문서
```

관련 레포: `pdcaw-cli`(CLI), `PDCA-workspace`(서버 · 웹 UI · MCP).
