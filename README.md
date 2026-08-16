## ✨ 核心理念

> **Summary 描述发生了什么。**

> **Work State 描述现在什么是真的，以及接下来应该做什么。**

因此，本 Skill 的所有信息都围绕一个问题进行筛选：

> **“如果一个新的 Agent 只看到这份 Work State，它能否正确继续当前工作？”**

不属于未来工作所必需的信息，会被主动舍弃，避免 Work State 重新变成一份冗长的聊天总结。

---

## 🚀 功能亮点

### ✅ 1. 工作状态提取

从长对话中提取真正重要的信息：

* 当前目标
* 当前进度
* 已确认决策
* 已否决方案
* 未解决问题
* 下一步行动
* 当前约束

### ✅ 2. 保护已确认决策

如果用户已经明确确认某个方案，后续 AI 的猜测或建议不能覆盖它。

### ✅ 3. 记录被否决方案

记录已经明确被拒绝的方案及原因，避免新的 Agent 重复走旧路。

### ✅ 4. 保留未解决问题

无法确定的内容进入 `Open Questions`，而不是让 Agent 擅自补全。

> [!NOTE]

> **“不知道”也是一种有效状态。**

### ✅ 5. 自动生成 Resume Prompt

生成一份可以直接复制到新聊天中的 **Resume Prompt（续接提示词）**，帮助新的 Agent 快速恢复工作状态。

### ✅ 6. 支持跨 Agent 续接

Resume Prompt 使用普通文本，不依赖特定 AI 平台，因此可以用于：

* Claude
* ChatGPT
* Gemini
* 其他聊天型 AI Agent

### ✅ 7. 支持上下文漂移恢复

当 Agent 重复询问、重新提出已否决方案、忘记当前目标或与已确认决定冲突时，可以通过最近保存的 Work State 重新建立工作状态。

---

## 🧠 Work State 结构

当前版本采用轻量的七字段结构：

```text
Work State

│

├── Goal

├── Current status

├── Confirmed decisions

├── Rejected approaches

├── Open questions

├── Next action

└── Constraints
```

| 字段                  | 作用                        |

| ------------------- | ------------------------- |

| Goal                | 当前工作的核心目标，以及什么情况下可以认为工作完成 |

| Current status      | 当前已经做到什么程度，以及现在正在处理什么     |

| Confirmed decisions | 用户已经明确确认或批准的决定            |

| Rejected approaches | 已经明确被否决的方案及原因             |

| Open questions      | 目前仍然没有解决的问题               |

| Next action         | 恢复工作后应该立即执行的一个具体下一步       |

| Constraints         | 工作过程中必须遵守的限制              |

---

## 🔄 工作流程

```text
长对话

   ↓

识别核心信息

   ↓

提取 Work State

   ↓

用户审阅 / 修改

   ↓

保存工作状态

   ↓

生成 Resume Prompt

   ↓

新聊天 / 新 Agent

   ↓

继续工作
```

> [!IMPORTANT]

> Work State 在保存或交接之前，应先让用户看到并确认。

> 这样可以避免 AI 自己推测的信息被错误地带入下一次对话。

---

## 📄 Work State 示例

```markdown
# Work State: Context Continuity MVP

Updated: 2026-08-16



## Goal

构建一个轻量的 AI 工作状态层，让用户无需手动管理上下文，也能继续之前未完成的工作。



## Current status

产品方向已经确定，目前正在设计第一版 MVP。



## Confirmed decisions

- 核心不是聊天总结，而是 Work State。

- 主要入口是“新聊天继续”。

- 需要支持 Agent 忘记上下文后的恢复。

- 跨 Agent 交接属于后续能力。



## Rejected approaches

- 只做 HANDOFF 文档生成器。

- 要求用户手动维护复杂 Memory。



## Open questions

- 第一版 MVP 的具体技术架构

- Evidence 在第一版中需要保留到什么程度



## Next action

开始实现第一版 MVP。



## Constraints

- 用户体验必须保持轻量。

- 不应该要求用户主动管理大量上下文。

- 优先控制基础设施和模型调用成本。
```

---

## 🔁 Resume Prompt 示例

