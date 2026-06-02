---
name: review-assistant
description: |
  PR/diff를 기능 단위로 재구성하고, 기능별 리뷰 축을 병렬로 펼쳐 리뷰 포인트를 발굴한 뒤,
  각 포인트를 코드/테스트/런타임 증거로 검증해 확정 finding을 만든다.

  Triggers:
  - "리뷰해줘"
  - "PR 리뷰"
  - "코드리뷰"
  - "전반적으로 리뷰 포인트 잡아줘"
  - "기능 단위로 리뷰해줘"
  - "review assistant"
---

# Review Assistant

## Purpose

이 스킬은 단순히 변경 파일을 훑는 리뷰가 아니라, PR/diff를 기능 단위로 재구성한 뒤 각 기능에 대해 correctness, performance, concurrency, synchronization, consistency, tests, rules 등을 병렬적으로 검토해 리뷰 포인트를 발굴한다.

각 리뷰 포인트는 deep review 이후 반드시 검증 단계를 거쳐야 한다. 코드만 보고 추측한 내용은 finding으로 확정하지 않는다.

## Core Rules

- 리뷰 요청은 기본적으로 read-only다. 사용자가 명시적으로 "고쳐줘", "반영해줘"라고 하지 않으면 코드를 수정하지 않는다.
- 사용자가 명시적으로 "PR에 남겨줘", "GitHub에 코멘트 달아줘"라고 하지 않으면 외부에 리뷰를 게시하지 않는다.
- changed file 목록을 그대로 리뷰하지 않는다. 먼저 기능 단위로 재구성한다.
- finding은 검증 가능한 문제만 올린다. 취향, 추측, 약한 가능성은 question 또는 note로 낮춘다.
- no findings일 때도 어떤 기능/축을 확인했는지 남긴다.

## Phase 1. Scope Lock

리뷰 기준을 먼저 고정한다.

- current branch 확인
- base branch는 기본적으로 `origin/develop`으로 둔다.
- PR 번호가 있으면 PR title/body/base branch/changed files 확인
- PR의 base가 `develop`이 아닌 경우에만 명시적으로 기록하고 그 기준을 따른다.
- `origin/develop...HEAD` 또는 PR merge-base 기준으로 diff 범위 확정
- merge-base SHA를 기록한다.
- 삭제된 파일의 이전 내용, 삭제 전 import, 삭제 전 symbol은 `origin/develop:path`가 아니라 merge-base snapshot에서 확인한다. PR이 이미 merge되었거나 `origin/develop`이 이동한 경우 `origin/develop:path`는 삭제 전 파일을 보장하지 않는다.
- 관련 AGENTS.md, package-level guide, docs 확인
- 리뷰-only인지, 게시까지 원하는지, 수정까지 원하는지 구분

## Phase 2. Feature Decomposition

파일 단위가 아니라 기능 단위로 변경을 묶는다.

각 기능마다 다음을 적는다:

- 사용자 경로
- 서버/API 경로
- 상태/store/storage 경로
- DB/transaction 경로
- analytics/history/experiment 경로
- observability/logging/metrics 경로
- 배포/generated artifact 경로
- 테스트 경로

예시:

```md
Feature A: PDF translation font size setting
- UI: translation font size control
- Storage: translationFontSize / translationFontSizeEnabled
- Sync: fontSizeChanged listener
- Risk: global font size path와 분리 여부
```

## Phase 3. Change-Type Impact Pass

기능별 matrix를 돌리기 전에 변경 유형별 영향도를 먼저 본다.

### 3.1 삭제 변경: Too Much / Too Little 양방향 검토

삭제 리뷰는 두 방향을 모두 확인한다.

#### Deleted Code Safety Check

삭제된 코드가 아직 필요한 경로에서 쓰이던 것은 아닌지 확인한다.

- 정적 import/export
- 동적 import
- string-based route/action/event key
- feature flag / experiment branch
- i18n key
- CSS selector / DOM selector
- background job / cron / webhook
- generated bundle / public artifact

