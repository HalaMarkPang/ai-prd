# AI-PRD

English | [中文](README.md)

AI-PRD is an Agent Skill for generating, revising, and reviewing product requirements documents (PRDs). It defines the workflow, decision rules, and validation criteria an agent follows when turning requirement materials into a draft suitable for engineering review. It applies to agent clients that support skills or rule files, including Claude Code, Codex, and Cursor.

The rule files are written in Chinese. Models with adequate Chinese comprehension apply them regardless of the conversation language. The PRD is produced in the language specified by the user.

## Problems addressed

PRDs generated directly by a model commonly exhibit the following deficiencies:

1. A full document is produced before the materials have been read in full, or while information is still missing; key conclusions rest on speculation.
2. Solution design begins before the requirement has been assessed; experiential complaints, causal guesses, and build goals are conflated.
3. Descriptions of the existing system lack verifiable evidence and diverge from the implementation.
4. A fixed template is applied without regard to the team's established format and review requirements.
5. When the user requests only a review or a partial edit, the agent rewrites the entire document or writes to an online document.

## Mechanisms

| Problem | Mechanism | Rule file |
|---|---|---|
| Generation despite unread or incomplete materials | Full-read gate and readiness intake: a complete conclusion is issued only after the designated materials have been read (or a partial-read boundary has been stated); eight checks are rated available, assumable, or blocking; at most five opening questions address blocking items, each with a default | [requirement-method.md](references/requirement-method.md), [intake.md](references/intake.md) |
| Requirement not assessed | Up-front assessment: statement layering (fact / interpretation / value judgment / goal); conclusions on validity, necessity, alternatives, and scope; multi-option confrontation before a build decision; critical unknowns that affect whether or how to build are clarified first and are not replaced by an open-question list | [requirement-method.md](references/requirement-method.md), [prd-generation.md](references/prd-generation.md) |
| Unsupported claims about the existing system | Code access levels (L0–L4) and evidence discipline: current-state claims must be verified to `file:line`; existing-system changes are checked hop by hop along the actual path; unverifiable claims are listed as open questions | [code-evidence.md](references/code-evidence.md) |
| Format inconsistent with team requirements | Format priority: the user's template or sample takes precedence; otherwise the skill's PRD template is used and trimmed to complexity. The default template includes acceptance criteria and code clues; externalization follows the user's instruction | [prd-template.md](templates/prd-template.md) |
| Unauthorized modification | Task modes and write authorization: full generation, concise review draft, partial revision, and read-only review; without an explicit user request, a read-only task is never converted into a write | [prd-generation.md](references/prd-generation.md#执行模式), [review-and-notification-rules.md](references/review-and-notification-rules.md) |

## Workflow

```text
Determine the task mode and write authorization
→ Read the designated materials in full (or state a partial-read boundary) → Readiness intake
→ Requirement assessment (statement layering; clarify critical unknowns; confront multiple options) / change-impact check (partial revision)
→ Verify the current state as needed
→ Establish the change baseline and business model
→ Define rules, data, interaction, and dependencies
→ Produce the document body or revision in the user's format
→ Review and validate; deliver within the authorized scope
```

## Constraints and boundaries

1. **Full read**: A complete assessment conclusion and a complete document body are issued only after the designated materials have been read. Section navigation and material-pack summaries are used only to locate the source text. A partial read yields only a stage judgment limited to what was read, with the effect of unread parts stated.
2. **Intake and clarification**: Opening readiness intake asks at most five questions in a single round, each with its reason and default; no questions are raised when the materials are complete. After assessment begins, only unknowns that may still change whether to build, the scope, or the option are clarified one at a time. Clarification items are resolved before delivery; the open-questions section retains only items still unresolved at delivery.
3. **Read-only review**: The task ends once feedback is delivered; no file is created and no document is modified. If the user subsequently requests changes explicitly, the task becomes a partial revision, and the write target is confirmed before any write.
4. **Revision limit**: If the internal review score is below 85, at most two revision rounds are performed. If the draft still does not meet the threshold, self-revision stops; unmet dimensions and gaps are listed for the user to supply material, accept the current version, or assign manual completion.
5. **Unattended runs**: In unattended environments such as CI, the agent neither asks questions nor aborts. Blocking items and adopted assumptions are recorded in the generation note at the top of the PRD and in its open questions, and are marked as not confirmed by the user.

## Installation

`SKILL.md` is the entry point and references the rule files under `references/` and `templates/`. The complete directory must be installed.

| Client | Installation |
|---|---|
| Claude Code | `git clone https://github.com/HalaMarkPang/ai-prd ~/.claude/skills/ai-prd` |
| Cursor | Clone into `~/.cursor/skills/ai-prd`, or into a project's `.cursor/skills/ai-prd` |
| Codex | Clone into `~/.codex/skills/ai-prd`, or reference `SKILL.md` from `AGENTS.md` and ensure that the complete directory is readable |
| Other agents | Instruct the agent to read `SKILL.md` and every file it references; `SKILL.md` alone does not load the complete rule set |

Method-sync boundaries are described in [SYNC.md](SYNC.md).

## Usage

State the task objective in the conversation and provide the relevant materials. The examples below correspond to the four task modes:

| Task mode | Example request |
|---|---|
| Full generation | Use AI-PRD to generate a PRD for an overdue-task reminder. The code is in `~/code/my-product`. |
| Concise review draft | Produce an engineering review draft from this BRD, structured according to the attached team template. |
| Partial revision | Revise only the "Exceptions and failure handling" section of this PRD; leave all other content unchanged. |
| Read-only review | Review this PRD and report issues in the conversation only. Do not modify the document. |

The agent's first reply contains its understanding of the requirement, a readiness table, and any questions that require confirmation. The user may answer each question or instruct the agent to continue under the stated defaults; in the latter case, the adopted assumptions are recorded in the generation note at the top of the PRD and in its open questions.

For a complete walkthrough, see the [overdue-task reminder example](examples/overdue-task-reminder/README.md) (in Chinese). The product, data, and code in the example are fictional.

### Code access

When a requirement involves changes to an existing system, the agent confirms the available code access during intake and records the level applied in the PRD:

| Level | Condition | Basis for current-state claims |
|---|---|---|
| L0 | No code provided | None; all related claims are marked as open questions |
| L1 | Code snippets or API documentation provided | Only the provided material |
| L2 | Repositories available locally | Read-only local search, verified to `file:line` |
| L3 | Multiple repositories with complex call chains | Located via a code graph or reference search, then verified in source |
| L4 | Repositories hosted on GitHub, GitLab, or similar platforms | Read-only platform APIs; recall is typically narrower than local search |

Public repositories require no credentials. Credentials for private repositories are stored only in the user's local environment variables; the agent does not record them in the conversation, the PRD, or any log.

## Output conventions

- The PRD body is intended for engineering review. Intake results, the requirement assessment, source facts and constraints, the requirement model, the review report, and the score are internal process artifacts and are excluded from the body by default.
- Key conclusions carry a source label: explicit user statement, original material, existing PRD, code or system state, reasoned inference, or open question.
- The PRD specifies product-side data needs, business meaning, and error-handling requirements. It does not define API field names, error codes, timeouts, caching, or other technical parameters unless those parameters appear in confirmed interface documentation.
- A metric whose definition, data source, time window, or attribution method is incomplete is treated as a post-launch observation metric, not as a committed target.
- Functional correctness, operational quality, and business impact are validated separately; tests that were not executed are not reported as passed.
- Static prototypes support only layout and operations visible in the figure. An interactive HTML prototype, when requested as a verification artifact, remains separate from the PRD body and does not replace its structure.

## Directory structure

```text
ai-prd/
├── SKILL.md                              # Entry point: task modes, intake, main workflow
├── SYNC.md                               # Sync boundaries with any internal method suite
├── references/
│   ├── intake.md                         # Readiness intake
│   ├── prd-generation.md                 # Execution modes, preconditions, steps, and validation
│   ├── requirement-method.md             # Requirement analysis and expression methods
│   ├── review-and-notification-rules.md  # Draft review, asynchronous events, and rate limiting
│   ├── review-regression-cases.md        # Synthetic check cases
│   ├── doc-sources.md                    # Material reading and completeness checks
│   └── code-evidence.md                  # Code access levels and evidence discipline
├── templates/
│   ├── prd-template.md                   # PRD template
│   └── evidence-template.md              # Code evidence template
└── examples/
    └── overdue-task-reminder/            # Fictional example
```

## Contributing

Issues and pull requests are welcome. When modifying a rule, update the [check cases](references/review-regression-cases.md) accordingly, stating the expected and prohibited behavior, and observe the sanitization and sync boundaries in [SYNC.md](SYNC.md).

## License

[MIT](LICENSE)
