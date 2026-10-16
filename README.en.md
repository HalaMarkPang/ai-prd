# AI-PRD

English | [中文](README.md)

An Agent Skill that makes AI agents write PRDs (product requirements documents) the way a senior product manager would. It turns a one-line idea, a BRD, meeting notes, an existing PRD, and the current codebase into a PRD that engineers can review and QA can turn into test cases.

> The rule files are written in Chinese. Agents that read Chinese (Claude, GPT, Gemini, Qwen, etc.) follow them regardless of the conversation language, and the PRD is written in whatever language you ask for.

What makes it different from "give me a PRD template":

- **Intake before writing.** The agent first inventories your materials, tells you what is missing and why it matters, and asks one consolidated round of at most five questions, each with a default you can accept. It does not produce a confident-looking PRD built on guesses.
- **Should we build it at all?** Before writing, it assesses whether the problem is real, whether it is worth doing now, cheaper alternatives, and scope. If the case does not hold, it aligns with you first.
- **Evidence-backed current state.** It gathers code evidence according to the access you can provide. Any claim about the current system must be traced back to `file:line`; anything unverified is marked "to be confirmed".
- **Your format first.** If you have a template or a sample PRD, it follows that. Otherwise it uses its PRD template, trimmed to the requirement's complexity.
- **Modes and permissions.** Full draft, review-ready concise draft, partial revision, and read-only review. Read-only requests never turn into writes.

## Quick start

### Install

AI-PRD is a standard Agent Skill directory with `SKILL.md` as the entry point. Keep the **whole directory**: the entry file references files under `references/` and `templates/`.

| Client | How |
|---|---|
| Claude Code | `git clone https://github.com/HalaMarkPang/ai-prd ~/.claude/skills/ai-prd` |
| Cursor | Clone into `~/.cursor/skills/ai-prd` or a project's `.cursor/skills/ai-prd` |
| Codex | Clone into `~/.codex/skills/ai-prd`, or reference `SKILL.md` from `AGENTS.md` and keep the directory readable |
| Other agents | Have the agent read `SKILL.md` and the files it references; pasting `SKILL.md` alone is not enough |

### Use

Just ask your agent:

```text
Use AI-PRD to write a PRD: overdue tasks get ignored, add an overdue reminder.
```

```text
Turn this BRD into a PRD using our team template. The code is in ~/code/my-product.
```

```text
Review this PRD and tell me the issues in chat only. Do not edit the document.
```

The first reply contains a readiness table and the questions it needs answered. Answer them, or reply "continue with defaults". See the [worked example](examples/overdue-task-reminder/README.md) (in Chinese).

## Workflow

```text
Task mode and write permissions
→ Intake: inventory materials, ask one round of questions
→ Requirement assessment (new) / change impact check (revision)
→ Verify current state as needed (code evidence traced to lines)
→ Change baseline and business model
→ Rules, data, interaction, dependencies
→ Draft in your format, or produce a patch
→ Validate and deliver within the granted permissions
```

## Connecting code

During intake the agent asks where your code lives:

| Level | Your situation | What the agent does |
|---|---|---|
| L0 | No code access | Marks every current-state claim as "to be confirmed" |
| L1 | You paste snippets or API docs | Reasons only from what you pasted |
| L2 | Repos are cloned locally | Read-only local search with `file:line` evidence |
| L3 | Many repos, complex call chains | Locates with a code-graph tool, then reads the source |
| L4 | Repos on GitHub / GitLab | Read-only API queries; you set a read-only token locally |

Tokens stay in your local environment variables. The agent never asks you to paste them into the chat.

## Contributing

Issues and PRs are welcome. When changing rules, update the cases in `references/review-regression-cases.md`.

## License

[MIT](LICENSE)
