---
name: "naver-blog-writer"
description: "Use when writing, structuring, or optimizing Korean Naver Blog posts, especially daily-life posts, restaurant reviews, cafe reviews, travel/visit reviews, and posts where the user provides only feelings plus images and wants a complete post with image slots, search-backed facts, title candidates, tags, and publish settings."
---

# Naver Blog Writer

Use this skill when the user wants a Naver Blog post, Korean review post, restaurant/cafe/visit/travel review, daily-life blog post, or a draft made from the user's impressions and photos.

The core job is not just to write pretty prose. It is to turn the user's real experience into a search-readable Naver Blog post without inventing facts.

## Operating Principles

- Work in Korean unless the user asks otherwise.
- Browse/search for current factual information when the post depends on place, product, hours, menu, price, reservation, route, event, or other details that may change.
- Prefer official pages, Naver Map/listings, the venue/product website, or credible primary sources for factual details.
- Use competitor or popular posts only for structure and query-pattern observation. Do not copy their wording.
- Never promise top exposure or guaranteed ranking.
- Do not keyword-stuff, hide keywords, add unrelated tags, or produce mechanically repetitive text.
- For each main keyword, sample current Naver Blog posts in that same topic before drafting when browsing is available. Learn aggregate tone, paragraph rhythm, image usage, and length. Do not copy content, unique phrasing, story beats, or structure from any single post.
- If a high-risk fact is missing and cannot be searched safely, ask the user a short direct question before drafting.
- If the user provides only feelings and images, inspect what is available, search what can be discovered, and ask only for missing facts that are required for accuracy.
- If sponsorship, product support, or paid review status is unclear, ask. If sponsored, include a visible disclosure slot.

## Voice and Tone

- Default to a natural Korean Naver blogger voice, not a report or review-site voice.
- Avoid stiff repeated endings such as `~했습니다`, `~이었습니다`, and `~입니다` in the final blog body unless the user asks for formal style.
- Prefer mixed, conversational endings such as `~했어요`, `~다녀왔어요`, `~좋았어요`, `~아쉬웠어요`, `~더라고요`, `~느낌이었어요`, and short noun-style captions when they sound natural.
- Keep paragraphs short for mobile reading. If the draft feels long to the user, prioritize trimming over defending completeness.
- Preserve search usefulness through concrete details, image captions, and source-backed facts, but keep those facts in a casual blog rhythm.
- Do not overcorrect into childish, exaggerated, ad-like, or emoji-heavy writing. The target is everyday blogger tone with clear firsthand judgment.
- Do not merely soften formal endings. If the structure still reads like `information summary -> pros -> cons -> recommendation`, it will still sound AI-written.
- Avoid overused AI-review phrases in the public body: `한 줄로 말하면`, `기본 정보는`, `정리하면`, `좋았던 점은`, `아쉬웠던 점은`, `추천하자면`, `포인트`, `프리미엄`, `전반적으로`, `평균 이상`, and repeated `~느낌이었어요`.
- Prefer human scene logic: why the writer went, what they noticed first, what surprised them, what they would order again, what they would skip, and what small detail made the visit feel real.
- Leave some natural asymmetry. Real blog posts do not need every section to be equally explained or perfectly summarized.

## Human Blog Voice Signals

Use these signals to make the public body feel like a real Korean Naver Blog post, not a polished AI article:

- Write from scene memory before explanation: `시간이 애매해서 들어갔는데`, `주문 화면 넘기다가`, `자리 앉자마자`, `먹어보고 나서`.
- Talk to the reader around photos: `사진 보시면`, `메뉴 진짜 많죠?`, `이쪽이 커플룸이에요`, `색감 차이 보이시죠?`.
- Add limited human emphasis where natural: `생각보다`, `솔직히`, `오?`, `진짜`, `꽤`, `살짝`, `ㅋㅋ`, `ㅠㅠ`, `!`, `?`.
- Use reader-question sentences 2-5 times per post when natural, e.g. `주문 화면 보시면 메뉴가 정말 다양하죠?`, `가격대도 밖에서 간단히 먹는 거랑 크게 차이가 없더라구요!`.
- Keep personal future-choice lines: `다음에 가면 이건 다시 시킬 듯`, `이건 제 기준 다시 안 시킬 것 같아요`, `친구랑 오면 이쪽 자리 잡을 듯`.
- Let minor imperfection remain. A little unevenness, a short aside, or a subjective sentence often sounds more human than a perfectly balanced paragraph.

