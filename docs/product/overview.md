# 产品目标

把用户从网页、书籍、视频、课程或个人笔记中获得的阅读输入转化为可持续复习的学习材料。产品不限制原始内容的类型，也不要求输入必须具备书名、作者、封面或章节等字段。产品包含两个相互独立、可选择连接的学习流程：

```text
流程一：阅读输入 → Grok Bot 生成材料 → 用户审核 → 主动回答
        → Grok Bot 反馈并揭示答案 → 用户评价回想表现 → FSRS 排期

流程二：用户选择概念 → 费曼解释 → Grok Bot 追问 → 重新解释
        → 保存理解缺口 → 可选生成补充问答卡
```

流程一（今日复习｜问答卡）是日常、高频的记忆复习闭环；
流程二（深度练习 ｜费曼解释、开放题、摘要重写）是用户主动发起或单独安排的低频深度练习。
两者共享卡片集、内容卡片、概念和来源数据，但不会强制连续执行。系统只基于用户提供的阅读输入生成学习内容。Grok Bot 服务负责生成与评价，用户负责最终记忆评分，FSRS 服务负责下一次问答卡复习时间，应用组件负责流程和数据一致性。

## 产品边界

Anki 机制覆盖问答卡、队列、揭示答案、用户评分和间隔调度。本产品使用卡片集和内容卡片承载任意阅读输入，并增加 Grok Bot 驱动的问答式与选择题生成，以及用户可选的开放题、摘要重写、费曼追问和薄弱点回流。

问答卡复习是日常记忆闭环，深度练习是独立的理解训练。二者共享卡片集、内容卡片、概念和来源，但不会自动串联。Grok Bot 服务不是调度器，数据库服务也不承担生成逻辑。即使 Grok Bot 服务暂时不可用，用户仍能查看内容卡片、手动创建问答卡、揭示既有答案、完成评分并继续 FSRS 复习。

## 参考资料

- [Anki Manual：Introduction](https://docs.ankiweb.net/manual/intro)
- [Anki Manual：Getting Started](https://docs.ankiweb.net/manual/getting-started)
- [Anki Manual：Studying](https://docs.ankiweb.net/manual/studying)
- [Anki Manual：Deck Options 与 FSRS](https://docs.ankiweb.net/manual/deck-options)
- [ts-fsrs](https://github.com/open-spaced-repetition/ts-fsrs)
