# PDCA 체계 대개편 설계

> 이 문서는 PDCA 체계(스킬 · 문서 규약 · pdcaw CLI · PDCA-workspace)를 통째로 다시 세우는 큰그림이다.
> bkit을 계승하지 않는다. 이 체계는 `pdca-skill v1`이다.
> 이 개편 자체는 PDCA 사이클로 돌리지 않는다. 새 체계의 첫 실전 사이클(sing-diary)이 이 설계의 검증이다.
>
> 상태: 설계 중. 결정된 항목은 본문에, 미결은 §10에 둔다.

---

## 0. 왜 다시 세우나

bkit PDCA 스킬을 커스텀해 쓰다가 손볼 곳이 계속 늘었다. 원인은 셋이다.

1. **전제가 다르다.** bkit은 Next.js 웹앱(Zod, route.ts, Playwright, bkend.ai, 4계층 Clean Architecture)을 전제한다. 우리 프로젝트는 Unity, CLI, Hono 서버 등 제각각이다. 템플릿의 절반이 매번 폐기 대상이다.
2. **형식이 내용을 밀어낸다.** 고정 3안 비교(A 최소/B 클린/C 절충)는 C가 이기도록 짜인 의식이다. Match Rate는 11사이클 중 4번 산식을 갈아치웠다. 문서 상단 요약은 개정마다 늘어난다. Design은 Plan을 재진술하느라 두꺼워지고, 그 위에 진행 상황까지 덧붙어 20버전을 넘겼다.
3. **경계가 잘못 그어져 있다.** 검증이 Do가 아니라 analysis에서 처음 일어나서 "코드는 완결됐는데 실행 확인이 없다"가 경고 후에도 재현됐다(9차). Act(회고 반영 · 백로그 갱신 · 릴리즈노트 · 규칙 개정)는 report, backlog-sync, 웹 클로드 수작업, 사람 손으로 흩어져 있다.

## 1. 원칙

| # | 원칙 | 뜻 |
|---|---|---|
| P1 | **정본은 git.** | 문서는 레포에 있다. 서버(PDCA-workspace)는 백로그 · 릴리즈 · 열람을 맡는다. CC 세션은 문서를 디스크에서 읽고, 서버에서 문서를 내려받지 않는다. |
| P2 | **문서는 독자로 정의한다.** | 각 문서는 "누가 읽고 무엇을 결정하는가"로 정의한다. 독자가 같은 내용은 한 문서에만 있다. 재진술 금지. |
| P3 | **검증은 Do가 한다.** | 구현 배치마다 검증하고 do 문서에 기록한다. analysis는 그 기록을 감사한다. |
| P4 | **세션 하나 = 스킬 하나.** | 세션은 자기 단계 문서만 쓴다. 이전 단계 문서는 읽기 전용이다. Do만 여러 세션에 걸칠 수 있다. |
| P5 | **의식적 형식 금지.** | 고정 개수 선택지, 합산 수치(Match Rate 류), 개정마다 자라는 요약, 프로젝트 상수의 사이클 문서 재기재를 두지 않는다. |
| P6 | **CC는 CLI, 웹은 MCP.** | CC 스킬은 서버를 `pdcaw` CLI로만 만진다. MCP 툴은 claude.ai 웹 대화 전용이다. 회사 계정처럼 MCP를 못 붙이는 환경에서도 스킬이 그대로 돈다. |
| P7 | **판단은 사람.** | 미루기 · 보류 · 폐기 · 확정은 사용자가 정한다. 스킬은 후보와 근거를 내고 멈춘다. |
| P8 | **웹 검수는 사람 손.** | Plan · Design 검수는 사용자가 웹 클로드에 문서를 옮겨 문답하고, 의견을 CC에 붙여넣는다. 자동화하지 않는다. |

## 2. 라이프사이클

```
[propose]  백로그 → 사이클 후보 → 사용자 선택 (버전·사이클명 확정)
[plan]     실측 → 범위·SC 고정 → plan.md → 웹 검수 → 확정
[design]   결정 지점 → 선택지·추천 → design.md → 웹 검수 → 확정
[do]       배치별 구현 + 검증 → do.md 누적 (세션 N개)
[close]    감사(analysis) → report → release → 인덱스 → 업로드·태그 → 백로그 동기화
```

| 단계 | 모델 | 스킬 | 읽는 것 | 쓰는 것 | 끝나는 조건 |
|---|---|---|---|---|---|
| propose | Opus | `cycle-propose` | 백로그(CLI), 직전 report | 없음(제안만) | 사용자가 안 하나를 고르고 버전 · 사이클명 확정 |
| plan | Fable | `pdca-plan` | 코드, 백로그 항목 detail, 직전 report | `plan.md` | 헤더 상태 `확정` |
| design | Fable(`adopt` `explore`) / Opus(그 외) | `pdca-design` | 확정 plan, 코드 | `design.md` | 헤더 상태 `확정` |
| do | Sonnet | `pdca-do` | 확정 design, do.md(이전 세션) | 코드, `do.md`, 필요 시 `qN.md` | do.md 배치 전건 검증 완료 |
| close | Opus | `pdca-close` → `backlog-sync` | plan · design · do, git diff | `analysis.md` `report.md` `release.md` `_INDEX.md` | 태그 푸시, 백로그 갱신 보고 |

**모델 분배의 근거**: 이득의 대부분은 모델 차이가 아니라 세션 분리(단계마다 확정 문서만 들고 새 컨텍스트로 시작)에서 온다. 그 위에서 판단하는 단계(plan · design · close)는 Opus 이상, 실행하는 단계(do)는 Sonnet이 기준선이다. Fable은 plan과 새 영역 design에만 쓴다. Design의 빠진 결정 지점 하나가 do의 질문 파일 하나이고 그게 사용자 왕복 하나라서, 새 영역일수록 design에 더 강한 모델을 쓴다. close는 감사 · 회고 · 릴리즈노트 · 백로그 판정이 겹치는 판단 단계라 Sonnet에서 Opus로 올린다. 웹 검수 모델은 작성 모델보다 약하지 않아야 한다.

**게이트**
- plan이 `확정`이 아니면 design 세션을 열지 않는다. design이 `확정`이 아니면 do 세션을 열지 않는다. 스킬은 시작 시 선행 문서 헤더를 읽어 이를 확인하고, 아니면 이유를 말하고 멈춘다.
- do 세션에서 설계를 바꿔야 하면 질문 파일을 만들고 멈춘다(§3.3). 답을 받으면 design.md를 개정하고(Version History 행) 계속한다. do 세션이 design.md를 고치는 유일한 경우다.
- close는 do.md의 모든 배치가 `검증 완료`일 때만 시작한다.

