# 产品目标

把用户从网页、书籍、视频、课程或个人笔记中获得的阅读输入转化为可持续复习的学习材料。产品不限制原始内容的类型，也不要求输入必须具备书名、作者、封面或章节等字段。产品包含两个相互独立、可选择连接的学习流程：

```text
流程一：阅读输入 → 生成练习草稿 → 用户审核 → 主动回答
        → 反馈并揭示答案 → 用户评价回想表现 → FSRS 排期

流程二：用户选择概念 → 讲解追问 → 追问与诊断 → 重新解释
        → 保存理解缺口 → 可选生成补充问答卡草稿
```

流程一（今日复习｜问答卡）是日常、高频的记忆复习闭环；
流程二（理解练习｜讲解追问、理解问答、概括对照）是用户主动发起的低频理解训练。
两者共享卡片集、内容卡片、概念和来源数据，但不会强制连续执行。

**实现说明（与 MVP 对齐）**：参考实现位于独立运行环境的 `<APP_HOME>/`（不在本仓库）。当前 MVP 为单用户 box + SQLite + 本地 FSRS；材料生成与评阅默认为**启发式 / 规则 stub**，更高质量内容可通过 `draft-import` 由 Flash/LLM 替换。无多租户鉴权，无默认通知推送，无微信读书导入。

## 产品边界

Anki 机制覆盖问答卡、队列、揭示答案、用户评分和间隔调度。本产品使用卡片集和内容卡片承载任意阅读输入，并增加问答式与选择题生成，以及用户可选的理解问答、概括对照、讲解追问和薄弱点回流（补卡需用户确认）。

问答卡复习是日常记忆闭环，理解练习是独立的理解训练。二者共享卡片集、内容卡片与概念，但不会自动串联。即使生成/评阅增强不可用，用户仍能查看内容卡片、手动创建问答卡、揭示既有答案、完成评分并继续 FSRS 复习。

## 参考资料

- [Anki Manual：Introduction](https://docs.ankiweb.net/manual/intro)
- [Anki Manual：Getting Started](https://docs.ankiweb.net/manual/getting-started)
- [Anki Manual：Studying](https://docs.ankiweb.net/manual/studying)
- [Anki Manual：Deck Options 与 FSRS](https://docs.ankiweb.net/manual/deck-options)
- [ts-fsrs](https://github.com/open-spaced-repetition/ts-fsrs)
