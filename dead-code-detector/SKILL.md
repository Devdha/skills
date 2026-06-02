---
name: dead-code-detector
description: |
  PR, branch, diff, or repository scope에서 죽은 코드, 고아 코드, 불필요한 코드, wet하게 반복된 코드,
  과도한 diff를 줄일 수 있는 삭제/축소 후보를 기능 단위로 찾고 검증한다.

  Triggers:
  - "dead code"
  - "deadcode"
  - "죽은 코드"
  - "불필요한 코드"
  - "diff 줄여줘"
  - "코드 줄일 수 있는지 봐줘"
  - "wet code"
  - "dead-code-detector"
---

# Dead Code Detector

## Purpose

이 스킬은 PR/diff를 기능 단위로 재구성한 뒤, 실제 기능 구현에 필요하지 않은 코드와 과도한 반복을 찾아 diff를 줄이는 데 집중한다.

목표는 “깔끔해 보이는 리팩토링”이 아니라 다음을 검증해서 줄이는 것이다.

- 죽은 코드: 더 이상 reachable하지 않은 함수, 컴포넌트, 타입, CSS, i18n key, asset, route, event
- 고아 코드: 삭제/변경된 기능과 연결되어 있었지만 함께 정리되지 않은 잔여물
- 과한 코드: 요구사항 대비 불필요한 fallback, wrapper, abstraction, generated artifact, duplicate path
- wet code: 같은 결정을 여러 곳에 복붙해서 diff와 유지보수 비용을 키우는 코드

## Core Rules

- 기본 동작은 read-only다. 사용자가 명시적으로 "고쳐줘", "삭제해줘", "반영해줘"라고 하지 않으면 코드를 수정하지 않는다.
- 삭제 후보는 검증 전에는 확정하지 않는다. `rg` 한 번으로 사용처가 안 보인다는 이유만으로 dead code라고 판단하지 않는다.
- diff 축소가 목적이어도 동작 변경은 목적이 아니다. 삭제가 behavior를 바꾸면 cleanup 후보가 아니라 기능 변경 후보로 분류한다.
- 테스트 전용 fixture, story/demo, migration history, 문서 예시, public API compatibility shim은 목적이 있으면 dead code로 보지 않는다.
- generated/dist/public artifact는 기본 broad search에서 제외하되, 배포 산출물 정합성을 볼 때만 명시적으로 포함한다.
- wet code 판단은 "중복처럼 보임"이 아니라 "같은 계약/상태/분기/렌더링이 여러 구현으로 흩어져 실제 유지보수 위험을 만든다"까지 확인한다.

## Phase 1. Scope Lock

먼저 분석 기준과 수정 권한을 고정한다.

- current branch 확인
- base branch는 기본적으로 `origin/develop`
- PR 번호가 있으면 PR title/body/base branch/changed files 확인
- PR base가 `develop`이 아니면 명시하고 그 기준을 따른다.
- `origin/develop...HEAD` 또는 PR merge-base 기준으로 diff 범위 확정
- merge-base SHA 기록
- 삭제 전 코드 내용은 `origin/develop:path`가 아니라 merge-base snapshot에서 확인
- 관련 AGENTS.md, package-level guide, docs 확인
- 분석-only인지, 삭제 계획까지인지, 실제 patch까지 원하는지 구분

## Phase 2. Diff Budget and Feature Decomposition

changed file 목록을 그대로 보지 말고 기능 단위로 묶는다.

각 기능에 대해 다음을 기록한다.

- 요구사항과 사용자 플로우
- 실제 entrypoint
- 새로 추가된 파일/함수/타입/CSS/i18n/asset/test
- 삭제된 파일/함수/타입/CSS/i18n/asset/test
- generated/dist/public artifact 포함 여부
- 기능 구현에 필요한 핵심 diff와 주변 정리성 diff
- diff budget 판단: `essential`, `supporting`, `questionable`, `unrelated`

예시:

```md
Feature A: translated PDF export
- Essential: export layout selection, PDF composition
- Supporting: readiness state, tests
- Questionable: unrelated share URL state mutation
- Unrelated: generated dist bundle
```

## Phase 3. Candidate Discovery

### 3.1 Deleted-Code Safety Check

삭제된 코드가 아직 필요한 경로에서 쓰이던 것은 아닌지 먼저 확인한다.

- 정적 import/export
- dynamic import
- string route/action/event key
- feature flag / experiment branch
- i18n key
- CSS selector / DOM selector
- analytics/history event
- cron/webhook/background job
- generated/public artifact

삭제가 안전하지 않으면 "delete regression risk"로 분류하고 cleanup 후보에서 제외한다.

### 3.2 Orphaned Dead Code Sweep

삭제/대체된 코드와 연결된 잔여물이 남았는지 찾는다.

확인 대상:

- helper function / hook / component
- constant / enum / type / interface
- CSS class / asset / SVG / image
- i18n key
- analytics/history/experiment key
- route/API helper
- DB query helper
- mock/test fixture
- barrel export/index export
- generated/public artifact hook

검증 방식:

- 삭제된 symbol/key/class/event에서 주변 symbol을 추출한다.
- exact `rg`로 사용처를 확인한다.
- import/export graph와 entrypoint reachability를 확인한다.
- string key 기반 사용처는 별도 검색한다.
- 삭제 전 내용은 merge-base snapshot에서 읽는다.
- 테스트/문서/fixture 목적이면 dead code가 아닌 보존 사유를 기록한다.