### 2.1 두 트랙

| | 사이클 | 패치 |
|---|---|---|
| 언제 | 설계가 필요한 일 | 해보니 안 맞아서 바로 고칠 것 |
| 문서 | 6종 | `release` 하나 |
| 버전 | minor 이상 | patch |
| 종류 | `explore` `adopt` `enhance` `refine` | `fix`. 설계가 필요한 fix는 사이클로 올릴 수 있다 |
| 시작 | `cycle-propose` → `pdca-plan`, 또는 `pdca-plan`에 직접 지시 | 그냥 고친다 |
| 끝 | `pdca-close` | `pdca-close`. plan.md가 없으면 패치 모드: release → 인덱스 행 → 훅 → 커밋 → 업로드 → 태그 → backlog-sync |

백로그는 입구 중 하나일 뿐 필수 관문이 아니다. 사이클은 plan §2.1에 "직접 지시: {문장}"으로 백로그 없이 시작할 수 있고, 패치는 백로그 항목에서 나왔으면 close가 done 처리하고 아니면 아무것도 만들지 않는다.

### 2.2 브랜치 · CI

| 브랜치 | 역할 | 전진 방식 |
|---|---|---|
| `main` | 검증된 릴리즈 포인터. 보호. 프로덕션 배포 트리거 | 태그로 `--ff-only`만. 체리픽 금지 |
| `develop` | 통합 + CI. 개발 환경 배포 | 사이클 · 패치 브랜치의 `--no-ff` 머지 커밋만 받는다. 태그는 이 머지 커밋에 찍는다 |
| `{버전}-{사이클명}` | 사이클 · 패치 작업. 폴더명과 같다 | `pdca-plan`(또는 패치 시작 시 사람)이 develop에서 분기. 문서 · 코드 모두 자유롭게 커밋 |

체리픽을 쓰지 않는 이유: SHA가 갈라져 main과 develop이 영영 다른 역사가 되고, 태그가 develop 커밋을 가리켜 main의 `git describe`가 어긋난다. develop은 사이클이 닫힐 때만 머지를 받으므로 develop == 최신 태그가 항상 성립하고, main은 그중 CI를 통과한 태그까지만 따라온다. "일부만 골라 main에 넣을" 상황은 사이클 단위 릴리즈에서 생기지 않는다.

**CI는 태그 앞에서 돈다.**
1. do 마지막 배치 검증 완료 → 사이클 브랜치 푸시 → CI가 사이클 브랜치에서 돈다.
2. CI 초록이 close의 게이트다. 빨간 채로 close를 시작하지 않는다.
3. close가 문서를 사이클 브랜치에 커밋한다. 문서 커밋에도 CI가 한 번 더 돈다.
4. develop에 `--no-ff` 머지 → 머지 커밋에 태그 → 태그 푸시 → pdcaw 업로드.
5. develop CI 초록 확인 → main을 태그로 ff.

4 뒤에 develop CI가 깨지면(머지 결과에서만 나는 실패) 태그를 되돌리지 않고 패치 트랙으로 바로 이어 간다. main은 5를 통과하지 못했으니 직전 태그에 서 있다. develop이 잠깐 빨간 건 허용하고 main은 빨갛지 않게 하는 것이 3층의 목적이다.

close가 CI 상태를 직접 읽으려면 `gh` 같은 도구가 필요한데 환경마다 다르다. RULE.md에 CI 확인 명령을 두고, 없으면 close가 사용자에게 CI 초록을 확인받고 진행한다.

**릴리즈 브랜치 전진의 변형.** main이 GitHub 룰셋으로 직푸시 금지면 ff가 불가능하다. 그 프로젝트는 RULE.md에 `PR merge commit` 방식을 적는다: develop → main PR을 Merge commit으로 병합하고 develop에 main을 백머지한다. Squash · Rebase는 태그 SHA를 main 밖에 남기므로 금지. 태그는 develop 머지 커밋에 찍고, `git branch -a --contains {버전}`에 main · develop이 둘 다 나오면 정렬된 것이다(sing-diary가 이 변형).

"PDCA 문서는 사이클 도중 커밋하지 않는다" 규칙은 폐기한다. 사이클 브랜치가 있으니 develop 역사는 머지 커밋 하나로 깨끗하고, 휘발성 환경에서 문서를 커밋 없이 들고 있는 위험이 더 크다.

사이클 두 개를 동시에 돌리면 `_INDEX.md` 맨 위 행에서 충돌한다. 순차가 기본이고 병행은 수동 해결로 감수한다.

**웹 검수 루프(plan · design 공통)**
1. 스킬이 초안을 쓰고 문서 경로만 보고하고 멈춘다.
2. 사용자가 웹 클로드에 문서를 옮겨 문답하고, 의견을 CC에 붙여넣는다.
3. 스킬이 의견을 반영하고 Version History에 행을 추가한다. 반복.
4. 사용자가 "확정"이라 하면 헤더 상태를 `확정`으로 바꾸고 확정일을 적는다. 여기서 세션이 끝난다.

## 3. 문서

### 3.1 위치 · 이름 · 언어

```
docs/PDCA/
  v0/                                        메이저 폴더. 숫자만
    v0.13.2-refine-backlog-hygiene/          사이클 폴더 = {버전}-{사이클명}
      v0.13.2-refine-backlog-hygiene.plan.md 파일 어간 = 폴더명
      v0.13.2-refine-backlog-hygiene.report.md
  v1/                                        메이저 = 시대. 코드네임은 인덱스 절 제목에만
    v1.2.0-enhance-lyric-sync/
    v1.2.1-fix-lyric-offset/                 패치도 같은 층. 버전이 정렬을 보장
      v1.2.1-fix-lyric-offset.release.md     패치 트랙은 release 하나만
  _INDEX.md
```

