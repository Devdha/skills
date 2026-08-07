---
name: zen
description: Use when 태스크를 설계부터 구현, PR 생성, 리뷰 하네스 수렴, 최종 검증(화면·로직 실환경)까지 한 번에 자율로 진행하고 싶을 때 — "zen", "zen으로 진행해", "태스크 파이프라인", "설계부터 리뷰까지 알아서" 같은 요청. 설계 승인 후 무중단 자율 실행이 필요한 상황에서 사용.
---

# zen — 태스크 자율 파이프라인

## 개요

설계(인터랙티브, 기획 적립: Moonlight는 Notion, 그 외 `docs/plans/`) → 구현 → PR + 리뷰 하네스 → 재리뷰 수렴 → 최종 검증(화면·로직 실환경), 5단계 파이프라인.

**핵심 원칙: 설계 문서가 자율성의 계약서다.** Phase 1의 산출물(design.md)에 의사결정 근거 + 구현 스펙 + 검증 시나리오 + finding 판단 기준이 모두 담기고, Phase 2~5의 모든 자율 판단은 이 문서를 근거로 한다. 사용자 게이트는 설계 승인 단 한 번. 동료 리뷰어 지정(6단계)은 파이프라인 범위 밖 — 사용자가 직접 한다.

**최종 검증은 파이프라인의 마지막이다.** 리뷰 반영으로 코드가 바뀔 때마다 증거가 낡아지는 것을 막기 위해, 검증 증거(스크린샷·실환경 재현 로그)는 리뷰 수렴이 끝난 **최종 head SHA에서 1회** 확보해 PR에 업데이트한다.

## 서브 스킬 (canonical 위치, 모든 호스트 공통)

| 스킬 | 역할 | 본문 |
|---|---|---|
| zen-brainstorm | Phase 1 설계 + Notion 기획 적립 | `~/.claude/skills/zen-brainstorm/SKILL.md` |
| zen-verification | Phase 5 최종 검증 (화면·로직) | `~/.claude/skills/zen-verification/SKILL.md` |
| zen-review-harness | generic 리뷰 하네스 | `~/.claude/skills/zen-review-harness/SKILL.md` |

Claude Code에서는 Skill 도구로, Codex/Grok에서는 위 경로의 파일을 읽어 그대로 따른다. 포인터는 `~/.codex/skills/`, `~/.grok/skills/`, `~/.agents/skills/`에 있다 — **canonical의 frontmatter(description)를 수정하면 포인터들의 frontmatter도 함께 갱신한다** (호스트 트리거는 포인터의 description으로 동작한다).

## 작업 공간 & 설정

- 전역 설정: `~/.zen/config.json` — `s3_bucket`, `s3_prefix`, `s3_region`, `public_url_prefix`, `notion_planning_db`(태스크 기획 Notion DB URL)
- 프로젝트 메모: `~/.zen/projects/{repo}.md` — dev 서버 명령, base URL, 테스트 계정의 **env 변수명**. 캐시하는 것은 변수명뿐이다 — **크리덴셜 값은 어떤 파일에도 기록하지 않는다.**
- `{repo}` = `gh repo view --json nameWithOwner`의 owner/name을 slug화 (`owner-name`). slug 규칙: 소문자, 영숫자 외 문자는 `-`.
- 런 디렉토리(`run_dir`): `~/.zen/runs/{repo}/{branch-slug}/` — `design.md`, `state.json`, `verification/`, `reviews/v{N}/`, `report.md`. 브랜치명은 Phase 1(design.md §0)에서 확정되므로 run_dir도 그 시점에 생성한다. 그 전의 임시 산출물은 `~/.zen/runs/{repo}/_pending/`에 두었다가 옮긴다.
- `state.json` — Phase 1에서 run_dir 생성 시 아래 초기값으로 생성:

  ```json
  {"phase": 1, "branch": null, "base_branch": null, "pr_number": null,
   "verify_attempts": 0, "fix_attempts": 0, "review_round": 0,
   "design_hash": null, "harness": null, "harness_version": null,
   "held_findings": [], "external_review_url": null,
   "planning_page_url": null, "post_verify_review_done": false,
   "started_at": "<UTC>"}
  ```

  - `phase` = **현재 진행 중인** Phase. 각 Phase 진입 시 갱신. 재개 시 해당 Phase를 처음부터 다시 실행한다.
  - 카운터는 전부 "완료 후 +1": `verify_attempts`는 검증 실행이 끝나 results.json 저장 직후, `review_round`는 그 라운드의 findings 처리(decisions.json 기록)까지 마친 직후. 따라서 "최대 3회" = 완료된 실행 3회.
  - `pr_number`는 PR 생성 **직후 즉시** 기록한다 (Phase 전환을 기다리지 않는다).
  - `held_findings`는 각 라운드 decisions.json에서 `held`/`rejected` 항목을 누적한 요약이다 (원본은 항상 decisions.json).
  - `planning_page_url` = Phase 1에서 생성한 기획 산출물 위치 — Notion 페이지 URL(Moonlight) 또는 `docs/plans/` 파일 경로(그 외). `post_verify_review_done` = Phase 5 구조적 수정에 대한 추가 하네스 라운드 사용 여부 (파이프라인당 1회 한).