### 3.3 Added-Code Necessity Sweep

새로 추가된 코드가 요구사항에 실제로 필요한지 본다.

- 단일 호출 wrapper
- pass-through helper
- speculative abstraction
- duplicate fallback path
- debug-only state/log
- local-only UI control
- 같은 데이터를 여러 형태로 캐싱
- 같은 분기 조건을 여러 레이어에 반복
- generated artifact가 PR에 포함된 이유

판정:

- `keep`: 요구사항/계약에 필요
- `shrink`: 같은 동작을 더 작은 diff로 가능
- `delete`: reachable하지 않거나 기능과 무관
- `defer`: 삭제 가능성이 있지만 검증 비용이 높음

### 3.4 Wet Code Sweep

중복이 실제 위험인지 확인한다.

찾을 신호:

- 같은 상태 전환이 UI, export, API, 테스트에 각각 복붙됨
- 같은 selector/query/filter가 여러 helper에 분산됨
- 같은 timeout/retry/fallback 정책이 파일마다 다름
- 같은 domain decision을 다른 이름의 함수들이 반복함
- 테스트가 구현 세부사항 복붙으로 과도하게 커짐

합치기 전 확인:

- 중복이 의도적으로 다른 boundary를 가진 것은 아닌가
- abstraction이 오히려 coupling을 늘리지는 않는가
- 한 파일 안의 작은 중복보다 cross-module 계약 중복이 더 위험하지 않은가

## Phase 4. Static Tool Pass

가능하면 기존 도구를 사용하되, 도구 결과를 그대로 믿지 않는다.

- `rg` / import graph
- `tsc --noEmit`
- package lint
- `knip` 또는 repo-local dead-code tool
- `depcruise` / circular check
- package-specific test command

주의:

- `knip`은 설정상 빠지는 경로가 있을 수 있다. 결과는 후보 목록일 뿐이다.
- generated/dist noise는 기본 제외한다.
- shell에서 `[` `]` `(` `)` 등 glob 문자가 포함된 경로는 quote한다.

## Phase 5. Candidate Validation Gate

각 후보는 다음 중 하나로 검증해야 한다.

- call site와 entrypoint를 따라 reachable하지 않음을 확인
- 삭제 후 targeted test/typecheck/lint가 통과함을 확인
- runtime/browser에서 기능이 그대로 동작함을 확인
- merge-base와 HEAD diff 비교로 기능 대체가 완료됐음을 확인
- generated artifact는 실제 배포 경로와 비교

판정:

- `confirmed`: 삭제/축소 가능성이 높고 검증 완료
- `probable`: 강한 후보지만 runtime 또는 외부 경로 검증이 남음
- `question`: 제품/운영 의도가 필요
- `discarded`: 사용처나 보존 이유가 확인됨

## Phase 6. Cleanup Plan

사용자가 실제 반영을 원하면 작은 순서로 계획한다.

1. behavior lock: 기존 테스트 실행 또는 필요한 regression test 추가
2. dead/orphan deletion
3. generated/dist 제외 또는 재생성 정책 정리
4. wet code 축소
5. speculative abstraction 제거
6. targeted verification

원칙:

- 한 번에 한 냄새 범주만 고친다.
- 커밋이 필요하면 범주별로 작게 끊는다.
- public API, migration, feature flag, analytics event 삭제는 더 엄격하게 검증한다.

## Phase 7. Output Format

분석-only 결과:

```md
## Confirmed Deletions
- [confidence] 제목
  - 파일/라인:
  - 왜 불필요한지:
  - 검증:
  - 삭제/축소 제안:

## Shrink Candidates
- [confidence] 제목
  - 파일/라인:
  - 현재 diff 비용:
  - 더 작은 구조:
  - 검증 필요:

## Wet Code Candidates
- [confidence] 제목
  - 중복된 결정:
  - 실제 유지보수 위험:
  - 합칠 위치:
  - 합치지 말아야 할 위험:

## Discarded Candidates
- 후보:
  - 기각 이유:

## Diff Budget
- Essential:
- Supporting:
- Questionable:
- Unrelated:

## Verification
- 실행한 명령:
- 확인한 reachability:
- 못 확인한 gap:
```

실제 반영까지 한 경우:

```md
## Cleanup Applied
- 삭제/축소한 항목:
- 유지한 항목과 이유:

## Changed Files
- path: 변경 요약

## Verification
- tests/typecheck/lint/runtime:
- 남은 risk:
```

후보가 없으면:

```md
확정 삭제/축소 후보는 없습니다.

확인한 범위:
- Feature A: orphan sweep / added-code necessity / wet code 확인
- Feature B: generated artifact 정합성 확인

남은 한계:
- runtime은 확인하지 못함
```

## Parallelization Guidance

병렬화가 유효한 경우:

- 기능이 2개 이상으로 분리됨
- UI/server/test/generated artifact 축이 독립적임
- deletion orphan sweep과 added-code necessity sweep이 분리 가능함
- broad repo audit에서 package별 ownership이 분리됨

병렬화가 부적절한 경우:

- diff scope가 아직 고정되지 않음
- 같은 파일을 서로 다른 subagent가 삭제 후보로 동시에 판단해야 함
- 한 후보의 reachability 결론이 다른 후보의 입력이 됨

Leader는 후보를 반드시 통합하고, 중복/충돌/검증 부족을 정리한 뒤 삭제 가능성을 확정한다.
