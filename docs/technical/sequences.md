# 系统时序

## 1. 阅读材料生成与发布

```mermaid
sequenceDiagram
    actor U as 用户
    participant A as 材料生成组件
    participant D as 数据库服务
    participant G as Grok Bot 服务
    participant K as 学习规则组件

    alt 手动录入
        U->>A: 提交卡片集、内容卡片与阅读输入
    else 微信读书笔记导入
        U->>A: 授权读取并选择划线与笔记
        A->>D: 查询 ImportRecord 并执行去重
        A-->>U: 预览卡片集与内容卡片映射
        U->>A: 确认导入
    end
    A->>A: 校验正文、去重结果和可选来源字段
    A->>D: 保存 ContentCard 与版本
    A->>G: 创建材料生成任务
    G->>K: 加载概念提取、出题及来源约束
    K-->>G: 返回结构和质量规则
    G-->>A: 返回概念、两类问答卡、开放题、摘要及来源引用（MVP：启发式或 draft-import）
    A->>A: 校验结构、数量和来源引用
    alt 输出有效
        A->>D: 保存为草稿，答案标记为隐藏
        A-->>U: 展示草稿和来源供审核
        U->>A: 编辑、删除、拆分并确认
        A->>D: 发布材料并初始化问答卡状态
        D-->>A: 返回发布结果
        A-->>U: 显示已创建内容及待学习数量
    else 输出无效
        A->>D: 记录失败状态和错误
        A-->>U: 保留原始输入并允许重试
    end
```

## 2. 主动回想与 FSRS 调度

> 注：开场须先选集；通知服务为产品能力，MVP 默认关闭 learning 短间隔推送（见规则 20、22）。

```mermaid
sequenceDiagram
    actor U as 用户
    participant A as 问答卡复习组件
    participant D as 数据库服务
    participant G as Grok Bot 服务
    participant F as FSRS 服务

    U->>A: 打开今日复习并指定卡片集
    A->>D: 查询该集到期、逾期、学习中和新问答卡
    D-->>A: 返回队列，排除失效、作废、暂停与暂缓问答卡
    alt 问答式
        A-->>U: 展示问题和文本输入
        U->>A: 提交回答文本或选择想不起来
    else 选择题
        A-->>U: 展示问题和选项
        U->>A: 提交所选选项或选择想不起来
    end
    A->>D: 保存回答文本或选项、时间和问答卡版本
    A->>D: 服务端读取答案、要点和来源
    A->>G: 请求评价回答
    alt Grok Bot 成功
        G-->>A: 正确处、遗漏、错误、模糊点和依据
    else Grok Bot 失败
        A->>A: 保留回答，启用手工对照模式
    end
    A->>F: 预览四档评分的复习时间
    F-->>A: 返回间隔预览
    A-->>U: 揭示答案、来源、评价和间隔预览
    U->>A: 选择忘记、困难、良好或容易
    A->>F: 提交当前状态、实际时间和用户评分
    F-->>A: 返回新记忆状态和到期时间
    A->>D: 事务保存日志、状态及同源问答卡暂缓
    D-->>A: 保存成功
    opt 该概念近期多次忘记或困难
        A-->>U: （目标）建议讲解追问：现在开始、稍后安排或忽略；MVP 未做此建议流
        U->>A: 选择处理方式
        A->>D: （目标）仅在选择稍后安排时记录理解练习计划；MVP 无 DeepPracticeSchedule
    end
    A-->>U: 展示下一题或会话结果
```

## 3. 概括对照与理解问答

```mermaid
sequenceDiagram
    actor U as 用户
    participant A as 理解练习组件
    participant D as 数据库服务
    participant G as Grok Bot 服务

    U->>A: 选择卡片集或内容卡片的理解练习
    A->>D: 读取阅读输入与未完成练习
    D-->>A: 返回 open_questions 与隐藏 content_summaries
    A-->>U: 展示理解问答或概括对照任务，不展示参考
    U->>A: 提交理解问答回答或自写概括
    A->>D: 保存用户原始版本
    A->>G: 提交回答、评价维度和来源
    G-->>A: 返回概念、因果、证据、应用和边界反馈
    A-->>U: 展示反馈、摘录和参考摘要（MVP：规则 stub）
    U->>A: 提交修订版本
    A->>D: 保存修订、未解决缺口和下次练习日期
    D-->>A: 保存成功
    A-->>U: 显示本次理解练习结果
```