## Phase 0 — preflight

1. **재개 감지**: 현재 브랜치의 slug로 state.json이 존재하고 완료 전이면 재개 모드 — state·산출물(design.md, results.json, reviews/)을 읽고, git/gh 실제 상태(브랜치, head, PR)와 대조한다. `pr_number`가 비어 있으면 `gh pr list --head {branch}`로 복구. **마지막 갱신이 24시간 이상 지났거나 실제 상태와 불일치하면 요약을 보고하고 사용자 확인 후 재개한다.** design.md가 있고 phase ≥ 2면 Phase 1 게이트는 다시 밟지 않는다. 완료된 run_dir에 새 태스크로 시작하는 경우엔 `{branch-slug}.{완료시각}`으로 아카이브하고 새로 만든다.
2. git repo, 기본 브랜치, **PR base 브랜치**(프로젝트 컨벤션 문서·기존 PR 관찰로 판단) 확인 — base는 state에 기록. `gh auth status` 미인증이면 여기서 중단하고 안내 (aws와 동일 규칙).
3. `~/.zen/config.json` 없으면 세팅 마법사: S3 버킷/리전/public URL prefix 질문 → `aws sts get-caller-identity` 자격 확인 → **테스트 객체를 업로드하고 비인증 `curl -sI`로 200 확인** → **ListBucket 차단 확인** (무자격 목록 조회가 거부되는지) — 리스팅이 열려 있으면 중단하고 버킷 정책 수정 안내. lifecycle 정책을 쓰면 만료 시 PR 이미지가 깨진다는 트레이드오프를 고지한다. **`notion_planning_db`(태스크 기획 DB URL)도 여기서 질문해 저장한다.**
4. **기획 적립 위치 확인**: **Moonlight면** `config.notion_planning_db`가 있고 Notion 연동(MCP 등)으로 해당 DB에 접근 가능한지 확인한다 — config에 없거나 접근 실패면 **게이트 전이므로 지금 사용자에게 확인받는다** (Phase 1 승인 직후 기획 페이지 생성이 게이트라서(예외 5), 여기서 채우지 않으면 승인 직후 중단된다). **그 외 프로젝트는 확인 불필요** — 기획은 레포 `docs/plans/` 파일로 적립된다.
5. 프로젝트 규칙 파악: CLAUDE.md/AGENTS.md, 브랜치·PR·**커밋 메시지** 컨벤션, 프로젝트 테스트 작성 스킬.
6. **리뷰 하네스 선택·고정** (프로젝트 전용 하네스 우선 — 임의 폴백 금지):

   1. **탐지**: 레포의 `.claude/skills/`, `.agents/skills/`에서 이름에 `review-harness` 또는 `code-review`가 들어간 스킬을 찾는다. **description에 DEPRECATED가 표기된 것은 제외. 복수 매칭 시 `review-harness`가 이름에 포함된 쪽 우선.**
   2. **프로젝트 전용 하네스가 하나라도 있으면 그것을 `state.harness`에 고정한다.** 예: Moonlight → **`moonlight-code-review-harness` 필수**. 이 경우 **zen-review-harness로 대체·축소 실행하는 것은 금지**다. “시간이 없어서”, “비슷한 리뷰를 직접 하면 된다”는 사유로 전용 하네스를 건너뛰면 **Phase 3 미실행**으로 간주한다.
   3. **호스트별 실행 주체** (핵심 — 누가 스킬을 돌리는가):
      | 호스트 | 전용 하네스 실행 |
      |---|---|
      | **Claude Code** | 현재 세션에서 `state.harness` 스킬을 **직접** 실행 (Skill 도구 / 스킬 본문 수행). |
      | **Grok** | 현재 세션에서 `state.harness` 스킬 본문을 **읽고 직접** 수행. **`claude -p` 위임 금지.** |
      | **Codex** | 현재 세션에서 전용 하네스를 흉내 내지 말고 **`claude -p`로 위임** (Phase 3 §2). |

      전용 하네스 문서에 “Claude Code 전용”이 적혀 있어도 **Grok은 폴백하지 않고** 같은 스킬 워크플로우를 현재 호스트 도구(에이전트 spawn, Bash, MCP 등)로 수행한다. 병렬 에이전트가 없으면 스킬이 허용하는 범위에서 순차 패스로 진행하되, **산출물 경로·Notion·전 Phase 생략은 금지**다.
   4. **Moonlight + Notion (필수 완료 조건)**: `state.harness`가 `moonlight-code-review-harness`이면 하네스 스킬 **Phase 6 Notion "PR 리뷰" DB 저장까지 포함**해야 한 라운드가 완료된다. `_workspace/pr-{n}/v{M}/` findings만 있고 Notion 페이지 URL이 없으면 **그 라운드는 미완료**다 — `review_round`를 올리지 말고, `state.external_review_url`이 비어 있으면 완료/수렴 성공으로 보고하지 않는다. Phase 0에서 Notion MCP(또는 하네스가 쓰는 Notion 연동) 가용성을 확인한다. 부재·권한 실패면 **예외 5**로 중단·보고 (로컬 JSON만으로 대체 완료 금지).
   5. **zen-review-harness 폴백은 레포에 프로젝트 전용 하네스가 아예 없을 때만** 허용한다. Codex에서 `claude -p` 위임이 실패하면 예외 5로 중단·보고하고, 무단으로 zen-review-harness에 갈아타지 않는다.
   6. 선택 결과를 `state.harness`에 고정하고, 이후 라운드에서 다른 스킬로 바꾸지 않는다.
