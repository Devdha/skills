---
name: zen-review-harness
description: Use when PR을 다관점으로 리뷰하고 수렴형 재리뷰까지 해야 하는데 프로젝트 전용 리뷰 하네스 스킬이 없을 때만 — zen Phase 3/4의 generic 폴백. Moonlight는 moonlight-code-review-harness를 Claude/Grok이 직접 실행, Codex만 claude -p.
---

# zen-review-harness — generic 다관점 리뷰 하네스

## 개요

moonlight 코드리뷰 하네스의 일반화 버전. PR 기반 4관점(제품/코드/아키텍처/테스트) 리뷰 + 수렴형 재리뷰. 리뷰 이력은 로컬 `~/.zen/runs/{repo}/{branch-slug}/reviews/v{N}/`에 저장한다 (외부 DB 의존 없음).

**프로젝트에 전용 하네스 스킬이 있으면 이 스킬을 사용하지 말고 그것을 사용한다** (`.claude/skills/`, `.agents/skills/`에서 이름에 `review-harness`/`code-review`가 들어간 스킬 확인. DEPRECATED 제외).

**특히 Moonlight (`corca-ai/moonlight`)**: 반드시 `moonlight-code-review-harness`다. 이 스킬(zen-review-harness)로 대체·축소 실행하는 것은 **금지**.

호스트별 실행 (zen Phase 0 §6 / Phase 3 §2):
- **Claude / Grok**: `moonlight-code-review-harness`를 **현재 세션에서 직접** 실행
- **Codex**: `claude -p`로 위임
- 실패 시 이 스킬로 갈아타지 말고 중단·보고
- **Notion "PR 리뷰" 저장(moonlight harness Phase 6)까지 완료**해야 한 라운드 완료. 로컬 `_workspace`만으로는 미완료 (zen `state.external_review_url` 필수)

## 수렴 원칙

같은 PR을 반복 리뷰할 때 이슈 수는 감소해야 한다.

- **첫 리뷰(v1)**: 마지막 리뷰라는 각오로 완전 탐지. 리팩토링/이동/이름 변경이 있으면 파생 drift(임포트, 문서 참조, 테스트 매핑, 용어)를 선제 검증.
- **재리뷰(v2+)**: 1차 미션은 이전 라운드 **채택 이슈의 반영 검증**. 추가로 **이전 라운드 이후 변경된 diff(반영 커밋)에 한해서는 critical/warning 탐지를 항상 수행한다** — 반영이 만든 새 결함을 놓치지 않기 위해서다. 변경되지 않은 코드에 대한 신규 발굴과 nit/스타일/취향 발굴은 금지. 이전에 기각·보류된 이슈 재제기 금지. findings가 없으면 빈 배열이 정답.
- 라운드마다 이슈 수 추이(v1: n → v2: m)를 출력하고, 증가하면 `⚠️ 수렴 실패`를 표시한다.

## 실행

1. **컨텍스트**: `gh pr view {n} --json title,body,files,headRefName,baseRefName` + `gh pr diff {n}`. PR body에 기획서/설계 문서 링크가 있으면 수집 (설계 의도에 맞는 구현을 지적하는 false positive 방지). zen 파이프라인 안이라면 design.md §4(판단 기준)를 반드시 로드.
   회차 `N`: zen 안에서는 `state.review_round + 1`, 단독 호출이면 `reviews/` 하위 최대 v{k}+1 (없으면 1). 단독 호출 시 branch-slug는 headRefName으로 만든다. 재리뷰면 `reviews/v{N-1}/findings.json`과 `decisions.json`을 로드.

2. **4관점 리뷰**:
   - Claude Code: Agent 도구로 4관점을 병렬 실행 (각자 findings JSON 산출).
   - 병렬 에이전트가 없는 호스트(Codex/Grok 단독 등): 4관점을 **순차의 독립 패스**로 수행. 한 패스에 관점을 섞지 않는다 (관점 누락 방지).

   | 관점 | 보는 것 |
   |---|---|
   | 제품 | UX/동작 누락, 기대 플로우, i18n(다국어 프로젝트면), 이벤트 트래킹, 용어 일관성 |
   | 코드 | 정합성, 타입 안정성, 보안, 효율, 기존 패턴과의 일관성, 불확실한 부분 질문 |
   | 아키텍처 | 파일 위치/co-location, 결합도, 크로스 모듈 영향, 레거시 잔재, 리팩토링 파장 |
   | 테스트 | 신규 동작 커버 여부, 테스트 설계 품질, 변경으로 어긋난 기존 테스트 전제 |

3. **통합**: 관점 간 중복 병합, 심각도 조정, `reviews/v{N}/findings.json` 저장:

   ```json
   {
     "summary": "1-2줄 요약",
     "findings": [
       {"path": "src/…", "line": 42, "body": "…", "category": "제품|코드|아키텍처|테스트",
        "severity": "critical|warning|suggestion|question"}
     ],
     "v_verification": [
       {"past_id": "v1-3", "title": "…", "status": "반영됨|부분 반영|미반영|오반영", "notes": "…"}
     ]
   }
   ```

   심각도: `critical`=반드시 수정, `warning`=수정 권장, `suggestion`=제안, `question`=질문. `v_verification`은 재리뷰(v2+)에서 필수 (첫 리뷰에서는 빈 배열).

4. **리포트 출력**: 심각도순으로 사용자(또는 zen 오케스트레이터)에게 출력. 재리뷰면 이전 채택 이슈별 반영 상태(`v_verification`)를 먼저 보고.

5. **처리 기록**: `decisions.json`은 finding을 **처리하는 주체**(zen 오케스트레이터 또는 사용자)가 기록한다 — `[{finding, action: "applied|held|rejected", reason}]`. 하네스는 이 파일을 다음 라운드에 로드만 하고 직접 쓰지 않는다. 수렴 실패(이슈 수 증가)를 감지하면 리포트에 `⚠️ 수렴 실패`를 명시해 호출자(zen)가 중단·인계를 판단하게 한다.

## 흔한 실수

- 재리뷰에서 4관점 전부 신규 탐지 모드로 재실행 → 수렴 위반. 재리뷰의 중심은 반영 검증이다.
- design.md §4의 의도적 트레이드오프를 critical로 지적 → 설계 문서가 우선. 지적 대신 decisions에 rejected로 기록.
- findings를 대화 텍스트로만 남김 → 금지. 반드시 findings.json 파일로 저장 (호스트 간 회수, 라운드 간 전달에 필요).
- Moonlight 등 전용 하네스가 있는 레포에서 이 스킬을 실행 → 금지. Claude/Grok은 전용 하네스 직접, Codex는 `claude -p` 위임.
