# AI-PRD

English | [中文](README.md)

AI-PRD is an Agent Skill for generating, revising, and reviewing product requirements documents (PRDs). It specifies the workflow, decision rules, and validation criteria an agent follows when turning requirement materials into a draft for engineering review. It applies to agent clients that support skills or rule files, such as Claude Code, Codex, and Cursor.

The rule files are written in Chinese. Models with adequate Chinese comprehension apply them regardless of the conversation language, and the PRD is produced in the language the user specifies.

## Problems addressed

PRDs generated directly by AI commonly exhibit five deficiencies:

1. The full document is generated despite missing information, and key conclusions rest on assumptions.
2. Solution design begins before the requirement itself has been assessed.
3. Descriptions of the existing system lack evidence and diverge from the actual implementation.
4. A fixed template is applied without regard to the team's established format and review needs.
5. When the user requests only a review or a partial edit, the agent nonetheless rewrites the whole document or modifies an online document.

## Mechanisms

| Problem | Mechanism | Rule file |
|---|---|---|
| Generation despite missing information | Readiness intake: the agent inventories the materials, rates eight checks as available, assumable, or blocking, and raises questions on blocking items in a single round, each with a default | [intake.md](references/intake.md) |
| Requirement not assessed | Up-front requirement assessment: a conclusion is reached on validity, necessity, alternatives, and scope; if the requirement is not justified, the agent aligns with the user before proceeding | [prd-generation.md](references/prd-generation.md) |
| Unsupported claims about the existing system | Code access levels (L0–L4) and evidence discipline: current-state claims in the PRD must be verified to `file:line`; claims that cannot be verified are listed as open questions | [code-evidence.md](references/code-evidence.md) |
| Format inconsistent with team requirements | Format priority: the user's template or sample takes precedence; otherwise the skill's PRD template is used and trimmed to the complexity of the requirement | [prd-template.md](templates/prd-template.md) |
| Unauthorized modification | Task modes and write authorization: full generation, concise review draft, partial revision, and read-only review are distinguished; without an explicit user request, a read-only task is never converted into a write | [prd-generation.md](references/prd-generation.md#执行模式), [review-and-notification-rules.md](references/review-and-notification-rules.md) |

## Workflow

```text
Determine the task mode and write authorization
→ Readiness intake
→ Requirement assessment (new requirements) / change impact check (partial revision)
→ Verify the current state as needed
→ Establish the change baseline and business model
→ Define rules, data, interaction, and dependencies
→ Produce the document body or revision in the user's format
→ Review and validate; deliver within the authorized scope
```

## Process constraints

1. **Intake**: Questions are raised in a single round, limited to five, each stating its reason and its default. No questions are raised when the materials are complete.
2. **Read-only review**: The task ends once feedback is delivered; no file is created and no document is modified. If the user subsequently requests changes explicitly, the task becomes a partial revision, and the write target is confirmed before any write.
3. **Revision limit**: If the internal review score is below 85, the agent performs at most two revision rounds. If the draft still does not meet the threshold, the agent stops revising, lists the unmet dimensions and gaps, and the user decides whether to supply additional material, accept the current version, or assign the work for manual completion.
4. **Unattended runs**: In unattended environments such as CI, the agent raises no questions. Blocking items and default assumptions are recorded in the generation note at the top of the PRD and in its open questions, and are marked as not confirmed by the user.

## Installation

AI-PRD uses `SKILL.md` as its entry point, which references the rule files under `references/` and `templates/`. The complete directory must be installed.

| Client | Installation |
|---|---|
| Claude Code | `git clone https://github.com/HalaMarkPang/ai-prd ~/.claude/skills/ai-prd` |
| Cursor | Clone into `~/.cursor/skills/ai-prd` or into a project's `.cursor/skills/ai-prd` |
| Codex | Clone into `~/.codex/skills/ai-prd`, or reference `SKILL.md` from `AGENTS.md` and ensure that the complete directory is readable |
| Other agents | Instruct the agent to read `SKILL.md` and every file it references; `SKILL.md` alone does not load the complete rule set |

## Usage

State the objective of the task in the conversation and provide the relevant materials. The examples below correspond to the four task modes:

| Task mode | Example request |
|---|---|
| Full generation | Use AI-PRD to write a PRD for an overdue-task reminder. The code is in `~/code/my-product`. |
| Concise review draft | Turn this BRD into an engineering review draft, structured according to the attached team template. |
| Partial revision | Revise only the "Exceptions and failure handling" section of this PRD and keep all other content unchanged. |
| Read-only review | Review this PRD and report issues in the conversation only. Do not modify the document. |

The agent's first reply contains its understanding of the requirement, a readiness table, and the questions to be confirmed. The user may answer each question or reply "continue with defaults". In the latter case, the agent records the adopted default assumptions in the generation note at the top of the PRD and in its open questions.

For a complete walkthrough, see the [overdue-task reminder example](examples/overdue-task-reminder/README.md) (in Chinese). The product, data, and code in the example are fictional.

## Code access

When a requirement involves changes to an existing system, the agent confirms the available code access during intake and records the level applied in the PRD:

| Level | Condition | Basis for current-state claims |
|---|---|---|
| L0 | No code provided | None; all related claims are marked as open questions |
| L1 | Code snippets or API documentation provided | Only the provided material |
| L2 | Repositories available locally | Read-only local search, verified to `file:line` |
| L3 | Multiple repositories with complex call chains | Located with a code graph, then verified in the source |
| L4 | Repositories hosted on GitHub, GitLab, or similar platforms | Read-only platform APIs; recall is typically narrower than with local search |

Public repositories require no credentials. Credentials for private repositories are stored only in the user's local environment variables; the agent does not record them in the conversation, the PRD, or any log.

## Output conventions

- The PRD body is intended for engineering review. Intake results, the requirement assessment, source facts and constraints, the requirement model, the review report, and the score are internal artifacts and are excluded from the body by default.
- Key conclusions carry a source label: explicit user statement, original material, existing PRD, code or system state, reasoned inference, or open question.
- The PRD specifies product-side data needs, business meaning, and error-handling requirements only. It does not define API fields, error codes, timeouts, caching, or other technical parameters.
- A metric whose scope, data source, time window, or attribution method is incomplete is treated as a post-launch observation metric, not as a committed target.
- Functional correctness, operational quality, and business impact are validated separately; tests that were not executed are not reported as passed.

## Directory structure

```text
ai-prd/
├── SKILL.md                              # Entry point: task modes, readiness intake, main workflow
├── references/
│   ├── intake.md                         # Readiness intake
│   ├── prd-generation.md                 # Execution modes, preconditions, steps, and validation
│   ├── requirement-method.md             # Requirement analysis and expression methods
│   ├── review-and-notification-rules.md  # Draft review, asynchronous events, and rate limiting
│   ├── review-regression-cases.md        # Check cases
│   ├── doc-sources.md                    # Material reading and completeness checks
│   └── code-evidence.md                  # Code access levels and evidence discipline
├── templates/
│   ├── prd-template.md                   # PRD template
│   └── evidence-template.md              # Code evidence template
└── examples/
    └── overdue-task-reminder/            # Fictional example
```

## Contributing

Issues and pull requests are welcome. When modifying a rule, update the [check cases](references/review-regression-cases.md) accordingly, specifying the expected and prohibited behavior associated with the new rule.

## License

[MIT](LICENSE)