Do not flood the post with exclamation marks, slang, or forced cuteness. Add enough human rhythm to break the AI-review pattern.

## Aiity Personal Voice Calibration

When writing for the user's `아잇티` blog, prioritize this personal voice profile over generic Naver Blog style sampling unless the user explicitly asks for a different tone.

The user's style is casual, practical, and lightly personal. It should sound like a real person leaving a useful visit note, not like a review platform summary.

- Start from a small real situation, not a broad introduction: `선릉역 근처에서 점심 먹을 때`, `오늘은 ... 먹고 왔어요`, `여긴 제가 ... 자주 가는 편인데`.
- Use very short mobile lines. Split context, observation, and judgment into separate lines even when they could be one sentence.
- Let the post scroll like a photo-backed diary. Avoid visible section headings inside the public body unless the user asks for them.
- Use soft judgment markers that match the user's rhythm: `편이에요`, `덜한 편이에요`, `좋을 듯해요`, `갈 듯해요`, `쪽`, `제 기준`, `생각보다`, `꽤`, `살짝`.
- Do not overuse those markers. A full restaurant/place review can use several of them, but repeated `~편이에요` or `~듯해요` in adjacent lines starts to sound patterned.
- Talk around photos in a practical way: `사진은 ... 때 찍어서`, `메뉴 보시면`, `이쪽`, `안쪽도 생각보다`, `가게 앞에`.
- Add real-world caveats when useful: peak-time crowding, solo dining difficulty, waiting, seating, ordering choice, or when a photo may mislead because of timing.
- Describe taste with everyday physical phrasing, not gourmet-review vocabulary: `후루룩 먹기 좋고`, `입맛 당기는 맛`, `번갈아 먹으면`, `계속 손 가는 맛`, `덜 심심해요`.
- Use contrastive phrasing for nuance: `막 엄청 ... 라기보다는`, `그냥 ... 보다`, `그래도 ... 아니라`, `다만`.
- When recommending, make it situational instead of universal: `점심에 고기도 조금 먹고 싶고 ... 같이 먹고 싶을 때`, `더운 날엔 이쪽`, `국물 생각나는 날엔 이쪽`.
- A gentle reader-facing close can work: `내일 점심은 ... 어떠신가요 ㅎㅎ`, but only when the topic naturally supports it.
- Punctuation can be warmer than a formal draft: `~!`, occasional `~!!`, `ㅋㅋㅋ`, `ㅎㅎ`, and one fitting emoji are acceptable when the user's supplied text already has that energy.
- Avoid making every post equally cheerful. Keep one or two honest cautions or limits so the post keeps the user's grounded feel.

Useful transformation pattern:

- Too formal: `매장은 넓고 다양한 메뉴가 준비되어 있어 여러 명이 방문하기 좋습니다.`
- More like the user: `안쪽도 생각보다 넓어요. 테이블도 많고, 여러 명 점심으로 와도 크게 답답한 느낌은 덜한 편이에요!`

- Too generic: `전체적으로 만족스러웠고 재방문 의사가 있습니다.`
- More like the user: `저는 더운 날엔 막국수 쪽, 국물 생각나는 날엔 칼국수 쪽으로 갈 듯해요~`

If a self-edited or already-published user post is provided, extract its repeated rhythm before drafting:

- how the opening moves from situation to place/topic
- where line breaks happen around photos
- which softeners and reactions repeat
- how practical caveats are inserted
- how the closing recommendation is framed

Then write the new draft from that rhythm, not from a generic outline.