- 버전을 시작에 정하므로(§5.1) 폴더에 버전을 쓸 수 있다. `YYYY-MM` 폴더는 버전을 close에서야 알던 시절의 대체물이었다.
- 폴더명과 파일 어간이 같아야 한다. 폴더에서 꺼내 놓아도 자기식별되고 버전순 정렬된다. 긴 것은 감수한다.
- 메이저 폴더는 `v{N}`뿐이다. 코드네임(예: v1 해돋이)은 `_INDEX.md`의 메이저 절 제목에 둔다. 폴더에 넣으면 개명 부담이 생긴다.
- **사이클의 열쇠는 버전이다.** 폴더 · 파일 · 브랜치 · 서버 모두 버전으로 식별한다. 사이클명은 사람이 읽는 꼬리표라 겹쳐도 된다.
- 사이클이 끝나도 옮기지 않는다.
- stage 6종, 이 순서: `plan` `design` `do` `analysis` `report` `release`. 패치 트랙(§2.1)은 `release`만 쓴다.
- 한국어, `.md`만. 사용자는 문서 안에서 "사용자"로 부른다.
- 인덱스는 `docs/PDCA/_INDEX.md` 하나.
- PDCA 밖 문서는 `docs/{주제}/`에 둔다. `docs/` 아래에는 서버에 올라가도 되는 것만 둔다.

### 3.2 공통 형식

**헤더**는 표 하나로 고정한다. 값은 **교체**하되 덧붙이지 않는다. 이력은 Version History에만 남긴다.

```
# {cycle} {단계명}

| 프로젝트 | 사이클 | 버전 | 상태 | 작성 | 갱신 |
|---|---|---|---|---|---|
| sing-diary | enhance-lyric-sync | v0.4.0 | 확정 | 2026-09-04 | 2026-09-05 |
```

상태 값: plan · design은 `초안 → 확정`, do는 `진행 중 → 완료`, analysis · report는 `완료`만. 갱신은 마지막으로 고친 날이라 확정일 · 완료일을 겸한다.

**release는 예외**로 헤더 표와 Version History를 두지 않는다. 독자가 서비스 사용자라 제목과 첫 줄(버전 · 날짜)이 진실이다.

**요약 절은 없다.** 문서 상단에 Executive Summary, Context Anchor 류를 두지 않는다. plan §1 목적 한 단락과 report §1 결과 한 단락이 그 역할을 하며, 둘은 "개정 시 늘리지 않고 다시 쓴다".

**Version History**는 문서 맨 아래 표. 개정 시 행 추가. `| 버전 | 날짜 | 변경 | 계기 |`. 계기 열에 "웹 검수 의견", "질문 q2 답 반영", "감사 결함 수정"처럼 왜 바뀌었는지를 적는다.

**작성 방식**: 골격(헤딩 전부)을 먼저 쓰고 절 하나씩 채운다. 한 번에 통째로 쓰지 않는다.

**참조 표기**: 다른 문서의 절을 가리킬 때 문서명을 붙인다. `Plan §3`, `Design D-4`, `Do B2`. 번호만 쓰지 않는다(7차 M-5 재발 방지). 다른 사이클을 가리킬 때는 버전을 쓴다(`v0.13.2 report §5.3`). 사이클명은 겹칠 수 있어 열쇠가 못 된다.

**링크 금지**: PDCA 문서 안에 마크다운 링크(`[..](..)`)를 쓰지 않는다. 파일이 옮겨지거나 이름이 바뀔 때마다 링크 검사 CI가 깨진다. 다른 문서는 `Plan §3`처럼 이름으로, 파일은 `` `src/foo.ts` ``처럼 경로를 인라인 코드로, 백로그 항목은 id로 가리킨다. 인덱스의 문서 열도 stage 이름만 나열한다. 외부 URL이 꼭 필요하면(CVE, 공식 문서) 그때만 링크를 허용하되 사이클 문서 안 상호 링크는 예외 없이 금지한다.

**ID 체계**
| ID | 뜻 | 사는 곳 |
|---|---|---|
| `FR-n` | 요구사항 | plan |
| `SC-n` | 성공 기준(실증 가능한 문장) | plan. 이후 문서는 참조만 |
| `F-n` | 실측 사실 | plan |
| `R-n` | 리스크 | plan |
| `D-n` | 설계 결정 | design |
| `B-n` | 구현 배치 | design이 정의, do가 소비 |
| `q-n` | 질문 파일 | do (파일 `{cycle}.qN.md`) |
| `I-n` | 감사 발견 결함 | analysis |

### 3.3 질문 파일

- do 세션에서 설계 변경이 필요하거나 design이 답하지 않은 갈림길이 나오면 `{cycle}.qN.md`를 만들고 멈춘다.
- 내용: 무엇이 막혔나, 선택지와 대가, 스킬의 추천. 사용자가 답을 파일에 적거나 대화로 준다.
- 답을 design.md(설계 변경) 또는 do.md §4 구현 결정(설계 범위 내 세부)에 반영한 뒤 파일을 지운다. do.md §5에 번호 · 결론 한 줄을 남긴다.
- close는 시작 시 `*.q*.md`가 남아 있으면 멈춘다.

### 3.4 문서별 정의

각 문서는 독자 · 답하는 질문 · 담는 것 · 담지 않는 것 · 개정 규칙으로 정의한다.

#### plan

- **독자**: 사용자. 범위 승인.
- **질문**: 왜 지금 이걸 하나, 어디까지 하나, 끝났다는 걸 어떻게 아나.
- **담는 것**
  1. 목적: 한 단락. 백로그 항목 문장을 인용한다. 선행 사이클과의 관계.
  2. 범위: 포함(백로그 id로 세어 N건 고정. 백로그 없이 시작한 사이클은 "직접 지시: {문장}") / 제외(인접해서 손대고 싶어질 것을 미리 열거) / 발견 시 처리(고치지 말고 백로그로).
  3. 실측: F-n. 추정 금지. 파일 · 함수 · 스키마를 직접 읽어 확정한 사실. 성패를 가르는 것은 ★.
  4. 요구사항: FR-n.
  5. 성공 기준: SC-n. 각 기준에 검증 수단(자동 테스트 / 사용자 육안 / 로그 · 수치)을 열로 붙인다. 사람 손이 필요한 비율이 높으면 그 사실을 적는다.
  6. 리스크: R-n과 대응.
- **담지 않는 것**: 아키텍처 선택표, 프레임워크 비교, 컨벤션, 환경변수 표, 프로젝트 레벨. 설계는 design에.
- **개정**: 웹 검수 반영. 확정 후에는 do 세션의 질문 답으로 범위가 바뀌는 경우만.

#### design

