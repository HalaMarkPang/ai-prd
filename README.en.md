# AI-PRD

English | [中文](README.md)

AI-PRD is an Agent Skill for writing, revising, and reviewing product requirements documents (PRDs). It defines the workflow, decision rules, and validation criteria that an agent follows from raw requirement materials to an engineering-review draft. It works with agent clients that support skills or rule files, such as Claude Code, Codex, and Cursor.

The rule files are written in Chinese. Capable models follow them regardless of the conversation language, and the PRD is written in the language the user requests.

## Problems addressed

PRDs generated directly by AI typically fail in five ways:

1. The full document is generated despite missing information, so key conclusions rest on assumptions.
2. Solution design begins before the requirement itself has been validated.
3. Descriptions of the existing system lack evidence and diverge from the implementation.
4. A fixed template is applied regardless of the team's format and review needs.
5. When asked only to review or partially edit, the agent rewrites the whole document or modifies an online page.

## Mechanisms

| Problem | Mechanism | Rule file |
|---|---|---|
| Generating despite missing information | Readiness intake: inventory the materials, rate eight items as available / assumable / blocking, and ask one consolidated round of questions about blockers, each with a default | [intake.md](references/intake.md) |
| No validation of the requirement | Up-front assessment of validity, necessity, alternatives, and scope; if the requirement does not hold, align with the user first | [prd-generation.md](references/prd-generation.md) |
| Unsupported claims about the current system | Code access levels (L0–L4) and evidence discipline: claims must be verified to `file:line`; unverified claims are listed as open | [code-evidence.md](references/code-evidence.md) |
| Format mismatch | Format priority: the user's template or sample comes first; otherwise the skill's PRD template, trimmed to the requirement's complexity | [prd-template.md](templates/prd-template.md) |
| Unauthorized edits | Task modes and write permissions: full draft, concise review draft, partial revision, read-only review; without an explicit user request, read-only tasks never become writes | [prd-generation.md](references/prd-generation.md#执行模式), [review-and-notification-rules.md](references/review-and-notification-rules.md) |

## Workflow

```text
Determine task mode and write permissions
→ Readiness intake
→ Requirement assessment (new) / change impact check (revision)
→ Verify the current state as needed
→ Change baseline and business model
→ Rules, data, interaction, and dependencies
→ Draft in the user's format, or produce a revision
→ Review, validate, and deliver within the granted permissions
```

## Process constraints

1. **Intake**: Questions are asked in a single round, at most five, each stating why it is asked and what default applies. No questions are asked when the materials are complete.
2. **Read-only review**: The task ends once feedback is given; no file is created and nothing is written. If the user then explicitly requests edits, the task switches to partial revision and the write target is confirmed before writing.
3. **Revision limit**: If the internal review score is below 85, the agent revises at most twice. If the draft still falls short, the agent stops revising on its own, lists the unmet dimensions and gaps, and the user decides whether to supply more material, accept the current version, or hand the work over for manual completion.
4. **Unattended runs**: In unattended environments such as CI, the agent asks no questions. Blockers and default assumptions are recorded in the generation note at the top of the PRD and in its open questions, marked as unconfirmed by the user.

## Installation

AI-PRD uses `SKILL.md` as its entry point, which references the rule files under `references/` and `templates/`. The whole directory must be installed.

| Client | Installation |
|---|---|
| Claude Code | `git clone https://github.com/HalaMarkPang/ai-prd ~/.claude/skills/ai-prd` |
| Cursor | Clone into `~/.cursor/skills/ai-prd` or a project's `.cursor/skills/ai-prd` |
| Codex | Clone into `~/.codex/skills/ai-prd`, or reference `SKILL.md` from `AGENTS.md` and ensure the whole directory is readable |
| Other agents | Instruct the agent to read `SKILL.md` and every file it references; `SKILL.md` alone does not load the full rule set |

## Usage

State the task and provide the relevant materials in the conversation. The examples below correspond to the four task modes:

| Task mode | Example |
|---|---|
| Full draft | Use AI-PRD to write a PRD for an overdue-task reminder. The code is in `~/code/my-product`. |
| Concise review draft | Turn this BRD into an engineering-review draft using the attached team template. |
| Partial revision | Revise only the "exceptions and failure handling" section of this PRD and leave the rest unchanged. |
| Read-only review | Review this PRD and report issues in the conversation only. Do not edit the document. |

The agent's first reply contains its understanding of the requirement, a readiness table, and open questions. The user may answer each question or reply "continue with defaults". In the latter case, the agent records the defaults it adopts in the generation note at the top of the PRD and in its open questions.

See the [overdue-task reminder example](examples/overdue-task-reminder/README.md) (in Chinese). The product, data, and code in the example are fictional.

## Code access

For changes to an existing system, the agent confirms code access during intake and records the level used in the PRD:

| Level | Condition | Basis for current-state claims |
|---|---|---|
| L0 | No code provided | None; all such claims are marked as open |
| L1 | Snippets or API documentation provided | Only the provided material |
| L2 | Repositories available locally | Read-only local search, verified to `file:line` |
| L3 | Multiple repositories, complex call chains | Located with a code graph, then verified in source |
| L4 | Repositories on GitHub, GitLab, or similar | Read-only platform APIs; recall is usually narrower than local search |

Public repositories require no credentials. Credentials for private repositories are kept only in the user's local environment variables and are never recorded in the conversation, the PRD, or logs.

## Output conventions

- The PRD body targets engineering review. Intake results, requirement assessment, source facts and constraints, requirement models, review reports, and scores are internal artifacts and are excluded from the body by default.
- Key conclusions are labeled with their source: explicit user statement, original material, existing PRD, code or system state, reasoned inference, or open question.
- The PRD describes product-side data needs, business meaning, and error-handling requirements. It does not define API fields, error codes, timeouts, caching, or other technical parameters.
- Metrics whose scope, data source, time window, or attribution is incomplete are treated as post-launch observations, not committed targets.
- Functional correctness, operational quality, and business impact are validated separately; tests that were not run are not reported as passed.

## Directory structure

```text
ai-prd/
├── SKILL.md                              # Entry: task modes, readiness intake, main flow
├── references/
│   ├── intake.md                         # Readiness intake
│   ├── prd-generation.md                 # Execution modes, constraints, steps, validation
│   ├── requirement-method.md             # Requirement analysis and expression methods
│   ├── review-and-notification-rules.md  # Draft review, async events, rate limiting
│   ├── review-regression-cases.md        # Check cases
│   ├── doc-sources.md                    # Reading materials and completeness checks
│   └── code-evidence.md                  # Code access levels and evidence discipline
├── templates/
│   ├── prd-template.md                   # PRD template
│   └── evidence-template.md              # Code evidence template
└── examples/
    └── overdue-task-reminder/            # Fictional example
```

## Contributing

Issues and pull requests are welcome. When changing a rule, update the [check cases](references/review-regression-cases.md) with the expected and prohibited behaviors.

## License

[MIT](LICENSE)