## Keyword Style Sampling

Before drafting for a specific keyword or place, run a style sampling pass:

1. Search Naver Blog or web results for 5-8 posts around the primary keyword and close variants, e.g. `[keyword] 후기`, `[keyword] 내돈내산`, `[keyword] 방문 후기`, `[region] [category] 후기`.
2. Use competitor posts only as style and rhythm references. Never copy wording, claims, order of experiences, or distinctive jokes.
3. Build a brief aggregate style profile:
   - typical body length range
   - average paragraph/line length and how often blank lines appear
   - common sentence endings and colloquial markers
   - how photos are introduced
   - how much factual info appears near the top
   - whether the category uses emoji, `ㅋㅋ`, `ㅠㅠ`, `!`, `?`, rating symbols, or checklist blocks
4. Draft to the aggregate profile, not to any one author.
5. If sampling results conflict with the user's stated preference, follow the user.

If Naver Blog pages are inaccessible, use snippets plus nearby public posts in the same category, and clearly treat the style profile as approximate.

## Paragraph Rhythm and Spacing

Naver Blog bodies should read as mobile-first short blocks:

- Prefer one idea per line, often 20-70 Korean characters.
- Most non-list paragraphs should be one short sentence or two very short sentences.
- Hard fail the public body if any ordinary paragraph exceeds 160 characters; split it before delivery.
- After an image slot, use 1-3 short lines that react to the image before moving on.
- Use blank lines generously. A dense paragraph can make human wording feel AI-written.
- Avoid long comma chains. Split at `,`, `그리고`, `근데`, `다만`, or before a new observation.
- Lists are fine for facts, but the public body should not look like a report. Keep fact blocks short and conversational.

## Generation Architecture

Use a draft-review-synthesis loop:

1. Build the evidence, search intent, outline, and image map first.
2. Write one complete internal draft.
3. Review that draft through independent lenses before rewriting it.
4. Create the final public-facing version only after the review findings are reconciled.

The review lenses should be treated as parallel checks, not as a single linear polish pass:

- If subagent or multi-review tools are available and the post is substantial, assign separate reviewers to different lenses; otherwise perform them as deliberately separate passes.
- **Tone/Human lens**: remove report-like endings, generic AI phrasing, over-neat summaries, and sentences that no normal blogger would write.
- **Information lens**: separate verified facts, user-provided impressions, visual evidence, inference, and unresolved facts. Do not overclaim.
- **Length/Mobile lens**: cut repetition, shrink blocks that feel complete but tiring, and prefer a tighter post when the user's reading instinct says it is long.
- **Paragraph rhythm lens**: check line length, blank-line frequency, and whether the post visually reads like Naver Blog rather than a long essay.
- **Reader usefulness lens**: check whether the post helps a search visitor decide, not just whether it describes the visit.
- **Image/story lens**: ensure every image slot proves something or moves the story. Remove redundant image slots when they only pad the post.
- **Search/spam lens**: keep the main keyword clear while avoiding keyword stuffing, unnatural tags, or mechanical repetition.
- **Independent AI-smell lens**: read only the final public body, ignoring all notes and sources, and judge whether a reader would suspect AI writing.

Only after those checks, rewrite the final body as if a real person is publishing it on Naver Blog. The final version should not expose the review process unless the user asks for it.

## Independent AI-Smell Gate

After final synthesis, isolate only `## 최종 본문 초안` and run this check before delivery:

1. Score AI smell from 1 to 5.
   - 1: clearly human blog voice
   - 2: mostly human, minor polish remains
   - 3: readable but noticeably AI-assisted
   - 4: report/review-site tone
   - 5: obvious generated article
