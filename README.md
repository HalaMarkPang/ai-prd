# AI-PRD

[English](README.en.md) | 中文

一个让 AI Agent 像资深产品经理一样写 PRD 的技能（Agent Skill）。它把一句话需求、BRD、会议纪要、已有 PRD 和代码现状，转化为研发能评审、测试能写用例的 PRD。

它和“给我一个 PRD 模板”的区别在于：

- **先问诊，再动笔。** 拿到需求后，先盘点材料、告诉你还缺什么、缺了会怎样，集中一轮问清（最多 5 个问题，每个都附默认处理），而不是直接生成一份建立在猜测上的 PRD。
- **先判断该不该做。** 生成前评估需求真伪、必要性、替代方案和范围边界，不成立时先和你对齐。
- **现状有据可查。** 按你能提供的代码访问级别取证；写进 PRD 的现状结论必须回读到 `文件:行号`，查不到的标为待确认。
- **尊重你的格式。** 有自己的模板或样稿时完全按你的格式写；没有时用本技能的 PRD 模板并按复杂度裁剪。
- **分清模式和权限。** 完整生成、评审精简稿、局部修订、只读审查四种模式；只读任务不会被转成写入。

## 快速开始

### 安装

AI-PRD 是一个标准的 Agent Skill 目录（入口是 `SKILL.md`）。安装时需要保留**完整目录**，入口会引用 `references/` 和 `templates/` 下的文件。

| 客户端 | 安装方式 |
|---|---|
| Claude Code | `git clone https://github.com/HalaMarkPang/ai-prd ~/.claude/skills/ai-prd` |
| Cursor | 克隆到 `~/.cursor/skills/ai-prd`，或项目内的 `.cursor/skills/ai-prd` |
| Codex | 克隆到 `~/.codex/skills/ai-prd`，或在 `AGENTS.md` 中引用 `SKILL.md` 并保证整个目录可读 |
| 其他 Agent | 让 Agent 读取 `SKILL.md` 及其引用的文件；只粘贴 `SKILL.md` 不足以加载完整规则 |

### 使用

直接对 Agent 说：

```text
用 AI-PRD 帮我写个 PRD：任务过了截止时间没人管，加个逾期提醒。
```

```text
基于这份 BRD 生成 PRD，按我们团队的模板写，代码在 ~/code/my-product。
```

```text
看看这份 PRD 还有什么问题，只在对话里告诉我，不要改文档。
```

```text
只改这份 PRD 的“失败处理”一节，其他保持不变。
```

Agent 首次回复会给出一张就绪度表和需要你确认的问题，你可以逐条回答，也可以直接说“按默认继续”。完整演示见 [示例：任务逾期提醒](examples/overdue-task-reminder/README.md)。

## 工作流程

```text
确定任务模式与读写授权
→ 就绪度问诊：盘点材料，集中一轮问清还缺什么
→ 需求评估（新建）/ 变更影响核对（修订）
→ 按需查证现状（代码证据回读到行）
→ 变更基线与业务模型
→ 规则、数据、交互与依赖
→ 按你的格式生成正文或片段
→ 验证与按授权交付
```

只读审查在反馈后结束，不生成文件、不写入。

## 代码怎么接入

问诊时 Agent 会问你代码在哪里，按你的情况选择：

| 级别 | 你的情况 | Agent 的做法 |
|---|---|---|
| L0 | 不提供代码 | 现状结论全部标为“待确认” |
| L1 | 能贴代码片段或接口文档 | 只基于贴出的内容判断 |
| L2 | 仓库在本机 | 本地只读检索，给出 `文件:行号` |
| L3 | 跨仓库、调用链复杂 | 先用代码图谱工具定位，再回读源码 |
| L4 | 仓库在 GitHub / GitLab | 只读接口查询，需要你在本机配置只读 token |

token 只放在你本机的环境变量里，Agent 不会要求你把它贴进对话。详见 [代码证据](references/code-evidence.md)。

## 目录结构

```text
ai-prd/
├── SKILL.md                         # 入口：模式分流、就绪度问诊、主流程
├── references/
│   ├── intake.md                    # 就绪度问诊
│   ├── prd-generation.md            # 执行模式、前置约束、步骤与验证
│   ├── requirement-method.md        # 通用需求分析与表达方法
│   ├── review-and-notification-rules.md  # 成稿审查、异步事件与频控
│   ├── review-regression-cases.md   # 合成检查样例
│   ├── doc-sources.md               # 材料来源与读取
│   └── code-evidence.md             # 代码访问分级与源码闭环纪律
├── templates/
│   ├── prd-template.md              # PRD 模板（可裁剪）
│   └── evidence-template.md         # 代码证据档模板
└── examples/
    └── overdue-task-reminder/       # 虚构的完整示例
```

## 设计原则

1. 需求评估前置；不成立时先对齐，不强行生成。
2. 关键结论标注来源：用户说明、原始材料、已有 PRD、代码现状、合理推断、待确认。
3. PRD 不替研发定义接口协议、字段、错误码、超时、缓存等技术参数。
4. 指标口径不清时只写观察指标，不写目标承诺。
5. 同一规则只完整定义一次；共性集中，差异引用。
6. 功能正确性、运行质量、业务效果分开验证；没执行的测试不称通过。
7. 示例中的参数只属于示例，不会被当作默认规则。

## 贡献

欢迎提交 Issue 和 PR。修改规则时请同步更新 `references/review-regression-cases.md` 中的检查样例。

## 许可证

[MIT](LICENSE)