#### Orphaned Dead Code Sweep

삭제된 코드와 연결되어 있었지만 함께 삭제되지 않아 이제 아무 곳에서도 쓰이지 않는 고아 코드가 남았는지 확인한다.

확인 대상:

- helper function
- component
- hook/context
- constant/type
- CSS class
- asset
- i18n key
- analytics/history event
- experiment branch
- route/API helper
- DB query helper
- mock/test fixture
- barrel export/index export
- generated/public artifact

검토 방식:

- 삭제된 symbol/key/class/event 이름에서 연결된 주변 symbol을 추출한다.
- `rg`로 exact usage를 찾는다.
- import/export graph를 확인한다.
- string key 기반 사용처는 별도 검색한다.
- 남은 코드가 실제 entrypoint에서 reachable한지 확인한다.
- 삭제 전 코드 내용은 merge-base snapshot에서 읽는다.
- reachable하지 않으면 orphaned dead code finding 후보로 올린다.
- 테스트 전용 fixture나 문서 예시는 목적이 있으면 dead code로 보지 않는다.

### 3.2 추가 변경: Connected Feature Impact

새 코드가 기존 기능에 끼어드는 지점을 찾는다.

- shared util
- global state
- middleware
- provider
- listener
- API client
- DB write
- analytics event
- experiment assignment
- browser/extension runtime hook

확인할 영향:

- 기존 사용자 플로우 변화
- 기존 상태 동기화 변화
- 성능/비용 증가
- race window
- 권한/보안 경계 변화
- analytics/history 오염
- 테스트 커버리지 누락
- web/chrome/server/mobile 간 영향

## Phase 4. Parallel Review Point Mapping

기능별로 아래 축을 병렬로 검토한다. 가능하면 독립 축은 subagent로 나눠 동시에 수행한다.

각 축은 반드시 다음 중 하나를 남긴다:

- finding 후보
- question 후보
- no issue 판단 근거
- 해당 없음 사유

Review axes:

- Correctness: 기능 요구사항이 실제로 만족되는가
- Performance/Cost: 불필요한 반복 호출, 렌더링, DB/API 비용이 생기는가
- Concurrency: race, 중복 실행, transaction boundary 문제가 있는가
- Synchronization: UI/store/storage/server 상태가 어긋날 수 있는가
- Data Consistency: DB 상태, cache, billing, subscription, generated data가 정합적인가
- Error/Recovery: 실패, rollback, retry, partial success 경로가 안전한가
- Security/Auth: 권한, 직접 접근, browser-direct path, presigned URL, tenant boundary가 안전한가
- Observability: 로그/metric이 너무 시끄럽거나 cardinality가 높거나 debug value를 잃지 않는가
- Tests: 핵심 경로, regression, edge case가 테스트로 잠겨 있는가
- Project Rules: AGENTS.md, docs, i18n, analytics, experiment, coding style을 지키는가
- Deployment Artifacts: generated bundle/public artifact가 실제 배포 경로와 일치하는가

## Phase 5. Review Point Merge

병렬 결과를 합친다.

- 중복 후보 병합
- 같은 root cause를 하나로 묶기
- blocking / non-blocking / question / no-issue로 분류
- 증거가 약한 후보는 finding으로 올리지 않기
- 기능별로 빠진 축이 없는지 확인

## Phase 6. Deep Review Per Point

각 후보 포인트를 세부 검토한다.

- 관련 call site 추적
- caller/callee 계약 확인
- 변경 전후 diff 비교
- 상태 전파 순서 확인
- async ordering 확인
- transaction/rollback boundary 확인
- feature flag/experiment 분기 확인
- 브라우저 직접 경로와 서버 경로를 둘 다 확인
- 테스트가 실제 위험을 커버하는지 확인
- 문서/룰과 대조

## Phase 7. Validation Gate

deep review 후 바로 finding으로 내지 않는다. 각 후보는 검증을 통과해야 한다.