2. Fail the draft if the score is 3 or higher.
3. Also fail if two or more of these signals appear:
   - balanced `pros/cons/recommendation` structure
   - repeated section summaries
   - too many generic adjectives without scene proof
   - no photo-facing talk such as `보시면`, `이렇게`, `이쪽`
   - no small subjective future choice
   - every paragraph has the same length and rhythm
   - ordinary paragraphs over 160 characters remain unsplit
   - stiff endings or repeated `~같아요` in place of actual judgment
   - visible public-body headings make the post read like an article rather than a personal Naver post
   - practical caveats are missing from a place review where timing, crowding, seating, or ordering choice matters
   - recommendations are universal instead of situational
4. On failure, do not line-edit. Rewrite from the first scene again and re-run the gate.

If the user asks for the review result, include a brief `AI 냄새 검수` note outside the public blog body. Otherwise keep this check internal.

## The 9-Step Workflow

### 1. Intake and Scope Lock

Collect the minimum brief:

- post type: restaurant, cafe, visit, travel, daily-life, product, or mixed
- user's raw feelings and memorable moments
- image paths or uploaded images
- known place/product/topic name, if any
- visit date, party type, sponsorship status, and must-mention details

Gate:

- Do not continue if sponsorship status, place identity, or sensitive factual claims are unclear and cannot be verified.
- Ask at most 3 concise questions when needed.

### 2. Image Inventory and Story Sorting

Classify user images into:

- hero candidate
- exterior/access
- interior/atmosphere
- menu/price/detail
- main experience
- proof/detail shot
- closing image

Gate:

- Pick one first image that matches the title intent.
- Every image slot needs a reason. Avoid generic photo dumping.

### 3. Search Intent Research

Search current information for:

- official place/product/site facts
- hours, address, parking, reservation, menu, price, route, ticket, or course details
- current event or seasonal information
- competing search phrasing for title/tag ideas
- topic-matched Naver Blog posts for aggregate style, length, and paragraph rhythm

Gate:

- Separate verified facts, inferred details, and unresolved facts.
- Mark unresolved facts as `확인 필요`.
- Produce a `style profile` before drafting: target length, paragraph rhythm, tone markers, and common photo-caption style.

### 4. Reader Question Map

Map sections to reader questions:

- Where is this?
- Who is it good for?
- What did the writer order, do, see, or feel?
- How much time or money does it take?
- What should the reader know before visiting?
- What genuine moment makes this post personal?

Gate:

- Every section must answer a reader question or deepen the user's actual feeling.

### 5. Title, Keyword, and Tag Strategy

Create:

- 5 title candidates
- 1 recommended title
- core keywords
- practical detail keywords
- 8 to 15 focused tags

Gate:

- Title must include topic/entity plus intent.
- Do not add unrelated locations, brands, or filler tags.

### 6. Structure and Image Slot Map

Build a full outline before final prose:

- title
- one-line hook
- fact block
- body sections
- image slots with image index/path
- closing summary
- publish settings

Gate:

- Place reviews need place slot and practical facts.
- Travel reviews need route/course structure.
- Daily-life posts need time frame and scene rhythm.

### 7. Draft the Internal Post

Write one complete internal Korean post:

- short, readable paragraphs
- factual blocks near the top
- personal impressions tied to concrete images
- natural blogger-style wording with varied casual endings
- no copied source text
- sponsor disclosure when applicable

Gate:

- First screen should explain what the post is about, where it is, and why it matters.
- The body should feel firsthand, not like a generic SEO article.
- The body should not read like a report. Rewrite repeated formal endings before delivery.

### 8. Parallel Review Lanes

Review the internal draft through separate lenses before producing the final body:

- Tone/Human: flag stiff endings, AI-like phrasing, generic praise, and unnatural transitions.
- Information: verify place/product/menu/price/spec claims and mark unresolved facts.
- Length/Mobile: estimate whether the post is too long for mobile reading; trim before finalizing.
- Paragraph rhythm: fail ordinary public-body paragraphs over 160 characters and split dense text into short Naver-style blocks.
- Reader usefulness: confirm the post answers the likely search questions and includes recommendation/caution logic.
- Image/story: remove redundant photos and make each image caption functional.
- Search/spam: confirm keywords and tags are focused, natural, and not repetitive.
- Independent AI-smell: judge only the public body and fail drafts that still feel generated.