## 4. 可选讲解追问与薄弱点回流

```mermaid
sequenceDiagram
    actor U as 用户
    participant A as 理解练习组件
    participant D as 数据库服务
    participant G as Grok Bot 服务
    participant F as FSRS 服务

    alt 用户主动开始
        U->>A: 口令「讲解追问」，从卡片集、内容卡片或概念开始
    else 用户处理系统建议
        U->>A: 选择现在开始
    else 用户打开已安排任务
        U->>A: 从理解练习入口开始（MVP：显式 CLI/口令）
    end
    A->>D: 读取概念、来源和历史缺口
    D-->>A: 返回练习上下文
    A-->>U: 请用自己的话解释，不展示标准答案
    U->>A: 提交首次解释
    A->>D: 保存首次解释
    loop 逐轮追问
        A->>G: 提交解释、来源、历史和学生模式规则
        G-->>A: 返回一个针对当前缺口的追问
        A-->>U: 展示追问
        U->>A: 回答并补充解释
        A->>D: 保存本轮对话
    end
    A->>G: 请求总结理解缺口
    G-->>A: 返回错误、模糊处和遗漏联系
    A-->>U: 展示反馈，请用户重新解释
    U->>A: 提交修订解释
    A->>D: 保存修订和缺口
    A-->>U: 选择结束、安排下次或生成补充问答卡
    opt 用户选择生成并确认补充问答卡
        A->>G: 根据选定缺口生成问答卡草稿
        G-->>A: 返回问题、答案和来源依据
        A-->>U: 展示草稿供审核
        U->>A: 编辑并确认
        A->>D: 发布新问答卡
        A->>F: 初始化独立问答卡状态
        F-->>A: 返回初始学习状态
        A->>D: 保存调度状态
    end
    A-->>U: 显示练习结果和后续安排
```

## 5. 两条独立队列及可选连接

```mermaid
sequenceDiagram
    actor U as 用户
    participant A as 学习总览组件
    participant D as 数据库服务
    participant F as FSRS 服务

    par 问答卡复习队列
        A->>F: 查询 FSRS 到期问答卡
        F-->>A: 返回今日问答卡
    and 理解练习汇总
        A->>D: 查询 open_questions / content_summaries / feynman_sessions 状态
        D-->>A: 返回今日理解练习汇总
    end
    A-->>U: 分区展示两条队列
    U->>A: 自主选择进入其中一条
    opt 问答卡多次回想困难
        A-->>U: （目标）建议讲解追问；MVP 未做
        U->>A: 现在开始、稍后安排或忽略
    end
    opt 讲解追问发现值得长期记忆的缺口
        A-->>U: 询问是否生成补充问答卡
        U->>A: 确认后加入 FSRS 队列
    end
```

## 6. Grok Bot 调用失败与恢复

```mermaid
sequenceDiagram
    actor U as 用户
    participant A as 调用管理组件
    participant D as 数据库服务
    participant G as Grok Bot 服务

    A->>G: 发送带 request_id 的生成或评价请求
    alt 请求成功
        G-->>A: 返回结构化结果和模型版本
        A->>D: 按 request_id 保存结果
        A-->>U: 展示结果
    else 超时或服务错误
        G--xA: 返回错误或无响应
        A->>D: 保存失败状态、输入版本和重试次数
        A-->>U: 显示失败，可重试或手工继续
        U->>A: 选择重试
        A->>G: 使用同一 request_id 重试
        G-->>A: 返回结果
        A->>D: 幂等更新原任务
        A-->>U: 展示恢复后的结果
    end
```
## 完整图表

- [SVG](../../diagrams/grok-active-reading/system-sequence.svg)
- [PNG](../../diagrams/grok-active-reading/system-sequence@2x.png)