- **독자**: Do 세션(Sonnet, 맥락 없음). 질문 파일 없이 끝까지 구현할 수 있어야 한다.
- **질문**: 어떻게 만드나, 무엇을 결정했고 왜인가, 무엇이 바뀌고 어떤 순서로 하나, 각 배치는 무엇으로 검증하나.
- **담는 것**
  1. 결정: D-n. 결정 지점을 먼저 나열한다. **갈림길인 것만** 선택지를 펼친다. 선택지는 실재하는 만큼(둘이든 넷이든), 각각의 대가, 추천 하나와 이유. 동률이면 동률이라 쓰고 사용자가 고른다. 갈림길이 아닌 결정은 "이렇게 한다, 이유는 이것" 한 줄.
  2. 변경 지점: 신설 · 수정 · 삭제 파일과 각각의 책임. 인터페이스(시그니처 · 스키마 · 이벤트)는 코드 수준으로.
  3. 구현 배치: B-n. 3~5파일 단위. 배치마다 **검증 방법**(무엇을 어떻게 확인하면 이 배치가 끝난 것인가)을 붙인다. 배치는 **모듈**(케밥 이름, 세션 하나 분량)로 묶는다. `pdca-do --scope <모듈>`이 이 이름을 쓴다.
  4. 열린 질문: design이 답하지 못한 것. 비어 있어야 확정할 수 있다.
- **담지 않는 것**: plan 재진술(목적 · 배경 · 원칙), 프로젝트 상수(계층 규칙 · 네이밍 · import 순서), 진행 상황.
- **개정**: 웹 검수 반영. 확정 후에는 질문 파일 답을 반영할 때만. **do 세션이 진행 기록을 여기에 쓰지 않는다.**
- refine 사이클이면 한 페이지가 정상이다. 짧다고 채우지 않는다.

#### do

- **독자**: 다음 Do 세션(인계), close 세션(감사 근거).
- **질문**: 어디까지 했나, 설계가 정하지 않은 것을 어떻게 정했나, 무엇을 무엇으로 검증했나.
- **담는 것**
  1. 배치 현황: B-n별 `미착수 / 진행 중 / 구현 완료 / 검증 완료`. 담당 세션 번호.
  2. 구현 결정: design이 정하지 않은 세부를 어떻게 했는지와 이유. design을 바꾸는 결정은 여기가 아니라 질문 파일이다.
  3. 검증 기록: 배치마다 무엇을 실행 · 확인했고 결과가 무엇인지. 자동 테스트는 명령과 결과, 육안 확인은 사용자가 확인한 날짜와 내용. **여기 없는 것은 검증되지 않은 것**이다.
  4. 질문 이력: q-n, 결론 한 줄, 반영 위치(design D-n 개정 / do 구현 결정).
  5. 진행 로그: 세션마다 시작 · 종료 시 한 단락. 막힌 것, 다음 세션이 먼저 볼 것.
- **담지 않는 것**: 설계 재진술, Context Anchor, 체크리스트 상수.
- **개정**: 세션마다 누적. 하나의 do.md가 사이클 전체를 관통한다.
- **규율(Depth-First)**: 배치 하나를 끝까지(구현 + 검증) 하고 다음으로. 넓게 뼈대만 치지 않는다. 플레이스홀더 · TODO · 빈 핸들러를 남기고 다음 배치로 가지 않는다.

#### analysis

- **독자**: 사용자, report.
- **질문**: Do가 남긴 주장이 사실인가. SC는 실제로 충족됐나. 설계 결정은 지켜졌나.
- **담는 것**
  1. SC 판정표: SC별로 `do.md 근거 / 독립 재확인 방법 / 판정`. 판정은 `실증` `부분 실증(무엇이 빠졌나)` `미실증(사유)` `범위 변경(→ report §3 어디로)` 중 하나. **합산 수치는 두지 않는다.**
  2. 결정 이행: D-n별로 지켰나, 어긋났으면 어디서 어떻게.
  3. 결함: I-n. 발견 → 즉시 수정한 것과 이월한 것을 구분. 수정한 것은 재검증 결과.
  4. 감사 방법: 무엇을 직접 실행했고 무엇은 do.md 기록을 신뢰했는지.
- **담지 않는 것**: 코드 품질 점수, 컨벤션 점수, 아키텍처 점수, 커버리지 표, 성능 표. 필요하면 SC로 plan에 미리 넣는다.
- **개정**: 결함 수정 후 재판정 시.

#### report

- **독자**: 사용자, backlog-sync, 다음 사이클의 plan, 릴리즈노트 생성.
- **질문**: 무엇이 됐고 무엇이 안 됐나. 안 된 것은 어떤 종류로 안 됐나. 무엇을 배웠나.
- **담는 것**
  1. 결과: 한 단락. SC 판정표 요약 링크(analysis §1). 수치 없음.
  2. 완료 항목: plan §2 범위 대비.
  3. 미완료 항목, **넷으로 갈라서**:
     - 3.1 **이월**: 할 것이다, 이번엔 못 했다. → 백로그 todo(신규 또는 기존 갱신)
     - 3.2 **보류**: 조건이 충족되면 한다. 조건을 적는다. → 백로그 todo + 착수 조건
     - 3.3 **폐기 후보**: 하지 않기로 하는 게 맞아 보인다. 이유. → **사용자가 결정**, 스킬은 찍지 않는다
     - 3.4 **범위 밖 발견**: 사이클 중 발견했으나 이 사이클 일이 아닌 것. → 백로그 신규 todo
  4. 결정과 결과: D-n별 결과. 뒤집힌 결정은 왜.
  5. 회고: Keep / Problem / Try. Try는 실행 가능한 문장으로, 다음 사이클 plan이 이행 여부를 확인한다.
  6. 프로세스 · 도구 개선: 반복 비용이 관측된 것. 스킬 · RULE · CLI · 서버 어느 것을 고칠지.
  7. 다음 사이클 후보.
- **담지 않는 것**: Executive Summary 4관점, Changelog(→ release), 관련 문서 표(폴더가 말해준다), 품질 지표(→ analysis), Match Rate.
- backlog-sync는 §3.1~3.4, §5 Try, §6, §7을 읽는다. 절 번호를 바꾸면 backlog-sync를 같이 고친다.

#### release

- **독자**: 서비스 사용자. 게임 공지 톤.
- **질문**: 이번 버전에서 무엇이 달라졌나.
- **재료**: report, 직전 태그~이번 태그 git diff, do.md 검증 기록, 직전 release(연속성).
- **규칙**: 웹 클로드에서 쓰던 릴리즈노트 생성 규칙을 계승한다. 카테고리(✨기능 🔒보안 🛠️버그수정 ⚡성능 🎨UI/UX) 중 해당 없는 것은 섹션 생략, "-습니다" 체, 내부 식별자 대신 동작과 효익, 확인되지 않은 기능은 쓰지 않음(report 주장을 diff로 대조), 버전 간 중복 공지 금지, 다부작이면 시리즈 내 위치 언급, outro는 규모 반영.
- **버림**: 파일명에서 버전 추출(태그와 헤더가 진실), `/mnt/user-data/outputs` 경로, 제품명 하드코딩(프로젝트명은 `.pdcarc.json`), `.report.ko.md` 분기.
- 서버에서는 이 문서가 그 버전의 릴리즈노트다(§7.4).

