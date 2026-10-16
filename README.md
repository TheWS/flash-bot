# Flash Bot

Flash Bot 是一个运行在 Grok Bot / Flash 对话环境中的主动阅读与间隔复习产品设计。它将网页、书籍、视频、课程或个人笔记转化为问答卡、开放题和摘要练习，并通过 FSRS 安排长期复习。

## 核心流程

```text
阅读输入 → 生成练习草稿 → 用户审核 → 主动回想
        → 揭示答案与反馈 → 用户自评 → FSRS 排期

用户主动选择概念 → 费曼解释 → Bot 追问 → 暴露理解缺口
                  → 用户确认后生成补充问答卡草稿
```

日常问答卡复习和深度练习是两条独立流程。费曼解释、开放题和摘要重写不会自动插入日常 FSRS 队列。

## 文档导航

### 产品

- [产品概述](docs/product/overview.md)：目标、学习流程、术语和产品边界。
- [功能需求](docs/product/requirements.md)：功能编号、优先级、详细行为和验收标准。
- [用户工作流](docs/product/user-workflows.md)：对话口令及用户侧操作含义。

### 技术

- [系统架构](docs/technical/architecture.md)：组件、服务及职责边界。
- [数据模型与业务规则](docs/technical/data-model.md)：核心实体、状态和系统约束。
- [系统时序](docs/technical/sequences.md)：材料生成、复习、深度练习和异常恢复时序。
- [MVP 实现说明](docs/technical/mvp-implementation.md)：当前实现形态、CLI 映射和落地状态。

### Bot Skill

- [主动阅读学习规则](skills/active-reading-learning-rules/SKILL.md)
- [结构化输入输出契约](skills/active-reading-learning-rules/references/output-contracts.md)

### 图表

- [系统时序图（SVG）](diagrams/grok-active-reading/system-sequence.svg)
- [系统时序图（PNG）](diagrams/grok-active-reading/system-sequence@2x.png)

## 核心概念

- **卡片集（CardSet）**：组织任意来源的阅读输入，不绑定书籍结构。
- **内容卡片（ContentCard）**：保存一段原始阅读输入。
- **问答卡（ReviewCard）**：用于主动回想，支持问答式和选择题。
- **FSRS**：根据实际复习时间和用户自评计算下一次复习日期。
- **学习规则 Skill**：约束 Bot 如何生成材料、评价回答和进行费曼追问。

## 当前状态

仓库当前以产品设计、技术规范、系统时序和 Bot Skill 为主。MVP 采用 SQLite 持久化和本地 FSRS 调度；实现代码不包含在本仓库中。

## 安全约定

仓库不提交 API 密钥、访问令牌、本地数据库、备份或 `.env` 文件。运行时凭据应通过部署环境的密钥管理机制提供。
