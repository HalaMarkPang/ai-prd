# AI-PRD

English | [中文](README.md)

AI-PRD is an Agent Skill for writing, revising, and reviewing product requirements documents (PRDs). It defines the workflow, decision rules, and validation criteria an agent follows from raw requirement materials to an engineering-review draft. It works with agent clients that support skills or rule files, such as Claude Code, Codex, and Cursor.

The rule files are written in Chinese. Capable models follow them regardless of the conversation language, and the PRD is written in the language you request.

## Problems addressed

PRDs generated directly by AI tend to fail in five ways:

1. The full document is generated even when information is missing, so key conclusions rest on guesses.
2. Solution design starts before anyone checks whether the requirement holds.
3. Descriptions of the existing system have no evidence and do not match the implementation.
4. A fixed template is applied regardless of the team's format and review needs.
5. A request to review or partially edit results in a rewrite of the whole document or an online page.

## Mechanisms

| Problem | Mechanism | Rule file |
|---|---|---|
| Generating despite missing information | Intake: inventory the materials, rate eight readiness items as available / assumable / blocking, and ask one consolidated round of questions about blockers, each with a default | [intake.md](references/intake.md) |
| No check on whether the requirement holds | Up-front assessment of validity, necessity, alternatives, and scope; if it does not hold, align with the user first | [prd-generation.md](references/prd-generation.md) |
| Unsupported claims about the current system | Code access levels (L0–L4) and evidence rules: claims must be verified to `file:line`; anything unverified is listed as open | [code-evidence.md](references/code-evidence.md) |
| Format mismatch | Format priority: the user's template or sample comes first; otherwise the skill's PRD template, trimmed to the requirement's complexity | [prd-template.md](templates/prd-template.md) |
| Unauthorized edits | Task modes and write permissions: full draft, concise review draft, partial revision, read-only review; read-only tasks never become writes | [review-and-notification-rules.md](references/review-and-notification-rules.md) |

## Workflow

```text
Determine task mode and write permissions
→ Intake
→ Requirement assessment (new) / change impact check (revision)
→ Verify the current state as needed
→ Change baseline and business model
→ Rules, data, interaction, and dependencies
→ Draft in the user's format, or produce a patch
→ Validate and deliver within the granted permissions
```

A read-only review ends after feedback is given; no file is created and nothing is written.

## Installation

AI-PRD uses `SKILL.md` as its entry point, which references the rule files under `references/` and `templates/`. Install the whole directory.

| Client | Installation |
|---|---|
| Claude Code | `git clone https://github.com/HalaMarkPang/ai-prd ~/.claude/skills/ai-prd` |
| Cursor | Clone into `~/.cursor/skills/ai-prd` or a project's `.cursor/skills/ai-prd` |
| Codex | Clone into `~/.codex/skills/ai-prd`, or reference `SKILL.md` from `AGENTS.md` and keep the directory readable |
| Other agents | Have the agent read `SKILL.md` and the files it references; `SKILL.md` alone does not load the full rule set |

## Usage

State the task in the conversation. The examples below correspond to the four task modes:

| Task mode | Example |
|---|---|
| Full draft | Use AI-PRD to write a PRD for an overdue-task reminder. The code is in `~/code/my-product`. |
| Concise review draft | Turn this BRD into an engineering-review draft using the attached team template. |
| Partial revision | Revise only the "exceptions and failure handling" section of this PRD and leave the rest unchanged. |
| Read-only review | Review this PRD and report issues in the conversation only. Do not edit the document. |

The agent's first reply contains its understanding of the requirement, a readiness table, and open questions. Answer them, or reply "continue with defaults"; the defaults it adopts are then listed as open questions in the PRD.

See the [overdue-task reminder example](examples/overdue-task-reminder/README.md) (in Chinese). The product, data, and code in the example are fictional.

## Code access

For changes to an existing system, the agent confirms code access during intake and records the level used in the PRD:

| Level | Condition | Basis for current-state claims |
|---|---|---|
| L0 | No code provided | None; all such claims are marked as open |
| L1 | Snippets or API docs provided | Only the provided material |
| L2 | Repositories cloned locally | Read-only local search, verified to `file:line` |
| L3 | Multiple repositories, complex call chains | Located with a code graph, then verified in source |
| L4 | Repositories on GitHub, GitLab, or similar | Read-only platform APIs; recall is usually narrower than local search |

Credentials stay in the user's local environment variables and are never recorded in the conversation, the PRD, or logs.

## Output conventions

- The PRD body targets engineering review. Assessment, fact extraction, requirement models, and review reports are internal artifacts and are excluded from the body by default.
- Key conclusions are labeled with their source: user statement, original material, existing PRD, code, reasoned inference, or open question.
- The PRD describes product-side data needs, business meaning, and error handling. It does not define API fields, error codes, timeouts, caching, or other technical parameters.
- Metrics without a complete definition (scope, data source, time window, attribution) are treated as post-launch observations, not committed targets.
- Functional correctness, operational quality, and business impact are validated separately; tests that were not run are not reported as passed.

## Contributing

Issues and pull requests are welcome. When changing a rule, update the [check cases](references/review-regression-cases.md) with the expected and prohibited behaviors.

## License

[MIT](LICENSE)