7. **Phase 5 의존성 사전 점검 (게이트 전)**: 예상되는 검증 유형별로 아래를 **지금** 확인한다. 부재 시 사용자에게 확인받는다 — 승인 후에는 질문할 수 없다.
   - **화면**: `~/.zen/config.json` (S3) — 없으면 Phase 5에서 `upload_failed` 가능함을 고지(추측 버킷 금지) · `~/.zen/projects/{repo}.md`의 **surface 런북**(web/extension 등) + 로그인 방식(mock/env 변수명) · 브라우저 도구(Playwright 등) · aws cli 자격(업로드용) · 모바일 시나리오가 예상되면 디바이스 에뮬레이션 가능 여부
   - **로직**: 로컬 스택 기동 명령 · 실제 호출 수단(curl/스크립트) · 테스트 계정/시드 데이터 방법
   - **공통**: **Moonlight 하네스면 Notion MCP 연동**. 상세 preflight 절차는 **zen-verification §0**을 따른다 (시나리오 전 로그인/에셋 스모크 포함).

## Phase 1 — 설계 + 기획 적립 (인터랙티브, 유일한 게이트)

zen-brainstorm 실행 → `{run_dir}/design.md` 산출. **게이트 질문과 승인 판정은 zen-brainstorm의 종료 게이트가 수행한다 — zen이 별도로 다시 묻지 않는다.** 인터뷰 트랙(계기 → 유형 분기: 버그 라이트 재현 / 피쳐 고객 가치 → 대안 비교)과 design.md §1의 의사결정 기록은 zen-brainstorm 본문을 따른다.

**승인 직후 순서 (모두 Phase 1 완료 조건):**

