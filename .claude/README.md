# .claude/ — Claude Code Project Configuration

This folder configures Claude Code's behavior for the **AgentHive** project.

## Files

- `settings.json` — Project-scoped permissions, enabled skills, and env defaults. Auto-loaded by Claude Code.
- `README.md` — This file.

## Enabled Skills

| Skill | When it kicks in |
|---|---|
| `ui-ux-pro-max` | Any UI/component/page work |
| `frontend-design:frontend-design` | Building polished frontend interfaces |
| `claude-api` | Touching Anthropic SDK code (agents/) |
| `fetch-library-docs` | Looking up React/Next/FastAPI/Tailwind APIs |
| `simplify` | Reviewing changes for reuse & quality |
| `less-permission-prompts` | Trimming repeated permission prompts |
| `code-review:code-review` | Reviewing PRs / pending changes |
| `security-review` | Pre-merge security pass |
| `stripe:stripe-best-practices` | Any Stripe integration change |
| `stripe:explain-error` | Debugging Stripe error codes |
| `stripe:test-cards` | Local Stripe testing |
| `theme-factory` | Theming landing / dashboard surfaces |
| `doc-coauthoring` | Writing docs / proposals |
| `skill-creator` | Adding a new project-local skill |
| `interview` | Clarifying ambiguous user intent |

## Permissions

`settings.json` pre-approves common safe commands (npm/pnpm/pip/uvicorn/git read ops, scoped Read/Write/Edit) and denies destructive ones (`rm -rf`, `git push --force`, `git reset --hard`) plus reads of any `.env` or files matching `*secret*` / `*credentials*`.

## Adjusting Permissions

Use the `less-permission-prompts` skill or `update-config` skill to add new allow-list entries — don't hand-edit unless you're confident in the JSON schema.

## See Also

- `../CLAUDE.md` — Project-wide engineering rules.
- `../AGENTS.md` — Spec for the 6 in-product AI agents.
- `../project_plan.md` — Authoritative build plan.