### 3.5 인덱스 (`docs/PDCA/_INDEX.md`)

메이저마다 절을 둔다(`## v1 해돋이 (sunrise)`). 최신 메이저가 위. 절 안은 열 6개 고정: `| 버전 | 날짜 | 사이클 | 종류 | 한 줄 요약 | 문서 |`. 최신이 위. 한 줄 요약은 결과만, 60자 내외, 수치 최대 2개, 이월 · 후속 언급 금지. 문서 열은 존재하는 stage를 ` · `로 나열(패치는 `release`만). 링크 없음.

행은 **close에서** 추가한다(사이클 시작 시가 아니라). 진행 중 사이클은 폴더의 존재가 말해준다.

## 4. 검증 모델

- **검증 단위는 배치(B-n)**다. design이 배치마다 검증 방법을 정하고, do가 실행하고 기록하고, analysis가 감사한다.
- **검증 수단 3종**: 자동 테스트(명령 + 결과), 사용자 육안 확인(날짜 + 확인 내용, 사용자가 했다는 사실을 do.md에 남김), 로그 · 수치(실측값). 프로젝트마다 가용한 수단이 다르므로 plan SC에 수단을 미리 붙인다.
- **판정은 항목별**로만 한다. 합산 · 비율 · 점수를 만들지 않는다. 인덱스에도 두지 않는다.
- **"검증하는지 검증 기준을 고치는지"** 혼선 방지: 검증 기준(SC, 배치 검증 방법)은 plan · design에 있고 확정 후 고정이다. do 세션은 기준을 고칠 수 없다. 기준이 틀렸으면 질문 파일이다.

### 4.1 테스트 생애주기

테스트는 검증 수단 "자동 테스트"의 실체다. **이유 없는 테스트는 없다.** 모든 테스트는 SC 하나, 배치 검증 방법 하나, 또는 결함 I-n 하나를 근거로 존재한다. 이 절은 RULE.md 검증 수단에 자동 테스트가 있는 프로젝트에 적용된다.

| 시점 | 무엇을 | 규칙 |
|---|---|---|
| plan | 기존 테스트 파악 | §3 실측에 "이 영역을 덮는 기존 테스트"를 F-n으로 적는다. SC에 자동 테스트를 붙일 때 기존이 덮는지 새로 써야 하는지가 여기서 갈린다 |
| design | 테스트를 변경 지점으로 | §2.1 파일 표에 테스트 파일도 신설 · 수정 · 삭제로 올린다. 배치 검증 방법에 "어느 테스트가 무엇을 단언하는가"까지 쓴다. 동작이 바뀌어 깨질 기존 테스트는 여기서 "수정"으로 미리 지목한다 |
| do 생성 | 배치 안에서 코드와 함께 | 테스트 통과가 배치 `구현 완료`의 조건이다. design에 없는 테스트를 추가하면 do §2 구현 결정에 이유를 적는다 |
| do 수정 | design이 지목한 것만 자유 | design §2.1에 "수정"으로 올라 있는 테스트는 고친다. **그 밖의 실패하는 테스트를 통과시키기 위한 수정은 항상 질문 파일이다.** 테스트가 틀린 게 명백해 보여도 마찬가지다. 그게 "기준을 고치는" 순간이라 사람이 본다 |
| do 삭제 | design이 지목한 것만 | §2.1에 "삭제"로 올라 있을 때만. 기능이 없어지면 그 테스트도 design이 없앤다 |
| do 금지 | skip · quarantine · 조건부 건너뛰기 | 초록을 만들려고 테스트를 끄지 않는다. 불안정한 테스트는 do §2에 기록하고 그대로 두며, close가 I-n 또는 report §3.4로 백로그에 보낸다 |
| close 감사 | 단언 내용 확인 | SC에 붙은 테스트가 **존재하는지가 아니라 그 SC를 실제로 단언하는지** 읽고, 독립 재실행한다. 결함 I-n을 수정할 때는 그 결함을 잡았을 회귀 테스트를 함께 넣는다 |
| 패치 | 회귀 테스트 동반 | 버그 수정 패치는 재현 테스트를 먼저 빨갛게 만들고 수정으로 초록을 만든다. 테스트로 잡을 수 없는 결함(육안 전용)은 release에 그 사실을 쓰지 않고 do 없이 close에서 확인 기록만 남긴다 |
| CI 설정 | 코드로 취급 | 워크플로 · 테스트 러너 설정 변경은 design 변경 지점이거나 패치 트랙이다. 사이클 중 몰래 바꾸지 않는다 |

**CI 실행 범위**: 사이클 브랜치 푸시마다 전체 테스트. 느리거나 외부 자원이 필요한 테스트는 RULE.md에 별도 명령으로 분리하고, 그것이 SC 검증 수단이면 do §3에 수동 실행 결과를 기록한다.

**테스트 위치 · 명령**은 프로젝트 관례라 RULE.md 검증 수단 표가 적는다. 사이클 문서에는 반복하지 않는다.

## 5. 버전과 사이클명

### 5.1 버전은 사이클 시작에 정한다

- `cycle-propose`가 안을 낼 때 각 안에 버전을 함께 제안한다. `pdca-plan`은 확정 버전을 헤더에 적는다. 이후 문서는 복사한다. close의 태그는 이 버전이다.
- 추천 규칙(semver): 패치 트랙(`fix`) → patch, 사이클 트랙(`refine` `enhance` `adopt` `explore`) → minor, major는 사용자가 새 시대를 열 때만(코드네임과 함께). 규칙은 추천일 뿐 사용자가 바꿀 수 있다.
- 패치는 시작 절차가 없으므로 close가 최신 태그의 patch를 올려 제안하고 확인받는다.
- 근거는 `git tag`(로컬)와 `pdcaw cycle list`(서버) 둘 다 본다. 서버에 예약된 미래 버전(예: 0.14.0이 예약된 채 0.13.2가 최신)이 있으면 그 사다리를 존중한다.
- 시작 시 정한 버전이 close에서 충돌하면(그 사이 다른 사이클이 같은 번호를 썼다면) close가 멈추고 묻는다. 자동으로 올리지 않는다.

