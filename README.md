# AI-PRD

[English](README.en.md) | 中文

AI-PRD 是一套用于生成、修订和审查产品需求文档（PRD）的 Agent Skill。它规定了 Agent 从需求材料到研发评审稿的工作流程、判断规则和验证标准，适用于 Claude Code、Codex、Cursor 等支持 Skill 或规则文件的 Agent 客户端。

## 解决的问题

由 AI 直接生成的 PRD，常见问题集中在以下五类：

1. 信息不足时仍然直接生成全文，关键结论建立在推测之上。
2. 未判断需求是否成立，就进入方案设计。
3. 对现有系统的描述缺乏依据，与实际实现不符。
4. 套用固定模板，与团队既有格式和评审需要不一致。
5. 用户只要求审查或局部修改，Agent 却改写了全文或在线文档。

## 对应机制

| 问题 | 机制 | 规则文件 |
|---|---|---|
| 信息不足仍直接生成 | 就绪度问诊：先盘点材料，按八项检查标注“已具备 / 可假设 / 阻塞”，就阻塞项集中提问一轮，每个问题附默认处理 | [intake.md](references/intake.md) |
| 未判断需求是否成立 | 需求评估前置：从真实性、必要性、替代方案、范围边界四个方面给出结论，结论不成立时先与用户对齐 | [prd-generation.md](references/prd-generation.md) |
| 现状描述缺乏依据 | 代码证据分级（L0–L4）与取证纪律：写入 PRD 的现状结论须核对到 `文件:行号`，无法核对的列为待确认 | [code-evidence.md](references/code-evidence.md) |
| 格式与团队要求不符 | 格式优先级：用户提供的模板或样稿优先；没有时使用本技能的 PRD 模板，并按需求复杂度裁剪 | [prd-template.md](templates/prd-template.md) |
| 越权修改 | 任务模式与读写授权：区分完整生成、评审精简稿、局部修订、只读审查；只读任务不转为写入 | [review-and-notification-rules.md](references/review-and-notification-rules.md) |

## 工作流程

```text
确定任务模式与读写授权
→ 就绪度问诊
→ 需求评估（新建需求）/ 变更影响核对（局部修订）
→ 按需查证现状
→ 建立变更基线与业务模型
→ 定义规则、数据、交互与依赖
→ 按用户格式生成正文或修订片段
→ 验证，并在授权范围内交付
```

只读审查在反馈结论后结束，不生成文件，也不写入任何文档。

## 安装

AI-PRD 以 `SKILL.md` 为入口，并引用 `references/` 与 `templates/` 下的规则文件。安装时需保留完整目录。

| 客户端 | 安装方式 |
|---|---|
| Claude Code | `git clone https://github.com/HalaMarkPang/ai-prd ~/.claude/skills/ai-prd` |
| Cursor | 克隆到 `~/.cursor/skills/ai-prd`，或项目内的 `.cursor/skills/ai-prd` |
| Codex | 克隆到 `~/.codex/skills/ai-prd`，或在 `AGENTS.md` 中引用 `SKILL.md`，并保证整个目录可读 |
| 其他 Agent | 让 Agent 读取 `SKILL.md` 及其引用的文件；仅提供 `SKILL.md` 无法加载完整规则 |

## 使用

在对话中说明任务即可。以下示例分别对应四种任务模式：

| 任务模式 | 示例 |
|---|---|
| 完整生成 | 用 AI-PRD 为“任务逾期提醒”写一份 PRD，代码在 `~/code/my-product`。 |
| 评审精简稿 | 基于这份 BRD 生成研发评审稿，按附件中的团队模板组织。 |
| 局部修订 | 只修改这份 PRD 的“异常与失败处理”一节，其他内容保持不变。 |
| 只读审查 | 审查这份 PRD，只在对话中反馈问题，不修改文档。 |

Agent 的首次回复包含需求理解、就绪度表和待确认问题。可以逐项回答，也可以回复“按默认继续”，此时 Agent 会把采用的默认假设列入 PRD 的待确认问题。

完整过程见示例 [任务逾期提醒](examples/overdue-task-reminder/README.md)。示例中的产品、数据与代码均为虚构。

## 代码访问

涉及已有系统改造时，Agent 会在问诊中确认代码访问方式，并在 PRD 中注明实际采用的级别：

| 级别 | 条件 | 现状结论的依据 |
|---|---|---|
| L0 | 不提供代码 | 无；相关结论全部标为待确认 |
| L1 | 提供代码片段或接口文档 | 仅限所提供的内容 |
| L2 | 仓库位于本机 | 本地只读检索，核对到 `文件:行号` |
| L3 | 跨仓库、调用链复杂 | 代码图谱定位后回读源码 |
| L4 | 仓库位于 GitHub、GitLab 等远端 | 平台只读接口；召回范围通常小于本地检索 |

访问凭证仅保存在用户本机的环境变量中，Agent 不在对话、PRD 或日志中记录凭证。

## 输出约定

- PRD 正文面向研发评审。需求评估、事实抽取、需求模型和审查报告属于内部过程产物，默认不写入正文。
- 关键结论标注来源：用户说明、原始材料、已有 PRD、代码现状、合理推断或待确认。
- PRD 只描述产品侧的数据需求、业务含义和异常处理要求，不定义接口字段、错误码、超时、缓存等技术参数。
- 指标的口径、数据源、时间窗和归因方式不完整时，只作为上线后的观察指标，不写作目标承诺。
- 功能正确性、运行质量和业务效果分别验证；未执行的测试不标记为通过。

## 目录结构

```text
ai-prd/
├── SKILL.md                              # 入口：任务模式、就绪度问诊、主流程
├── references/
│   ├── intake.md                         # 就绪度问诊
│   ├── prd-generation.md                 # 执行模式、前置约束、步骤与验证
│   ├── requirement-method.md             # 需求分析与表达方法
│   ├── review-and-notification-rules.md  # 成稿审查、异步事件与频控
│   ├── review-regression-cases.md        # 检查样例
│   ├── doc-sources.md                    # 材料读取与完整性核对
│   └── code-evidence.md                  # 代码访问分级与取证纪律
├── templates/
│   ├── prd-template.md                   # PRD 模板
│   └── evidence-template.md              # 代码证据档模板
└── examples/
    └── overdue-task-reminder/            # 虚构示例
```

## 贡献

欢迎提交 Issue 和 Pull Request。修改规则时，请同步更新 [检查样例](references/review-regression-cases.md)，说明新规则对应的预期行为与禁止行为。

## 许可证

[MIT](LICENSE)