1. **기획 적립 (게이트)**: zen-brainstorm 종료 게이트가 수행한다 — **Moonlight는** `config.notion_planning_db`에 태스크당 1페이지, **그 외 프로젝트는** 레포 `docs/plans/{YYYY-MM-DD}-{branch-slug}.md` 파일 (속성·내용 구성은 zen-brainstorm 본문). **적립 실패면 Phase 2로 진입하지 않고 예외 5로 중단·보고한다.** 성공 시 URL/경로를 `state.planning_page_url`과 design.md §0에 기록.
2. design.md의 해시를 `state.design_hash`에 기록한다. **이후 design.md는 수정하지 않는다 (read-only 계약).** 판단 근거가 부족해지면 design.md를 고치는 대신 decisions.json에 자체 판단으로 기록하거나 예외 4로 중단한다.
3. design.md §0의 브랜치명으로 run_dir을 확정한다.

## Phase 2 — 구현

1. design.md §0의 브랜치명으로 브랜치 생성 (base = `state.base_branch`). 커밋 메시지도 프로젝트 컨벤션을 따른다. **비 Moonlight 프로젝트면 Phase 1의 `docs/plans/` 기획 파일을 브랜치 첫 커밋에 포함한다** (PR로 팀 공유되는 경로다).
2. 구현 + 테스트 동반 작성 (프로젝트 테스트 스킬이 있으면 사용).
3. lint / typecheck / test / build 통과할 때까지 수정. **상한: 동일 실패(같은 테스트/에러 시그니처)가 수정 시도 3회 연속 재현되거나 수정 시도 총 10회 초과 시, 환경/스코프 문제로 간주하고 중단·보고** (`fix_attempts`로 추적. 환경 기인 실패 의심 — 런타임 버전, 외부 서비스 — 이면 즉시 보고).
4. **push 가드**: 모든 push 직전 `git branch --show-current`가 `state.branch`와 일치하는지 확인한다. 기본/base 브랜치이거나 불일치면 push하지 않고 중단·보고. force push 금지.

## Phase 3 — PR + 리뷰 하네스

1. push + PR 생성: **`--base {state.base_branch}`를 명시**하고 프로젝트 PR 템플릿 준수. push/PR 생성 실패는 1회 재시도 후 중단·보고.

   - **PR 바디의 검증 자리에는 placeholder를 넣는다** (검증은 Phase 5에서 최종 head로 실행):

     ```markdown
     ## 검증 (zen-verification)
     > 🔍 최종 검증(화면/로직)은 리뷰 수렴 후 최종 커밋 기준으로 첨부됩니다.
     ```

   - §0 검증 유형이 **N/A**(화면 표면 없는 태스크)면 placeholder 대신 design.md §5의 대체 증거(테스트/CLI 출력/로그)를 지금 첨부한다.
   - PR 생성 직후 `state.pr_number` 기록. **Moonlight면** Notion 기획 페이지의 `PR` 열과 `상태`(진행 중)를 갱신한다 (best-effort — 실패해도 중단하지 않고 report에 명시). docs 파일 모드는 갱신 단계가 없다 (기획 파일이 PR에 포함돼 있다).

2. 하네스 실행 (`state.harness` 고정값, 라운드 1). **`state.harness` 스킬을 호스트 규칙대로 실행한다 — 오케스트레이터의 자체 코드 리뷰 메모는 하네스 실행이 아니다.**

   | 호스트 | 실행 방식 |
   |---|---|
   | **Claude Code** | `state.harness` 스킬 **직접** 실행 (Skill 도구). 전용 하네스면 스킬 본문 Phase 1~7을 생략·축소하지 않는다. |
   | **Grok** | 레포/`.agents`의 `state.harness` 스킬 본문을 **읽고 현재 세션에서 직접 수행**. **`claude -p`로 넘기지 않는다.** 스킬이 요구하는 워크트리·`_workspace`·Codex 스크립트·Notion MCP를 이 세션 도구로 돌린다. |
   | **Codex** | 전용 하네스는 여기서 직접 돌리지 않는다. **`claude -p` 위임만**: `claude -p "<하네스 스킬명> 스킬로 PR #{n}을 리뷰하라. 컨텍스트: design.md=<run_dir 절대경로>/design.md (§4 판단 기준 로드), 이전 라운드=<run_dir 절대경로>/reviews/v{N-1}/ (있으면 수렴 모드)"` — 경로 절대경로, 충분한 타임아웃, 스킬명은 영숫자·하이픈만. 위임 실패 시 예외 5 중단·보고 (zen-review-harness 무단 대체 금지). |
   | (폴백) **zen-review-harness** | 레포에 전용 하네스가 **없을 때만**. 현재 호스트에서 본문 수행 (병렬 없으면 관점별 순차). |

   **Moonlight 강제 규칙**: 레포가 `corca-ai/moonlight`(또는 동일 monorepo worktree)이면 `state.harness`는 항상 `moonlight-code-review-harness`다. Claude/Grok은 그 스킬을 **직접** 돌리고, Codex만 `claude -p`로 돌린다. 스킬 본문의 **Phase 1~7 전부**(특히 **Phase 6 Notion "PR 리뷰" 저장**)가 한 라운드의 완료 조건이다. `_workspace`만 채우고 Notion URL이 없거나, 수 분 내 자체 findings만 쓰면 **하네스 미실행/미완료**다.