### 5.2 사이클명

- 형태 `<종류>-<대상>`, 케밥 케이스. 종류는 `explore` `adopt` `enhance` `refine` `fix` 5개. 날짜 · 버전 · 순번은 넣지 않는다(버전은 폴더 · 파일 어간이 붙인다).
- 사이클명은 **일의 단위**만 말한다. 시대(코드네임)는 인덱스 메이저 절이 맡는다. 둘을 한 이름에 담지 않는다.
- **유일할 필요 없다.** 버전이 앞에 붙어 폴더 · 브랜치 · 서버 어디서도 버전이 열쇠다. `refine-backlog`가 v0.13.2와 v1.4.0에 둘 다 있어도 된다. 다만 `cycle-propose`는 같은 이름이 최근에 있었으면 그 사실을 한 줄 알린다(중복 방지가 아니라 "그때 뭘 했더라"를 상기시키기 위해).
- 종류가 둘에 걸치면 지배적인 하나. 못 고르면 응집도가 낮은 것이니 쪼갠다.

## 6. 스킬

여섯 개. 각 스킬은 자기 폴더에 `SKILL.md`와 필요한 템플릿을 동봉한다. 서버 접근은 `pdcaw`만 쓴다(P6). 문서는 디스크에서 읽는다(P1).

| 스킬 | 언제 | 읽음 | 씀 | 하지 않는 것 |
|---|---|---|---|---|
| `cycle-propose` | 다음 사이클을 정할 때 | `pdcaw backlog list`, 직전 report, git 태그, `pdcaw cycle list` | 없음 | 백로그 변경, 문서 작성 |
| `pdca-plan` | 사이클명 · 버전이 정해진 뒤 | 백로그 detail, 코드, 직전 report §5 Try | plan.md | 코드 수정, 커밋, design 착수 |
| `pdca-design` | plan 확정 뒤 | 확정 plan, 코드 | design.md | 코드 수정, 커밋, 구현 착수 |
| `pdca-do` | design 확정 뒤. `--scope <모듈명 또는 B-n,…>` 없으면 다음 미착수 모듈 | 확정 design, do.md | 코드, do.md, qN.md | design 임의 변경, 기준 변경, 검증 없이 다음 배치 |
| `pdca-close` | do 전 배치 검증 완료 뒤 | plan · design · do, git diff | analysis · report · release · `_INDEX.md`, 태그 | 버전 임의 변경, 폐기 결정, 푸시 전 확인 생략 |
| `backlog-sync` | close 마지막 단계(단독 실행도 가능) | report §3 · §5 · §6 · §7, `pdcaw backlog list/get` | `pdcaw backlog create/update` | `dropped` · `todo` 복원 찍기, 부분 해결 닫기 |

### 6.1 스킬 공통 규칙

- **호출 형태.** `pdca-plan`만 버전 · 사이클명(· 직접 지시 문장)을 받아 브랜치와 폴더를 만든다. `pdca-design` `pdca-do` `pdca-close`는 현재 브랜치 `{버전}-{사이클명}`에서 사이클을 읽는다. 브랜치에 없을 때만 버전을 인자로 준다.
- 시작 시 `.pdcarc.json`으로 프로젝트를 식별한다. 없으면 묻는다.
- 시작 시 프로젝트 `docs/RULE.md`를 읽는다. RULE.md는 **프로젝트별 예외와 종료 훅**만 담는다(§8). 절차 본문은 스킬이 정본이다.
- 선행 문서 헤더 상태를 확인하고 게이트를 지킨다(§2).
- 문서를 쓰기 전에 골격을 먼저 만든다. 헤더 값은 교체한다.
- 이 대화에서만 통하는 지시대명사를 산출물에 쓰지 않는다. PAT · 토큰 · 개인정보를 예시에 넣지 않는다.
- 판단이 갈리는 것은 바꾸지 말고 번호 매긴 질문으로 남긴다(P7).

### 6.2 pdca-close 절차 개요

0. 모드 판정: 현재 브랜치 `{버전}-{사이클명}`에서 버전을 읽는다. 사이클 폴더에 plan.md가 있으면 사이클 모드, 없으면 패치 모드(2 · 3 생략). 패치 브랜치가 아직 없으면 최신 태그 patch +1을 제안해 확인받고 브랜치와 폴더 `v{major}/{버전}-fix-{대상}/`을 만든다.
1. 게이트: (사이클) do.md 배치 전건 `검증 완료`, `*.q*.md` 없음. (공통) 브랜치 CI 초록(RULE.md의 CI 확인 명령 또는 사용자 확인), 버전이 `git tag` · `pdcaw cycle list`와 충돌 없음.
2. analysis.md 작성(감사). 결함은 즉시 수정 · 재검증하거나 report §3으로.
3. report.md 작성.
4. release.md 작성. 직전 태그~HEAD diff와 report를 대조. 직전 release 읽어 연속성 확인.
5. `_INDEX.md` 해당 메이저 절 맨 위에 행 추가.
6. RULE.md의 프로젝트별 종료 훅 실행(README 갱신, 버전 파일 수정 등).
7. 사이클 브랜치에 커밋 · 푸시. 문서 커밋 CI 확인.
8. develop에 `--no-ff` 머지. **푸시 전 사용자 확인.**
9. 머지 커밋에 태그 `git tag -a <v> -m "<cycle> — <한 줄 요약>"`, 태그 푸시. `git rev-parse <v>^{}`가 머지 커밋인지 검증.
10. `pdcaw upload --version <v>`: 릴리즈 생성 + 변경 문서 업로드 + release.md를 릴리즈노트로.
11. develop CI 초록 확인 → `main`을 태그로 `--ff-only` 전진, 푸시. **푸시 전 사용자 확인.** CI가 빨가면 main을 건드리지 않고 패치 트랙을 안내한다.
12. `backlog-sync` 실행.

폐기: 옛 종료 절차 8번(diff txt 저장). 용도(웹 클로드 릴리즈노트 생성)가 4번으로 흡수됐다.

## 7. pdcaw CLI

`upload` 하나짜리에서 스킬의 유일한 서버 접점이 된다. REST는 이미 PAT를 받으므로(`authMiddleware`가 `pdcaw_` 해시 조회) MCP를 거치지 않고 REST를 친다. 모든 명령은 `--json`을 받고, 스킬은 `--json`만 쓴다.