검증 방법:

- 코드 경로 직접 확인
- `rg`/import graph로 reachability 확인
- shell에서 `[` `]` `(` `)` 등 glob/meta 문자가 포함된 경로는 반드시 quote한다.
- broad `rg`는 generated bundle이나 dist가 신호를 덮지 않도록 기본적으로 제외한다. 단, Deployment Artifacts 축을 검증할 때는 generated/public artifact를 명시적으로 포함한다.
- targeted test 실행
- typecheck/lint/static check 실행
- 필요 시 local server 실행
- 이미 떠 있는 local server를 재사용할 때는 프로세스 cwd/branch가 리뷰 대상 작업트리와 일치하는지 확인한다. 다른 worktree의 서버이거나 응답이 없으면 runtime 검증 증거로 쓰지 않는다.
- local server가 포트 충돌, 무응답, 환경 변수 문제로 확인되지 않으면 다른 포트를 쓰거나 verification gap으로 명시한다.
- 필요 시 browser로 실제 재현
- DB query 또는 migration 상태 확인
- analytics/log/metric payload 확인
- generated artifact 실제 배포 경로 확인

판정:

- 검증됨: finding으로 승격
- 증거 약함: question으로 낮춤
- 반박됨: no issue로 폐기
- 실행 불가: verification gap으로 명시

## Phase 8. Finding Admission Rule

finding은 아래를 포함해야 한다.

- severity: P0/P1/P2/P3
- file/line
- 문제가 되는 기능 경로
- 실제로 깨지는 이유
- 검증 증거
- 재현 또는 확인 방법

Severity 기준:

- P0: 즉시 장애, 데이터 손상, 보안 사고, 결제/권한 치명 오류
- P1: 주요 기능 회귀, 실제 사용자 플로우 중단, 운영상 큰 문제
- P2: 제한된 조건의 버그, 누락된 edge case, 유지보수/관측성 문제
- P3: non-blocking 개선, 명확성, 작은 정리, 취향에 가까운 제안

형식:

```md
[P1] 제목
파일/라인:
문제:
왜 실제 문제인지:
검증:
제안:
```

## Phase 9. Output Format

출력은 findings를 먼저 둔다. 각 finding에는 심각도를 반드시 붙인다.

```md
## Findings
- [P1] 제목
  - 파일/라인:
  - 문제:
  - 왜 실제 문제인지:
  - 검증:
  - 제안:

## Questions
- finding으로 확정하기엔 증거가 부족하지만 확인이 필요한 항목

## Reviewed Matrix
- 기능별로 어떤 축을 봤는지 요약

## Discarded Candidates
- 검토했지만 문제 아니라고 판단한 후보와 이유

## Verification
- 실행한 명령, 브라우저 검증, 코드 확인 범위

## Gaps
- 못 돌린 테스트, 접근 불가한 환경, 남은 불확실성
```

finding이 없으면 이렇게 답한다:

```md
확정 finding은 없습니다.

확인한 범위:
- Feature A: correctness/performance/sync/tests 확인
- Feature B: deletion orphan sweep 확인
- Feature C: analytics/observability 확인

남은 한계:
- local runtime은 띄우지 못함
- external API 실제 응답은 mock 기준으로만 확인
```

## Parallelization Guidance

병렬화가 유효한 경우:

- 기능이 2개 이상으로 분리될 때
- UI/analytics/server/test 축이 독립적일 때
- 삭제 dead-code sweep과 runtime behavior review가 분리될 때
- security/observability/performance 축이 독립적으로 검토 가능할 때

병렬화가 부적절한 경우:

- 같은 파일을 같은 관점으로 중복 검토하는 경우
- merge-base/diff 범위가 아직 고정되지 않은 경우
- 한 축의 결론이 다른 축의 입력이 되는 경우

Leader는 병렬 결과를 반드시 통합하고, 중복/충돌/증거 부족을 정리한 뒤 finding을 확정한다.