3. **산출물 회수 (정규화 계약)**: 하네스가 무엇을 남기든, **zen이 결과를 zen 스키마로 정규화해 `{run_dir}/reviews/v{N}/findings.json`에 직접 저장한다** — `{summary, findings[]: {path, line, body, category, severity∈{critical,warning,suggestion,question}}, v_verification[]}`. 통합 파일을 남기지 않는 하네스(예: moonlight 하네스는 관점별 `opus_*`/`codex_*` findings JSON을 repo의 `_workspace/pr-{n}/v{M}/`에 남긴다)는 관점별 파일을 병합해 저장한다. 하네스가 자체 회차 번호(v{M})를 쓰면 `state.harness_version`에 기록하고 report에 병기. **외부 저장소 링크(Notion 페이지 등)는 `state.external_review_url`에 반드시 기록**한다.

   **Moonlight 라운드 완료 게이트** (Phase 3·4·5 공통): 아래를 모두 만족해야 해당 라운드의 decisions 기록·`review_round` 증가·수렴 판정에 들어갈 수 있다.
   1. `_workspace/pr-{n}/v{M}/`에 관점별 findings(또는 스킬이 정한 동등 산출물) 존재
   2. 하네스 Phase 6이 남긴 **Notion "PR 리뷰" 페이지 URL**이 있고 `state.external_review_url`에 저장됨
   3. zen 정규화 `findings.json` 저장

   Notion 저장이 스킵·실패하면 그 라운드는 **미완료** — 로컬 findings만으로 완료 처리 금지. 1회 재시도 후에도 실패 시 예외 5로 중단·보고.

4. finding 처리 (심각도 하이브리드):
   - **기각 불가**: 보안·데이터 손상·크래시에 해당하는 critical은 design.md §4로도 기각할 수 없다. 반영하거나, 반영이 §2 스코프를 벗어나면 중단·보고.
   - `critical`/`warning` → 반영. 단 (a) §4의 의도적 트레이드오프와 상충 → 반영하지 않고 `rejected`로 기록(근거 조항 인용), (b) **수정 범위가 §2의 변경 대상을 벗어나는 경우**(변경 셋 밖 다수 파일, 모듈 구조 변경, 대규모 리팩토링) → critical이면 예외 4로 중단·보고, warning이면 `held`.
   - `suggestion` → 반영하지 않고 `held` (reason은 §4에서 찾고, 해당 조항이 없으면 자체 판단 사유).
   - `question` → **`gh pr comment`로 즉시 답변 게시 (질문 원문과 파일:라인 인용).** 하네스의 PENDING draft 리뷰 플로우는 zen에서 사용하지 않는다.
   - **decisions.json은 zen이 단독 기록**: `{run_dir}/reviews/v{N}/decisions.json` = `[{finding, action: "applied|held|rejected", reason}]`. (하네스는 이 파일을 다음 라운드에 로드만 한다.) 예외로 중단하는 경우에도 그때까지 판정한 finding은 decisions.json에 기록하고 중단 사유를 남긴다 — 중단된 라운드는 decisions가 완결되지 않았으므로 review_round를 증가시키지 않는다.
   - **반영 후 재검증**: 반영 커밋 후 lint/test 재실행(Phase 2 §3 상한 공유). **리뷰 루프 중에는 화면 재검증을 하지 않는다** — 화면 증거는 Phase 5에서 리뷰 수렴이 끝난 최종 head SHA로 1회 확보한다 (이 재배열의 목적).
   - **외부 이력 저장소 갱신**: 하네스가 외부 저장소를 재리뷰 입력으로 쓰는 경우(예: moonlight 하네스의 Notion `채택` 컬럼), decisions 기록 후 그 판정 필드를 갱신한다 — `applied`→채택, `held`→보류, `rejected`→기각, 사유 필드에 `zen 자동 판정: {reason}` 표기. **이 갱신이 없으면 다음 라운드의 반영 검증이 공전해 무검증 통과가 된다.** Moonlight에서는 이 단계가 선택 사항이 아니라 **다음 라운드 진입 조건**이다.