```
pdcaw upload   [--version vX.Y.Z] [--all | --path <p>...]
               --version 시: docs/PDCA/*/{version}-*/ 폴더를 찾아 사이클명 · dir을 얻고
               릴리즈 생성(있으면 재사용) + 변경 문서 업로드
               + 그 폴더의 *.release.md가 있으면 그 내용을 releaseNote로 설정
               기준선은 --version 자신을 제외한 최신 태그 (태그 뒤에 실행해도 diff가 비지 않음)
               --version --path: 소급 릴리즈. 그 경로만 올리므로 중복 전송 없음
               --cycle 필터는 없앤다
pdcaw project  list
pdcaw cycle    list                                  # version · name · dir · hasReleaseNote
pdcaw backlog  list   [--status s,...] [--stale <days>] [--q <text>]   # 요약: id · title · status · priority · openedOn · updatedAt
pdcaw backlog  get    <id>                            # detail 포함 전체
pdcaw backlog  create --title <t> --priority <p> --opened-on <d> [--detail-file <f> | --detail <t>]
pdcaw backlog  update <id> [--status s] [--closed-on d] [--title t] [--priority p]
                           [--append-detail <t|@file>] [--detail-file <f>]
pdcaw doc      collect --stage <s> [--major vN] --out <dir|file>
                                                      # 로컬 docs/PDCA에서 한 stage를 한 폴더로 복사 또는 한 파일로 이어붙임
                                                      # (이어붙일 때 파일마다 경로 헤딩). GUI 선택기용. 서버 불요
pdcaw doc      outline <path> | read <path> [--section <n>] | grep <pattern> [--stage s] [--cycle c]
                                                      # 서버 문서용. CC에서는 로컬 grep을 쓰므로 후순위
```

- `--append-detail`은 클라이언트가 기존 detail을 읽어 새 블록을 앞에 붙이고 통째로 PATCH한다(서버 변경 없이 먼저 가능). 서버에 append가 생기면 그걸로 바꾼다.
- `backlog list`가 요약만 돌려주므로 100건 넘어도 컨텍스트에 들어온다. detail은 `get`으로 필요한 건만. (구현: 서버에 단건 GET · 요약 필터가 생기기 전까지 CLI가 전건을 받아 클라이언트에서 거른다. 서버 §9.2가 붙으면 그쪽으로 옮긴다.)
- 구현 상태: pdcaw 1.0.0 — main 머지 · npm 게시(2026-09-04). 86 테스트, 목 서버 스모크 전 명령 통과. `upload --version`은 서버 §9.1(`cycles.dir`, 6 stage) 이후에만 성공한다.
- `--stale`는 `status=todo`이고 `updatedAt`이 N일 이상 지난 것. backlog-sync 정체 스윕이 쓴다.
- 설정 · 인증 · 루트 판정은 현행 그대로(`.pdcarc.json`, `.env.local`의 `PDCAW_PAT`).
- 인자 표면 변경이므로 pdcaw는 major 범프(v1.0.0).

## 8. RULE.md의 새 역할

절차 본문이 스킬로 옮겨가므로 프로젝트 `docs/RULE.md`는 짧아진다.

- 이 프로젝트가 어떤 체계를 따르는지 한 줄(`pdca-skill v1`).
- 검증 수단: 이 프로젝트에서 가용한 것(예: Unity Test Runner PlayMode, 육안 확인은 사용자).
- 브랜치 이름(`main` `develop`이 아니면 무엇인지)과 CI 확인 명령(없으면 사용자 확인).
- **종료 훅**: close 6번에서 이 프로젝트가 추가로 하는 일. 예: README 갱신, `docs/fishing/fishing-code-guide.md` 갱신, `ProjectSettings/ProjectSettings.asset`의 `bundleVersion` · `AndroidBundleVersionCode`.
- 이 프로젝트만의 예외.

정본 RULE.md 템플릿은 이 레포(`templates/RULE.template.md`)에 둔다.

## 9. PDCA-workspace 변경

### 9.1 stage · 경로 확장 (구현됨)
- `pdcaStageSchema` · `pdca_stage` enum에 `do` `release` 추가. 마이그레이션(dev · main 각각 적용).
- `PDCA_STAGES` 6종, `STAGE_COLOR` 6색, 릴리즈 페이지 문서 버튼 4 → 6, 사이드바 트리 동일.
- **경로를 계산하지 않고 기록한다.** `cycles.yearMonth` → `cycles.dir`(사이클 폴더 경로, 예 `docs/PDCA/v1/v1.2.0-enhance-lyric-sync`). pdcaw가 올린 실제 경로를 저장하고 UI는 `{dir}/{basename(dir)}.{stage}.md`로 문서를 찾는다. 기존 행은 마이그레이션에서 `docs/PDCA/{yearMonth}/{name}`으로 채운다.
- **`cycles_proj_name_uq` 유니크 인덱스를 없앤다.** 사이클명은 겹쳐도 된다(§5.2). 열쇠는 `version`뿐. 409의 `target: 'name'` 분기와 pdcaw의 대응 코드도 지운다.
- 파서 규칙은 하나로 줄인다: `docs/PDCA/**/{stem}/{stem}.{stage}.md`, 폴더명 == 파일 어간, 어간은 `{version}-{name}`. 상위 경로는 자유. pdcaw `cycle-path.ts` 사본도 함께.
- 사이클 생성 시 `name`은 어간에서 버전 접두를 뗀 사이클명(`enhance-lyric-sync`), `version`은 접두(`v1.2.0`). 브랜치명 · 폴더명 · `--version`이 어긋나면 pdcaw가 중단한다.

### 9.2 백로그 API (구현됨)
- `GET /projects/:id/backlog?status=a,b&stale=<days>&q=<text>`: 전체 행(웹 보드용). 필터는 서버.
- `GET /projects/:id/backlog/summary?…`: `detail` 없는 요약 행. 응답 타입이 유니온이 되지 않게 경로로 갈랐다(Hono RPC 타입 전파 때문).
- `GET /backlog/:id` 신설(단건, detail 포함).
- `PATCH /backlog/:id`에 `appendDetail` 필드(서버가 기존 detail 앞에 붙임, `detail`과 동시 지정은 400). 원안 보존을 서버가 책임진다.
- MCP: `backlog_list` 기본 요약 + `staleDays` `q` `detail:true`, `backlog_get` 신설, `backlog_update`에 `appendDetail`. 툴 11개.
- 순수 함수(`filterBacklogRows` `stripDetail` `prependDetail`)는 `shared/backlog.ts` 한곳. pdcaw는 이 엔드포인트를 그대로 쓴다(클라이언트 필터 제거).