```text
You are continuing existing work.



Do not restart from scratch.



GOAL

构建一个轻量的 AI 工作状态层，让用户无需手动管理上下文，也能继续之前未完成的工作。



CURRENT STATUS

产品方向已经确定，目前正在设计第一版 MVP。



CONFIRMED DECISIONS

- 核心不是聊天总结，而是 Work State。

- 主要入口是“新聊天继续”。

- 需要支持 Agent 忘记上下文后的恢复。

- 跨 Agent 交接属于后续能力。



REJECTED APPROACHES

- 不要重新提出只做 HANDOFF 文档生成器。

- 不要把产品变成复杂的 Memory 管理工具。



OPEN QUESTIONS

- 第一版 MVP 的具体技术架构

- Evidence 在第一版中需要保留到什么程度



NEXT ACTION

开始实现第一版 MVP。



CONSTRAINTS

- 用户体验必须保持轻量。

- 不应该要求用户主动管理大量上下文。



如果以上信息存在明显不足或矛盾，只询问必要的澄清问题；否则直接继续下一步工作，不要重新总结整个背景。
```

---

## 🛡️ 冲突处理原则

发生冲突时，优先级遵循：

```text
用户明确确认
```

↓

```
用户明确陈述
```

↓

```
当前对话中的明确决定
```

↓

```
多次一致的上下文
```

↓

```
AI 推断
```

↓

```
模型猜测
```

> **AI 的建议不能覆盖用户已经明确确认的决定。**

只有用户明确改变决定，原来的确认状态才应该被更新。

---

## 🧩 上下文漂移恢复

当 Agent 已经出现明显的上下文漂移时，可以使用最近一次保存的 Work State。

```text
发现 Agent 跑偏
```

↓

```
读取最近 Work State
```

↓

```
重新确认：

- Goal

- Confirmed decisions

- Next action
```

↓

```
Agent 重新对齐
```

↓

```
继续工作
```

> [!WARNING]

> 恢复时不要假装 Agent 一直记得上下文。

> 应明确重新建立状态，让用户确认 Agent 已经回到正确轨道。

---

## 🚫 本 Skill 不做什么

为了保持轻量，目前明确不做：

* 自动保存所有聊天记录；
* 自动管理完整上下文窗口；
* 静默向其他 AI 平台同步内容；
* 保存对话中的所有信息；
* 将 Work State 做成庞大的知识库。

核心目标始终是：

> **保留能够继续工作的核心状态，而不是保存一切。**

---

## 🎯 适用场景

特别适合：

* 长时间 AI 项目协作；
* 产品设计；
* 编程开发；
* 深度研究；
* 长篇写作；
* 多轮方案讨论；
* 需要切换新聊天的任务；
* 需要从一个 AI Agent 转交给另一个 Agent 的工作。

---

## 🗂️ 项目结构

```text
context-continuity-skill/

├── README.md

├── LICENSE

├── CHANGELOG.md

│

├── skills/

│   └── context-continuity/

│       ├── SKILL.md

│       └── templates/

│           ├── work-state.md

│           └── resume-prompt.md

│

├── examples/

│   └── example-work-state.md

│

└── docs/
```

└── design.md

```

```

---

## 🚀 快速开始

将：

```text
skills/context-continuity/
```

目录放入你的 AI Agent 所使用的 Skill 目录中，并使用其中的：

```text
SKILL.md
```

作为 Skill 定义。

生成 Work State 后，可以将对应的 Resume Prompt 直接复制到新的 AI 对话中继续工作。

---

## 💡 设计理念

### 不是“压缩文字”，而是“压缩状态”

传统上下文压缩：

```text
完整对话

   ↓

压缩文本
```

Context Continuity：

```text
完整对话

   ↓

提取工作状态

   ↓

保留：

- 目标

- 决策

- 约束

- 当前状态

- 未解决问题

- 下一步

   ↓

新 Agent 继续工作
```

核心判断标准只有一个：

> **新的 Agent 能不能接着做，而不是它有没有读完所有历史。**

---

## 🛣️ 后续方向

当前版本刻意保持简单。

未来可以进一步探索：

* 工作状态 Checkpoint；
* 状态版本管理；
* Evidence / 来源追踪；
* 增量状态合并；
* 冲突检测；
* Context Drift 检测；
* State Alignment；
* 跨 Agent Handoff。

这些能力可以建立在当前 Work State 模型之上，而不需要改变最基础的使用方式。

---

## 🤝 贡献

欢迎围绕以下方向提交 Issue 或贡献：

* Work State 结构优化；
* 上下文提取规则；
* 冲突处理策略；
* Resume Prompt 优化；
* 跨 Agent 使用场景；
* 实际项目中的测试案例。

提交改动时，请优先保证：

> **轻量、可审阅、可携带、可继续工作。**

---

## 📜 开源协议

本项目采用 **MIT License**。

欢迎自由使用、修改和扩展。

---

> **Context Continuity**

> **不保存所有对话，只保存让你继续工作所需要的状态。**

```

```