## Phase 4 — 재리뷰 수렴 루프

1. 하네스 재실행 (수렴 모드 — 이전 라운드 findings/decisions를 전달하거나, 하네스가 자체 이력을 쓰면 Phase 3 §4의 외부 저장소 갱신이 선행되었는지 확인). **Moonlight는 이전 라운드 Notion 페이지가 있어야 모드 B 재리뷰가 성립한다** — `state.external_review_url` 또는 Notion 이력 없이 로컬만으로 재리뷰 완료를 선언하지 않는다.
2. 산출물 회수(Phase 3 §3, **Notion URL 포함 완료 게이트**) 후 findings를 **Phase 3 §4와 동일한 정책으로 처리**하고 push. Moonlight면 decisions 반영을 Notion `채택` 컬럼에 다시 기록한다.
3. **진동 감지**: v{N}에 v{N-1}에서 `applied` 처리된 finding이 재출현하거나, 하네스가 수렴 실패(이슈 수 증가)를 보고하면 — 라운드 잔여와 무관하게 즉시 중단하고 인계 (예외 6).
4. 종료 판정 (그 라운드의 decisions 기록까지 마친 뒤): **critical+warning == 0 → 수렴 완료, Phase 5로 진행. review_round == 3인데 critical 또는 warning 잔존 → 남은 이슈와 함께 인계.** (suggestion/question은 보류 항목이므로 종료 조건에 포함하지 않는다.) **Moonlight는 마지막 라운드의 Notion URL이 `state.external_review_url`에 있을 때만 수렴 완료로 본다.**
5. 수렴 완료 시 **CI 확인**: `gh pr checks {n}` — 실패한 check는 Phase 2 §3 정책으로 수정, 외부 장애/장시간 대기면 "완료"가 아니라 인계로 보고한다.

## Phase 5 — 최종 검증 + PR 증거 업데이트

**§0 검증 유형이 `화면`, `로직`, `화면+로직` 중 하나면 이 Phase는 무조건 실행한다.** 스킵 가능한 유일한 경우는 §0이 **N/A**(실행 표면 없는 태스크 — 문서·주석·설정)일 때뿐이다 — 그때만 state에 기록하고 종료 절차(§4)로 간다 (대체 증거는 Phase 3 §1에서 이미 PR에 첨부됨).

§0이 N/A가 아닐 때:

1. **반드시 zen-verification을 실행**한다 — 리뷰 수렴이 끝난 **최종 head SHA** 기준, §0 유형에 해당하는 시나리오 전부(`화면+로직`이면 둘 다, 모바일 지정 시나리오는 모바일 뷰포트 포함): design.md §3 시나리오 → 실환경 재현(브라우저 캡처 / 실제 API 호출·명령) → `{run_dir}/verification/results.json`. 완료 후 **PR 바디의 placeholder를 검증 결과 표로 교체**한다 (증거가 확보된 커밋 SHA 병기):

   ```markdown
   ## 검증 (zen-verification) — {commit SHA}
   | 시나리오 | 결과 | 증거 |
   |---|---|---|
   | 인용 팝업 열림 (PC) | ✅ | ![인용 팝업](https://…s3…/….png) |
   | 인용 저장 API | ✅ | `POST /api/quotes` → 201 (상세는 접기) |
   ```

   스크린샷은 S3 URL만 사용한다 (PR 첨부파일(user-attachments)은 API 자동화 불가, release 업로드 금지). 로직 증거는 요약 + `<details>` 본문 — 마스킹 규칙은 zen-verification을 따른다. **증거는 항상 PR head SHA에 대응해야 한다** — 검증 완료 후 어떤 이유로든 새 커밋이 생겼으면(§3의 검증 단계 수정 포함) 관련 시나리오를 재검증하고 표를 갱신한 뒤 종료한다. 재검증의 성격 판정·카운터는 §3 규칙을 따른다. **단, 제품 코드를 건드리지 않는 커밋(검증 스크립트 `tests/*.zen.spec.ts`, 문서)은 재검증을 트리거하지 않는다** — 검증 스크립트 커밋은 검증 완료 직후, 표 갱신 전에 넣는다.