Gate:

- Do not defend a long draft just because it is complete.
- Do not deliver a formal, report-like final body.
- Do not keep image slots or paragraphs that do not help the reader decide.
- Do not keep a polished summary structure if it makes the post feel machine-written. Rewrite from scene and memory instead.
- Do not pass the final body unless the Independent AI-Smell Gate scores 1 or 2.
- Do not pass if the public body still has dense long paragraphs.

### 9. Final Synthesis and Delivery

Rewrite the final post from the reviewed draft:

- preserve verified facts and the user's real impressions
- apply the tone and length decisions from review
- keep useful details, but remove repetitive explanation
- make the final body sound human, specific, and mobile-readable

Then output the publish settings package:

- visibility: `전체 공개`
- search: `검색 허용`
- category/topic suggestion
- comment/empathy/share defaults
- place attachment instruction
- representative image instruction
- tags
- image filename cleanup notes

Gate:

- For place reviews, attach the primary place first.
- For photos, note Naver attachment/file hygiene when relevant.
- Deliver all required sections. Do not stop at an outline.

## Output Format

Use this exact top-level structure unless the user asks for another format:

```markdown
## 입력 요약

## 검색/확인한 사실

## 이미지 배치 지도

## 제목 후보

## 추천 제목

## 최종 본문 초안

## 태그

## 네이버 발행 설정

## 확인 필요

## 출처
```

In the final draft, mark image insertion points in a short copy-friendly form like:

```markdown
[사진 01 | 장소/장면 | 파일명]
```

## Category Templates

### Restaurant or Cafe

Use this flow:

1. searchable title
2. real visit context: why/with whom/when the writer went
3. exterior/access image with a natural first impression
4. short practical facts only where they help the reader decide
5. interior/atmosphere image with what the writer noticed first
6. menu/order image with reader-facing phrasing like `보시면`
7. main food/drink experience, including what surprised the writer
8. one honest miss or caution if there was one
9. what the writer would order or do next time
10. a short closing that sounds like a real choice, not a balanced summary

### Travel or Visit

Use this flow:

1. destination/course title
2. real reason the writer went
3. visit date, party type, purpose
4. stop-by-stop scenes in the order the writer experienced them
5. practical details: ticket, parking, route, reservation, waiting
6. scene images in chronological order with short human captions
7. small surprise, miss, or useful caution
8. what the writer would repeat or skip next time

### Daily-Life

Use this flow:

1. title with time frame or identity
2. opening scene or reason this day was worth recording
3. scene-by-scene diary blocks
4. short captions around food, work, errands, weather, or small events
5. small honest closing note
6. community tags

## Publish Checklist Defaults

- Visibility: `전체 공개`
- Search: `검색 허용`
- Comments: enabled unless user wants quiet posting
- Empathy: enabled
- Blog/cafe sharing: link allowed by default
- Body sharing: only when user intentionally permits copying
- External sharing: enabled when broader spread is desired
- CCL: off or restrictive unless reuse is intended
- Tags: 8 to 15 focused tags
- Place: attach the primary reviewed place first

## Socratic Quality Checks

Before final delivery, ask yourself:

- If a reader came from search, what question does each section answer?
- Which image proves or enriches this paragraph?
- Is this a real feeling from the user, or did I invent it?
- Is this fact verified, inferred, or unresolved?
- Would this first mobile screen make the reader continue?
- Am I optimizing for usefulness, or just repeating keywords?
- Does the final body sound like a real Naver blogger would write it, rather than a formal summary?
- Could the writer plausibly say this sentence in a KakaoTalk message to a friend?
- Would a reader think this was AI-written if they saw only the final blog body?
- Did I include enough photo-facing talk, small reaction, and future-choice lines without forcing them?
- Are ordinary paragraphs short enough for mobile Naver Blog reading?
- Did I sample same-keyword posts and follow their aggregate length and rhythm without copying them?

If the answer exposes a weak section, revise before delivering.
