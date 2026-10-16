# 数据模型

## 目标模型 vs MVP 简化

产品文档中的「目标实体」描述完整愿景；**当前实现**以 `<APP_HOME>/schema.sql` 为准。下表先列 MVP 实表，再列仅目标扩展的实体。

### MVP 已实现表（与 schema 一致）

| 表 / 概念 | 关键字段 | 说明 |
|---|---|---|
| `card_sets` | id、name、description、tags_json、archived、created_at | 卡片集 |
| `content_cards` | id、card_set_id、title、**body**、created_at、updated_at | 原料；**主要为 body**，无完整 content/来源/失效字段集 |
| `knowledge_notes` | id、content_card_id、concept_name、explanation、source_excerpt | 概念笔记 |
| `review_cards` | 题面、答案、选项、**state / due_at / stability / difficulty / …**、invalidated、voided | 问答卡；**FSRS 状态直接落在本表**，无独立 ReviewState |
| `review_sessions` | id、card_set_id、started_at、ended_at、status、abort_reason | status：active / completed / aborted |
| `review_logs` | answer_text、rating、eval_note、prev/next_state_json、session_id | 每次复习一条；reveal 可写轻量 eval_note |
| `draft_batches` / `draft_items` | batch、item_type、payload_json、status | 草稿批次 |
| `open_questions` | content_card_id、question、hint、user_answer、feedback_json | 理解问答 |
| `content_summaries` | content_card_id、role、text、visible | 概括对照用摘要（含隐藏参考） |
| `feynman_sessions` / `feynman_turns` | target_type/id、status、turn_type、text、diagnosis_json | 讲解追问 |
| `config` | key、value | 如 retention |
| `audit_logs` | action、entity_type、entity_id、payload_json | 审计 |

**MVP 不存在的独立表**：`Practice`、`Weakness`、`DeepPracticeSchedule`、`AIJob`、`ImportRecord`、`ReviewState`、`DialogueTurn`（对话在 `feynman_turns`）、`UserSettings`（用 `config` 键值代替）。

### 目标扩展（未做或简化）

| 目标实体 | 意图 | MVP |
|---|---|---|
| Practice | 统一开放题/摘要练习对象 | 拆为 `open_questions` + `content_summaries` |
| Weakness | 结构化薄弱点 | 缺口可落在 feynman `diagnosis_json` / 草稿；无独立表 |
| DeepPracticeSchedule | 用户稍后安排的理解练习队列 | **未做**；今日总览汇总现有练习状态 |
| AIJob | 跟踪模型调用 | **未做** |
| ImportRecord | 微信读书等导入去重 | **未做** |
| 完整 ContentCard 来源/失效字段 | URL、页码、invalid/voided | **未做**；仅 `body` 为主 |

## 核心业务规则

1. 生成内容全部先进入草稿态，未经用户确认不得加入复习。
2. 卡片集和内容卡片不绑定书籍结构；MVP 内容卡片以 `body` 原文为主。
3. 微信读书等外部导入为**目标能力**，MVP 未做。
4. 失效/作废：问答卡有 `invalidated`/`voided` 标志；内容卡片完整失效工作流为**目标扩展**。
5. 问答卡支持问答式和选择题；FSRS 状态在 `review_cards`。
6. 主动回想必须遵循“先回答、后揭示”。
7. 用户评分是 FSRS 的唯一语义评分来源。reveal 侧规则评测只写 `eval_note`，不自动 `rate`。
8. 选择“困难”表示已经正确回想但较费力；没有回想出来必须选择“忘记”。
9. 问答卡复习与**理解练习**是两条独立流程。完成问答卡复习后不得自动进入讲解追问。
10. 概括对照、理解问答、讲解追问属于理解训练；只有用户主动开始（口令）后才进行，不与问答卡 FSRS 状态混用。无用户口令「深度练习」。
11. 复习后「建议讲解追问 / 稍后安排」为**目标能力**；MVP **未做**，仅支持显式 `feynman.py` / 口令「讲解追问」。
12. 讲解追问发现缺口后，补充问答卡须用户确认生成并 `draft-publish` 后才入 FSRS。
13. 时区和日界线默认 Asia/Shanghai。
14. 面向用户时，「新建卡片」仅表示原始输入入库；生成练习草稿与发布练习是后续独立步骤。
15. 今日复习开场必须指定卡片集；列表须展示「还剩 x/全部」。
16. 复习场结束区分主动中止（SESSION_ABORTED）与自然完成（SESSION_COMPLETE）；中止时未评分卡不打分。
17. 短间隔 learning 卡到点后默认不主动通知（MVP 无通知服务）。
18. 单用户部署；无多租户鉴权。