2. **단위 테스트·lint·mock 호출·자체 메모는 최종 검증의 대체가 아니다.** `verification_type: alternative_unit_tests`로 `overall_pass: true`를 찍고 Phase 5를 완료 처리하는 것은 **금지**. 단위 테스트는 Phase 2 완료 조건이지 Phase 5 증거가 아니다. 로직 검증은 실제 서버 프로세스에 실제 요청이 나가야 한다.
3. **실패(기대 결과 불일치) 시 조건부 처리** — 수정의 성격을 먼저 판정한다. **검증 실행은 성패·수정 성격과 무관하게 완료 시마다 verify_attempts를 소모한다 (state 규칙과 동일 — 최대 3회, 소진 후에도 실패면 중단하고 실패 사유 + 증거와 함께 보고 — 예외 1).**
   - **경미한 수정** (변경 셋 내 파일에 국한 + 공개 인터페이스·상태 구조·데이터 스키마 불변): 수정 커밋 → **재검증만**. 수정 커밋은 report에 "검증 단계 수정"으로 명시하고 CI를 재확인한다.
   - **구조적 수정** (그 외 — 변경 셋 밖 파일, 인터페이스/상태 구조/스키마 변경): 수정 커밋 → **`state.harness` 수렴 모드 1라운드 추가 실행** (Phase 3 §2~4와 동일 규칙 — findings 처리·decisions 기록·Moonlight 완료 게이트 포함. **파이프라인당 1회 한** — `post_verify_review_done`으로 추적. `review_round` 상한 3회와는 별도로 센다) → 재검증 → CI 재확인. **추가 라운드에서 critical/warning이 잔존하면 예외 2에 준해 중단·인계한다.** 이미 1회를 썼는데 또 구조적 수정이 필요하면 예외 6으로 중단·인계.
   - 어느 쪽인지 애매하면 구조적 수정으로 취급한다.
4. **환경 부재**(크리덴셜, dev 서버 기동 불가, **외부 서비스 의존으로 로컬 스택 재현이 원천 불가한 로직 시나리오**, Playwright 불가, S3 업로드 설정 부재 등)는 verify_attempts를 소모하지 않고 **예외 5로 중단·보고**한다 — 사용자에게 무엇을 채워야 하는지 명시. **환경을 채우지 않은 채 단위 테스트로 우회 완료하지 않는다.** S3만 실패하고 로컬 스크린샷은 있으면 로컬 경로 + `upload_failed: true`로 기록하고 zen-verification 규칙에 따르되, **스크린샷 자체를 생략하지 않는다.**

**종료 절차 (검증 통과 또는 N/A):**

1. **기획 산출물 갱신 (Moonlight만)**: Notion 기획 페이지 `상태` → `완료` (중단·인계로 끝나면 `중단`). best-effort — 실패 시 report에 명시. docs 파일 모드는 갱신하지 않는다 (PR 상태가 대변한다).
2. `{run_dir}/report.md` 작성 후 최종 보고: PR 링크, **기획 산출물(Notion URL 또는 docs 경로)**, **Notion 리뷰 URL**, 반영/보류/기각 내역(근거 포함), 수렴 추이(로컬 라운드 기준 — 하네스 자체 지표와 산식이 다르면 병기), 검증 표(+커밋 SHA)와 검증 단계 수정 내역, 사용한 하네스·어댑터·harness_version. "남은 단계: 리뷰어 지정은 직접 진행하세요."

## 자율 실행 규칙

