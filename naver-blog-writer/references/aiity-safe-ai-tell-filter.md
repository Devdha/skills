# Aiity-Safe AI-Tell Filter

This reference adapts AI-tell filtering ideas from `epoko77-ai/im-not-ai` (MIT, inspected at commit `14aeb52`) for the user's `아잇티` Naver Blog voice. Use it as a conservative detector before the Aiity voice passes, not as a generic final humanizer.

## Priority

1. Preserve facts, place names, menu names, prices, dates, and user-supplied impressions.
2. Remove AI/review-site smell only from clear spans.
3. Preserve Aiity-specific emphasis such as `!!`, `!!!`, `ㅋㅋㅋ`, `ㅎㅎ`, `~`, and fitting emoji unless it is clearly excessive for the supplied draft.
4. After this filter, always run `Aiity Micro-Diction Pass` and `Aiity Emphasis and Punctuation Pass`.

## Do Not Do

- Do not auto-call an external `humanize-korean` skill after the Aiity passes.
- Do not flatten all punctuation into periods.
- Do not remove `ㅋㅋㅋ`, `ㅎㅎ`, `~`, `!!`, `!!!`, or emoji just because they look less polished.
- Do not rewrite the whole post when only a few spans smell generated.
- Do not turn the post into a neutral "natural Korean" article. The target is the user's casual blog voice.

## High-Value Filters

### Translationese and Formal AI Korean

Flag and rewrite only when the phrase sounds generated or report-like in context.

- `~에 대해/대해서`
- `~를 통해/통하여`
- `~에 있어/있어서`
- `~에 기반하여`, `~을 바탕으로`
- `~와 관련하여`, `~와 관련된`
- `~할 수 있다` when it weakens a simple judgment
- `가지고 있다`
- `~에 의해`
- `되어진다`, `지게 된다`
- `확인할 수 있었습니다`
- `느껴졌습니다`

### Review-Site Words

Prefer Aiity's everyday words over polished review nouns.

- Avoid: `전반적으로`, `전체적으로`, `인상적`, `만족도`, `구성`, `포인트`, `메리트`, `퀄리티`, `추천드립니다`, `방문해보시길`
- Prefer: `여기`, `이 집`, `이날`, `생각보다`, `꽤`, `살짝`, `진짜`, `그래도`, `쪽`, `괜찮았어요`, `좋았어요!!`

### Mechanical Structure Smell

Flag if the public body reads like a review template rather than a Naver post.

- `좋았던 점`, `아쉬웠던 점`, `추천 대상`, `총평`
- `첫째`, `둘째`, `셋째` unless the user asked for a guide
- `정리하면`, `결론적으로`, `요약하면`
- balanced pros/cons/recommendation blocks
- repeated section-summary sentences

### Rhythm Smell

Flag only enough to restore a human blog texture.

- every paragraph has similar length
- every positive sentence ends calmly with a period
- no visible emphasis in a casual food/place review
- repeated `~같아요` replacing actual judgment
- repeated `~느낌이었어요`

## Span-Grounded Rewrite Rules

- First mark the exact phrases that smell AI-generated.
- Rewrite only those phrases and the smallest surrounding clause needed for natural Korean.
- Keep the rest of the user's sentence if it already sounds like the user.
- If the rewrite would change more than about 25% of the final body, stop and redo from the Aiity reference style instead.

## Examples

- AI-like: `전반적으로 만족도가 높은 식사였습니다.`
- Aiity-safe: `저는 꽤 만족스럽게 먹었어요!!`

- AI-like: `다양한 메뉴 구성이 인상적입니다.`
- Aiity-safe: `메뉴가 생각보다 많더라구요. 같이 간 사람끼리 취향 갈려도 괜찮을 듯해요!`

- AI-like: `든든한 한 끼를 원하는 분들께 방문해보시길 추천드립니다.`
- Aiity-safe: `점심에 든든하게 먹고 싶을 때 괜찮을 듯해요~!!`

- AI-like: `사진을 통해 매장의 분위기를 확인할 수 있었습니다.`
- Aiity-safe: `사진 보시면 안쪽도 생각보다 넓죠?`