### 9.3 문서 API (웹 클로드용, 후순위 — 미착수)
- `document_outline(path)`: 헤딩 트리만.
- `document_read(path, section?)`: 절 번호나 헤딩 텍스트로 그 절만.
- `document_grep(pattern, stage?, cycle?)`: 일치 행 + 전후 N행.
- REST 동형 엔드포인트. CC는 로컬 grep을 쓰므로 이 기능은 웹 문답 품질용이다.

### 9.4 릴리즈노트 (구현됨)
- `release` 문서가 존재하면 릴리즈 페이지는 그것을 렌더한다. `cycles.releaseNote` 필드는 기존 데이터와 수동 입력용으로 남긴다. 둘 다 있으면 문서 우선.
- pdcaw `upload --version`이 release.md를 `releaseNote`에도 써 넣는다(구 UI 호환).

### 9.5 삭제 (구현됨)
- MCP 프롬프트 `backlog_sync` `make_cc_prompt` 삭제. 절차는 스킬이 정본이다.

### 9.6 구현 상태
- main 머지 · 배포 완료(2026-09-04, 마이그레이션 0004 · 0005 적용). §9.1 · §9.2 · §9.4 · §9.5 완료, §9.3(문서 outline/section/grep)은 후순위로 미착수.
- 마이그레이션 `0004_pdca_v1_stage_dir`(enum 값 · `dir` 추가 · `year_month`에서 백필) → `0005_pdca_v1_drop_year_month`(`year_month` · `cycles_proj_name_uq` 제거). dev · main Neon 브랜치에 각각 적용해야 하며, 이는 사용자가 한다. enum 값 추가는 되돌릴 수 없다.
- `tsc -b` · vitest 74 · oxlint · vite build 통과. L1 하네스(실 DB)는 이 세션에서 돌리지 않았다.

## 10. 폐기 목록

| 무엇 | 왜 |
|---|---|
| bkit `pm` `team` `qa` `cleanup` `status` `next` `archive(이동)` 액션 | 안 쓰거나 RULE과 충돌 |
| Context Anchor, Executive Summary, Decision Record Chain, Upstream Context Chain | plan 재진술. 개정마다 자람 |
| Design 고정 3안(A/B/C) | 결정 지점별 실선택지로 대체 |
| Match Rate와 모든 합산 수치, L1/L2/L3 가중 산식 | 항목별 판정으로 대체 |
| Design §9 Clean Architecture, §10 Convention, Plan §7 Architecture Considerations, §8 Convention Prerequisites | 프로젝트 상수. 사이클 문서에 없음 |
| do.template의 Design Anchor, 웹 체크리스트(Security/API/Convention), `// Design Ref:` 주석 규약, `--scope module-N` | 웹앱 전제 또는 B-n 배치로 대체 |
| report §9 Changelog | release 문서로 |
| analysis 코드 품질 · 컨벤션 · 아키텍처 점수 | SC로 plan에 넣지 않은 것은 측정하지 않음 |
| `.bkit/state/pdca-status.json`, Task 시스템 연동 | 문서 헤더 상태가 상태 |
| 종료 절차 8번(diff txt) | release 문서 생성이 흡수 |
| RULE.md의 절차 본문 | 스킬로 이동. RULE은 프로젝트별 훅만 |
| MCP 프롬프트 2개 | 스킬로 이동 |
| 스킬의 MCP 툴 직접 호출과 두 겹 JSON 파싱 지침 | `pdcaw --json` |
| `.tmp` 질문 파일 | `{cycle}.qN.md` |
| 문서 간 마크다운 상호 링크, 인덱스의 폴더 링크 | 파일 이동 · 개명 시 링크 검사 CI가 깨짐. 이름 · 경로 인라인 코드 · id로 참조 |
| "PDCA 문서는 사이클 도중 커밋하지 않는다" | 사이클 브랜치가 develop 역사를 지켜준다. 휘발성 환경에서 미커밋 문서가 더 위험 |
| 사이클명 유일성, 서버 `(project, name)` 유니크, pdcaw `--cycle` | 버전이 열쇠. 사이클명은 꼬리표 |
| develop → main 체리픽 | 역사가 갈라지고 `git describe`가 어긋남. 태그로 ff만 |

### 6.3 구현 상태
- 여섯 스킬 모두 `skills/<이름>/SKILL.md`로 작성됐다. 템플릿은 각 스킬 폴더에 동봉.
- 스킬은 서버를 `npx pdcaw@1 … --json`으로만 만지고 문서는 디스크에서 읽는다. MCP 호출 지시는 어디에도 없다.
- 첫 실전 사이클(sing-diary)이 검증이다. 거기서 나온 마찰은 report §6이 스킬 · RULE · CLI · 서버 중 어디를 고칠지 지목한다.

## 11. 작업 순서

스킬이 CLI에 의존하고 CLI가 서버 API에 의존하므로 **표면을 먼저 고정**하고 구현은 병렬로 간다.

1. 이 문서 확정.
2. 템플릿 6종은 각 스킬 폴더에(`skills/pdca-plan/plan.template.md`, `skills/pdca-close/{analysis,report,release}.template.md` 등), `RULE.template.md`는 `templates/`에 (이 레포).
3. pdcaw 명령 표면 고정 → 구현(`backlog` `cycle` `project`는 기존 REST로 즉시 가능, `upload` releaseNote 연동은 서버 9.1 뒤).
4. 스킬 6개 작성(이 레포). CLI 표면만 알면 서버 없이 쓸 수 있다.
5. 서버 9.1 → 9.2 → 9.5 → 9.4 → 9.3.
6. sing-diary에 RULE.md 배치, 첫 사이클을 새 체계로. 여기서 나온 것이 개편의 검증이다.

## 12. 미결

1. (해결) do 세션 인자: `--scope <모듈 | B-n,…>`, 없으면 다음 미착수 모듈.
2. `pdcaw backlog list --stale`의 기본 일수. backlog-sync 현행은 14일.
3. release 문서의 이모지 · 톤이 프로젝트마다 다를 수 있음(게임 vs CLI). RULE.md 훅으로 톤 지정을 허용할지.
4. 웹 클로드가 서버 문서를 읽을 때(§9.3) 기본을 outline으로 할지 전체로 할지.
5. sing-diary의 검증 수단 목록(RULE.md에 적을 것). 프로젝트를 보고 정한다.
6. §2 모델 열은 제안 상태. 사용자 확정 대기.