- **사용자 개입은 항상 파이프라인보다 우선한다.** 실행 중 사용자 메시지가 오면 현재 작업 단위를 마치는 대로 멈추고 응답한다.
- Phase 1 승인 이후 사용자에게 질문하지 않는다. 예외 상황에서는 실행을 중단하고 상황을 보고한 뒤, 필요한 결정을 질문하고 응답을 기다린다. 예외:
  1. 최종 검증 3회 실패
  2. 리뷰 3라운드 후에도 critical 또는 warning 잔존
  3. 파괴적 작업 필요 (force push, 데이터 삭제, 설정 변경)
  4. design.md로 판단할 수 없는 스코프 변경 (스코프를 벗어나는 finding 반영 요구 포함)
  5. 환경 부재 — 크리덴셜, 도구, 권한, S3/CI/Notion 접근 (관련 시도 카운터를 소모하지 않는다)
  6. 진동/수렴 실패 감지 (Phase 5 구조적 수정 2회째 포함)
- Phase 전환 시 state.json 갱신 + 사용자에게 한 줄 진행 통보 (질문이 아니다). **Phase 하나가 30분을 넘기거나 전체가 2시간을 넘기면 통보에 경과 시간과 남은 단계를 포함한다.**

## 흔한 실수

- design.md 없이 Phase 2 진행 → 금지. finding 처리 근거가 사라져 판단이 임의적이 된다.
- **기획 적립(Moonlight: Notion 페이지 / 그 외: docs/plans 파일) 실패를 무시하고 Phase 2 진입** → 금지. Phase 1의 게이트다 — 예외 5로 중단·보고한다.
- **비 Moonlight 프로젝트에서 Notion에 기획을 적립하거나, docs 기획 파일을 브랜치 커밋에서 누락** → 금지. 적립 위치는 프로젝트로 결정되고, 파일 모드의 공유 경로는 PR이다.
- 재리뷰 라운드에서 새 nit 발굴 → 수렴 위반. 하네스의 수렴 모드 지침을 따른다.
- 서브 스킬 결과를 대화 기억에만 의존 → 금지. 모든 단계 산출물은 run 디렉토리 파일로 남기고 파일에서 다시 읽는다 (호스트 간 이동·재개 가능성).
- **리뷰 루프 중에 최종 검증을 실행하거나, 리뷰 전에 확보한 증거를 최종 증거로 사용** → 금지. 검증 증거는 Phase 5에서 최종 head SHA로 확보한다. head와 불일치하는 증거는 위조와 같다.
- **§0이 N/A가 아닌데 브라우저 캡처·실호출 증거 없이 단위 테스트만으로 Phase 5 완료 보고** → 금지. 예외 5(환경 부재)로 중단하거나, 환경을 갖춘 뒤 zen-verification을 실행한다.
- **로직 검증을 함수 직접 호출·mock 클라이언트로 대체하거나, 화면 없는 태스크를 N/A로 분류해 검증을 건너뜀** → 금지. API·배치·데이터 처리는 로직 시나리오로 실환경 재현한다.
- **Phase 5 구조적 수정을 하네스 재실행 없이 push하고 완료 선언** → 금지. 수렴 모드 1라운드(1회 한)를 거친다.
- 하네스 탐지에서 DEPRECATED 스킬을 선택 → Phase 0 §6의 제외 규칙을 따른다.
- design.md를 승인 후에 수정해 판단 근거를 소급 생성 → read-only 계약 위반. decisions.json에 기록하거나 중단한다.
- **Moonlight 등 전용 하네스가 있는 레포에서 zen-review-harness로 대체하거나, 오케스트레이터가 짧게 자체 리뷰한 뒤 harness 완료로 보고** → 금지. 전용 하네스 미실행.
- **Grok에서 전용 하네스를 `claude -p`로 위임** → 금지. Grok/Claude는 스킬 직접 실행, **`claude -p`는 Codex 호스트 전용**.
- **Codex에서 전용 하네스 본문을 직접 흉내** → 금지. Codex는 `claude -p` 위임.
- 전용 하네스 산출물 없이(`_workspace/pr-{n}/…`, Notion URL 등 하네스가 남기는 증거 없음) `reviews/v{N}/findings.json`만 채우고 수렴 완료 선언 → 금지.
- **Moonlight에서 Notion "PR 리뷰" 저장(Phase 6)을 스킵하거나 실패했는데 라운드 완료/파이프라인 완료로 보고** → 금지. `_workspace`만으로는 미완료. `state.external_review_url` 필수.
- **Moonlight 재리뷰를 Notion 이력 없이 로컬 findings만으로 모드 B 완료 처리** → 금지. 이전 라운드 Notion + 채택 컬럼 갱신이 선행되어야 한다.
