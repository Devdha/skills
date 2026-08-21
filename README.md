# Codex Skills

Personal Codex skills packaged as a small multi-skill repository.

## Skills

- `review-assistant`: reviews PRs and diffs by decomposing features, validating candidate findings, and reporting confirmed issues.
- `dead-code-detector`: finds confirmed dead code, orphaned code, unnecessary additions, and wet code candidates.
- `screenshot-verification`: verifies frontend and rendered UI changes with real screenshots and concrete evidence.
- `video-verification`: records and verifies time-based UI flows with video, key frames, runtime checks, and concrete evidence.
- `wm`: guides non-interactive `wm` CLI usage for git worktree workflows.
- `naver-blog-writer`: drafts and revises Korean Naver Blog posts with natural tone, image slots, search-backed facts, title candidates, tags, and publish settings.
- `zen`: autonomous task pipeline — design (with Notion planning log) → implementation → PR + review harness → convergence → final verification (screen/logic).
- `zen-brainstorm`: interactive design interview (trigger, bug repro / customer value, alternatives) producing an autonomous-execution contract (`design.md`) and a Notion planning page.
- `zen-verification`: final verification with real-environment evidence — browser screenshots (PC + mobile viewports, S3 URLs) or real API calls.
- `zen-review-harness`: generic multi-perspective PR review harness with convergent re-review (fallback when no project-specific harness exists).

## Install

Copy the skill directories into your Codex skills directory:

```bash
cp -R review-assistant dead-code-detector screenshot-verification video-verification wm naver-blog-writer ~/.codex/skills/
```

The `zen*` skills are canonically hosted at `~/.claude/skills/` and shared across agents (Claude Code, Codex, Grok) via pointer files in `~/.codex/skills/`, `~/.grok/skills/`, and `~/.agents/skills/` whose frontmatter mirrors the canonical files. To install the full bodies as canonical:

```bash
cp -R zen zen-brainstorm zen-verification zen-review-harness ~/.claude/skills/
```

Each skill keeps its instructions in `SKILL.md`. Optional `agents/openai.yaml` files provide UI metadata.
