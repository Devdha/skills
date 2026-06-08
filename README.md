# Codex Skills

Personal Codex skills packaged as a small multi-skill repository.

## Skills

- `review-assistant`: reviews PRs and diffs by decomposing features, validating candidate findings, and reporting confirmed issues.
- `dead-code-detector`: finds confirmed dead code, orphaned code, unnecessary additions, and wet code candidates.
- `screenshot-verification`: verifies frontend and rendered UI changes with real screenshots and concrete evidence.
- `wm`: guides non-interactive `wm` CLI usage for git worktree workflows.
- `naver-blog-writer`: drafts and revises Korean Naver Blog posts with natural tone, image slots, search-backed facts, title candidates, tags, and publish settings.

## Install

Copy the skill directories into your Codex skills directory:

```bash
cp -R review-assistant dead-code-detector screenshot-verification wm naver-blog-writer ~/.codex/skills/
```

Each skill keeps its instructions in `SKILL.md`. Optional `agents/openai.yaml` files provide UI metadata.
